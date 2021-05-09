---
title: "Go와 Raspberry Pi로 온습도계 만들기"
date: 2021-05-09T22:00:00+02:00
tags:
  - raspberry-pi
  - gpio
  - golang
  - bcm2835
series:
  - DIY Thermometer
mermaid: true
isCJKLanguage: true
---

## Introduction

처음 지금 살고 있는 집에 들어왔을 때는 환기에 대한 상당한 오개념을 가지고 있었다. 한국에서 살 때는
공기가 답답할 때나 한번씩 환기하면 충분했었다. 추운 겨울에는 난방 효율을 위해서 웬만하면 환기 없이
겨울을 보내곤 했었다. 하지만 그건 오산이었다.

독일의 창문은 [틸트 앤 턴][tilt-and-turn]이라는 방식의 창호를 많이 쓴다. 우리나라에서는 미닫이
창문이 흔하기 때문에 보기 쉽지는 않은데, 회사 사무실이나 도서관 같은 중앙 공조 시스템으로 환기를
하는 건물에서는 틸트 방식으로 여는 창문을 종종 볼 수 있다.

{{< figure
  src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/84/Bleekman_zimmer.jpg/512px-Bleekman_zimmer.jpg"
  alt="Bleekman zimmer"
  caption="Bleekman, [CC BY 3.0](https://creativecommons.org/licenses/by/3.0), via Wikimedia Commons"
  link="https://commons.wikimedia.org/wiki/File:Bleekman_zimmer.jpg"
>}}

위 사진은 위키미디어에서 검색한 어떤 독일 사람의 방 사진인데, 창문이 세 조각으로 되어 있고 중간
창문이 틸트 형식으로 열려 있는 것을 볼 수 있다. 손잡이가 위쪽 방향으로 돌려져 있는 것을 볼 수
있는데, 이걸 절반만 돌리면 여닫이 창문으로 바뀌는 마술같은 창문이 [틸트 앤 턴][tilt-and-turn] 방식
창문이 되겠다.

월세 계약서에 환기를 자주 하는 것이 세입자의 의무라고 되어 있어서, 저 창문을 틸트 형태로만 하루에
한번 열어서 환기를 시켰다. 이 정도만 해도 한국에서 살던 때에 비하면 매우 자주 환기를 한 셈이라 환기
때문에 어떤 문제를 겪을 거라곤 상상도 못 했었다.

그러나 며칠이 지나자, 창문 위쪽에 거뭇한 무언가가 나타나기 시작했다. 바로, 곰팡이였다. 급하게 곰팡이
제거제를 사다가 뿌리고 닦고 해도 일주일쯤 지나면 다시 나타나서 우리를 괴롭혔다.

또 하나의 문제는, 자꾸 창문 틀 아래쪽에 물이 고이는 문제였다. 낮에는 생기지 않는데 자고 일어나면
창문마다 물이 고여있어 매일 닦아줘야했다. 많이 고이면 마루바닥에까지 흐르는 바람에 자국까지 남고...

나중에 알게 된 사실인데, 그렇게 열어가지고는 제대로 환기가 되질 않았던 것이 문제였다. 벽지도 없이
페인트로 마감이 되어 있는지라 집의 습기가 제대로 제거가 되지 않았고, 추운 밤이 되면 습하고 더운 집
안 공기가 차가운 창문과 만나 응결이 됐던 거였다. 빨래도 건조기도 없고 날아갈까봐 발코니에서 말리지도
못해서 집 안에서 말리다보니 집 안 습도가 스페이스엑스마냥 저 하늘 높이 치솟는건 정해진 수순이었다.
그래서 결국은 창문을 완전히 열어서 아침 저녁으로 10분씩 환기시키기 시작했는데, 그제서야 습기가 좀
잡혔다.

그래서 내린 결론은, 집에 온습도계가 좀 있어야겠다는 것이었다. 기왕 하는거 온습도 센서를 사다가 REST
API 비슷하게 만들어서 웹 인터페이스로 볼 수 있게 만들고, 수집해서 모니터링용 그래프도 만들고, Alert
도 만들면 뭔가 SRE가 사는 집 같은 느낌이 들지 않을까?

[tilt-and-turn]: https://en.wikipedia.org/wiki/Window#Tilt_and_turn

## Goal

* 온습도를 확인할 수 있는 REST API 비슷한 것을 제공하는 HTTP 서버를 만들자.
* 만드는 김에 [Golang][golang]으로 작성해보자.
* 만드는 데 돈이 너무 비싸게 들면 안된다.

[golang]: https://golang.org/

## Non-goal

* 이번 시간에는 API 서버만을 만들어보도록 하자. UI, 수집, Alert은 다음 시간에 만들도록 하자.

## Background

[Raspberry Pi][rpi]는 오픈소스 싱글 보드 컴퓨터이다. 원래 홈 서버용으로 쓰려고 사놓고 [Kodi][kodi]
를 통해서 아이들 넷플릭스 머신으로 쓰고 있는 중이긴 하지만, 라즈베리파이는 여러 개의 GPIO 포트를
가지고 있어서 저수준 제어도 어느 정도 가능하다.

{{< figure
  src="https://raw.githubusercontent.com/Gadgetoid/Pinout.xyz/master/resources/raspberry-pi-pinout.png"
  alt="Raspberry Pi Pinout"
  caption="Raspberry Pi Pinout from [pinout.xyz](https://pinout.xyz)"
  link="https://pinout.xyz"
>}}

[DHTxx][dht] 센서는 온도와 습도를 측정한 뒤 디지털 신호로 내보내주는 디지털 온습도 센서이다. 센서
자체에는 단자가 4개 나와있지만 그 중 하나는 안 쓰는 단자고, Vcc와 GND 단자를 제외하면 하나의 단자로
측정한 온습도 정보를 전달한다. 내부적으로는 [Thermistor][thermistor]에서 나온 아날로그 신호를 DAC을
통해 디지털로 변환해서 내보내준다고 한다.

[Golang][golang]은 GC언어이기 때문에 non-deterministic한 GC delay가 발생할 수 있다. 이는 gpio의 신호
폭을 재거나 하는 시간에 예민한 동작에 방해가 될 가능성이 있다.

[bcm2835][bcm2835]는 [Raspberry Pi][rpi]에서 사용하는 SoC 이름이지만 이를 다루기 쉽게 싸 놓은
라이브러리의 이름이기도 하다. 이 라이브러리는 C 언어로 작성되어 있으며 [Raspberry Pi][rpi] 에서 무리
없이 잘 작동한다. 다만 주의해야할 점이 있는데, 이 라이브러리는 특정 조건이 만족되면 `/dev/mem`
장치를 통해 직접 물리 메모리에 접근하려고 시도한다. 루트 권한으로(EUID == 0) 실행되고 있거나
`CAP_SYS_RAWIO` kernel capability가 부여되어 있을 때가 그 조건인데, 이 조건을 만족하지 못하면 대신
`/dev/gpiomem` 장치에 접근을 시도하며 이 때는 GPIO 기능만을 사용할 수 있다. 주의해야 할 부분은
해당 조건을 만족하면서도 `/dev/mem`에 접근할 권한이 없을 때이다. 예를 들면 fakeroot 등으로 euid를
속였거나 Docker로 실행할 때가 문제가 된다. 우리는 GPIO만 관심이 있기 때문에 간단히 uid를 0이 아닌
값을 사용하는 것으로 얼마든지 피할 수 있다.

[JSON][json]은 비정형의 키-값 구조를 저장하기에 적절한 표현 방법으로, 일반 텍스트 형태를 하고 있어
사람 눈으로 확인하기 편리하고, 웹으로 표시할 때 JS 엔진이 무리 없이 간단하게 변환 가능하기 때문에
널리 쓰이는 표현방법이다. 사용시 주의할 점은 JS와는 달리 키값이 항상 쌍따옴표(`"`)로 감싸져 있어야
하고 맨 마지막 값 뒤에 콤마(`,`)가 있으면 안된다는 점과 주석이 지원되지 않는다는 점이다. 가끔 주석도
지원하는 해석기가 있지만 표준은 아니다.

[cgo][cgo] 패키지는 일종의 [FFI][ffi]로, [Go][golang] 코드 내에서 C 코드를 호출할 수 있게 해주는
기능이다. 사용 방법이 무척이나 간단하기 때문에, [Go][golang]을 쓰면서 다른 언어로 작성된
라이브러리와 결합하는 것을 두려워하지 않게 해 준다. 간단한 예시를 들면 다음과 같다.

```go
// #include <stdio.h>
// static void greeting(const char * name) {
//   printf("Hello, %s!\n", name);
// }
import "C"

func main() {
  C.puts(C.CString("Hello, world!"))
  C.greeting(C.CString("cgo"))
}
// Result:
// Hello, world!
// Hello, cgo!
```

주의할 점은, GC managed runtime인 Go 코드와 unmanaged인 C 코드가 만나는 지점에서 메모리 관리가 잘
처리되지 않으면 다양한 문제가 생길 수 있다는 점이다. [cgo][cgo] 페이지를 참조하면 좀 더 자세한
설명을 볼 수 있다.

[rpi]: https://raspberrypi.org/
[kodi]: https://kodi.tv/
[dht]: https://components101.com/sensors/dht11-temperature-sensor
[thermistor]: https://ko.wikipedia.org/wiki/%EC%84%9C%EB%AF%B8%EC%8A%A4%ED%84%B0
[bcm2835]: https://www.airspayce.com/mikem/bcm2835/
[json]: https://www.json.org/
[cgo]: https://golang.org/cmd/cgo/
[ffi]: https://en.wikipedia.org/wiki/Foreign_function_interface

## Design

<pre class="mermaid">
sequenceDiagram
    participant User
    participant API Server
    participant Sensor
    API Server->>API Server: Initiate GPIO
    User->>API Server: GET /
    API Server->>Sensor: Initiate sensor
    alt is succeded
      Sensor->>API Server: Send humidity/temperature
      API Server->>User: Send humidity/temperature
    else is failed
      Sensor->>API Server: Send malformed data
      API Server->>User: Send Error
    end
</pre>

구조는 간단하다. 사용자가 HTTP 요청을 보내면 API server는 센서를 깨우고, 센서로부터 받은 정보를
사용자에게 전달한다. 간단하고 효율적인 구현을 위해 [JSON][json] 형태로 결과를 인코딩해서 보낸다.

성공한 경우에는 다음 형태로 결과를 출력한다.

```json
{
  "humidity": 40.0,
  "temperature": 20.0
}
```

실패한 경우에는 다음처럼 에러를 인코딩해 보낸다.

```json
{
  "err": "Something went wrong"
}
```

### Source

tl;dr을 좋아하는 사람을 위해 소스코드를 깃헙에 올려 두었다.

<https://github.com/gwangyi/gondogye>

이 문서에서 다룬 내용에 보태서 Dockerfile이 추가되어 있다. 다음 포스팅에서는 이걸 Docker container
로 만드는 것을 다룰 예정이다.

### DHT11/22

나는 [DHT11][dht11] 센서를 독일 아마존에서 구입했다. 사실 잘 모르고 인터넷에서 라즈베리 파이로 온도
측정하는 예제를 찾다가 아무거나 얻어걸린 것을 산 건데, DHTxx센서는 DHT11과 DHT22 두 종류가 있었다.

DHT22가 DHT11보다 좀 더 정밀하고 좀 더 넓은 온·습도 영역을 측정할 수 있지만, DHT11이 좀 더 빠르게
측정할 수 있고 좀 더 촘촘하게 측정하는 것이 가능하다고 한다.

둘 다 생긴 것도 비슷하고 사용하는 프로토콜도 같다. 아무거나 사면 되지만 신경쓸 부분이 하나 있다.
원래 DHTxx 센서는 단자가 4개 있는데, 그 중 3개만 사용하며 GPIO에 연결할 때는 풀업 저항을 달아야
한다. 그게 귀찮으면 내가 산 것처럼 보드에 붙어 있는 형태의 센서를 사면 된다.

[데이터 시트][dht11-datasheet]에 나와 있는 타이밍 다이어그램을 살펴보면,

{{< figure src="https://components101.com/asset/sites/default/files/inline-images/DHT11%E2%80%93Temperature-Sensor-Working-waveform.png" >}}

1. 제일 먼저 HIGH 상태를 유지해준 후 18ms보다 긴 시간동안 LOW를 쓰고 PULL된 상태로 기다린다.
2. 신호가 LOW로 떨어지면 80ns만큼 유지된 뒤 다시 HIGH에서 80ns를 유지하고 나서 데이터를 전송하기 시
  작한다.
3. LOW에서 50ns만큼 유지한 뒤 HIGH가 되는데, 0일 때와 1일 때의 펄스 폭이 다르다. 0은 26-28ns, 1은
  70ns만큼 유지한다.
4. 총 40bit가 전송되며, 그 중 마지막 8bit는 checksum 역할을 한다.

8bit씩 나눠서 봤을 때 checksum은 다음과 같다.

```
data[4] = (data[0] + data[1] + data[2] + data[3]) mod 256
```

[dht11]: https://www.amazon.de/gp/product/B07V61GKVQ/
[dht11-datasheet]: https://components101.com/asset/sites/default/files/component_datasheet/DHT11-Temperature-Sensor.pdf

### bcm2835

[bcm2835][bcm2835] 라이브러리가 git으로 호스팅되고 있었다면 [submodule][git-submodule]으로 추가했을
테지만, 안타깝게도 그렇지가 못했다. 공식 홈페이지에서 [최신버전 소스코드][bcm2835-1.68]를 먼저
다운받았다.

[bcm2835][bcm2835]는 사실 [Go][golang]으로 작성된 것이 아니라 C로 작성되어있다. 하지만 훌륭한 C FFI
인 [cgo][cgo]를 사용하면 연결하는게 많이 어렵지 않다.

[bcm2835][bcm2835]에서 우리가 사용하게 될 함수는 그렇게 많지 않다.

* [bcm2835\_init()][bcm2835_init]: bcm2835 안에서 사용하는 자료구조를 초기화한다.
* [bcm2835\_gpio\_fsel()][bcm2835_gpio_fsel]: 지정한 GPIO 핀의 입·출력 방향을 지정한다.
* [bcm2835\_gpio\_write()][bcm2835_gpio_write]: 지정한 GPIO 핀의 출력값을 바꾼다.
* [bcm2835\_gpio\_lev()][bcm2835_gpio_lev]: 지정한 GPIO 핀에 입력되는 값을 구한다.

fen, ren 등의 에지 디텍터 함수들도 있지만, 이건 사용하니까 라즈베리파이가 죽었다... 죽는 이유는
정확히는 모르겠지만 아마도 에지 디텍터 관련 레지스터를 어떤 커널 드라이버에서 쓰고 있는데 마음대로
조작했다가 중간에 인터럽트가 걸리면서 뭔가가 망쳐지는 것 같다.

[git-submodule]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
[bcm2835-1.68]: http://www.airspayce.com/mikem/bcm2835/bcm2835-1.68.tar.gz
[bcm2835_init]: https://www.airspayce.com/mikem/bcm2835/group__init.html#ga9351fa3ec8eeff4e9d998d3d5d912a4f
[bcm2835_gpio_sel]: https://www.airspayce.com/mikem/bcm2835/group__gpio.html#gaf866b136c0a9fd4cca4065ce51eb4495
[bcm2835_gpio_write]: https://www.airspayce.com/mikem/bcm2835/group__gpio.html#ga22f9b05d8edda3ef57cd58728e9c3baa
[bcm2835_gpio_lev]: https://www.airspayce.com/mikem/bcm2835/group__gpio.html#ga693becf47034d3b2d8ed0ac4e74de173

#### Initiate Sensor

<pre class="mermaid">
graph
    1[Set pin mode as output]
    1 --> 2[Set pin to HIGH]
    2 --> 3[Wait 500ms]
    3 --> 4[Set pin to LOW]
    4 --> 5[Wait 20ms]
    5 --> 6[Set pin mode as input]
    6 --> 7[Wait until LOW]
</pre>

위에서 다이어그램을 보고 정리했던 순서도의 2번까지에 해당하는 내용이다. 처음에 500ms를 기다리는
이유는 다음에 보낼 신호가 1번에 해당하는 신호임을 확실하게 만들기 위해서이다.

#### Read Pulse Width

<pre class="mermaid">
graph
    1[Keep timestamp]
    1 --> 2[Wait for the change of the level of GPIO pin]
    2 --> 3[Calculate delta on the timestamp]
    3 --> 4[Update timestamp]
</pre>

시간을 잴 때는 [gettimeofday()][gettimeofday] 함수를 쓰면 us 단위로 현재 시각을 얻어올 수 있다.
2단계에서 무한정 기다리게 되는 일이 생길 수 있으므로 일정 숫자만큼 세고 나면 탈출할 수 있도록
조치하는 것이 필요하다. 이걸 고정된 값으로 하면 코어 주파수에 따라 시간이 들쭉날쭉할 수 있어서, 일단
충분히 큰 수만큼 세어 보면서 시간이 얼마나 걸리는지 잰 다음 원하는 시간(여기서는 100us) 비율로 나눠
그 시간만큼 걸릴만한 숫자를 계산해서 이를 활용한다.

[gettimeofday]: https://man7.org/linux/man-pages/man2/gettimeofday.2.html

#### Read Response

위에서 작성한 펄스 폭을 재는 함수를 활용하면 간단하게 작성할 수 있다. 읽어야 하는 펄스의 개수는
80개이다. 한 비트에 LOW/HIGH 두 번의 펄스가 있고, 8비트씩 5개 정수가 나오기 때문에 80개가 된다.

GPIO를 완전히 컨트롤할 수 있는 커널이나 펌웨어 레벨에서 접근하는 것이 아니기 때문에, 약간 밀리거나
시간 축정에 오차가 생기거나 하는 경우를 감안해서 처리할 필요가 있다. 이 프로젝트의 경우에는, 먼저
나오는 LOW 신호의 폭을 따져보고 50us에서 너무 많이 벗어나는 경우(30-70us 범위를 벗어나면) 잘못된
신호로 취급하기로 했다. 그 뒤에 나오는 HIGH 신호는 26~28us의 경우 0이고 70us일 경우 1이므로 50us보다
짧은지 긴지를 기준으로 값을 인식시켰다.

#### Checksum

마지막으로 체크섬 처리를 여기서 했는데, 사실 체크섬은 시간에 영향을 받지 않는 연산이라 대원칙에
따르면 [Go][golang] 쪽에서 처리해도 무관하다. 하지만 C에서 처리한 이유는, 이 체크섬을 C에서 처리하지
않는 경우 [Go][golang] 쪽으로 40비트 정수를 넘겨야 하지만 미리 처리하고 잘라내서 넘길 경우 32비트만
넘기면 되기 때문이다. 40비트를 넘기려면 최소 64비트 정수형 자료형을 써야하지만 8비트를 미리 자르면
32비트로 충분하기 때문에 조금 더 보기에 좋다.

#### Why C?

지금까지 C언어로 작성했는데, 그 이유는 [Go][golang]가 managed 언어이기 때문이다. 동작들이 us단위로
시간을 재면서 진행되어야 하는데, [cgo][cgo] 브릿지 오버헤드나 [Go][golang]의 GC 등이 예측하지 못한
타이밍에 갑자기 호출되어버리면 시간을 재는 작업이 엉터리가 되어버릴 위험성이 있다. 따라서 직접 GPIO
이용해 센서와 통신하는 부분은 C로 작성하여 예측할 수 없는 문제를 최소화시키도록 했다.

윗 단계에서 얻어낸 32비트 정수형 자료는 DHT11, DHT22에서 서로 다른 의미를 가지고 있다. 이 것까지
모두 해석해서 반환할 수도 있지만, 이 해석하는 작업은 us 단위로 시간을 따져가며 계산할 필요가 없기
때문에 C API쪽에서 처리하지 않고 그대로 반환한다. 이 값은 [Go][golang] 쪽에서 해석할 것이다.

#### dht/dht.h

위에서 작성한 C로 작성된 함수들의 인터페이스는 대략 다음과 같다.

{{< gist-it "gwangyi/gondogye" "main" "dht/dht.h" >}}

나중에 이 파일은 [cgo][cgo]에서 include할 예정이다. 이를 통해 자동으로 브릿지가 생성돼서
[Go][golang] 쪽에서 쉽게 호출이 가능해진다.

### Application Server

[Go][golang]에는 간단한 http 서버를 제작할 수 있는 라이브러리인 [net/http][net-http]가 표준
라이브러리에 포함되어 제공된다.  물론 [gin][gin]과 같이 본격적인 Web server framework가 없는 것은
아니지만, 간단하게 온습도 정보를 제공하는 정도의 API 서버는 [net/http][net-http]로 충분할 것이다.

API 서버는 앞에서 작성한 C API를 호출하여 32비트 정수값을 얻어온 뒤에, 실제 연결된 센서 종류에 따라
제대로 된 온습도 정보를 추출해 [JSON][json]으로 반환할 예정이다.

[net-http]: https://golang.org/pkg/net/http/
[gin]: https://gin-gonic.com/

#### CGO Binding

[cgo][cgo] Binding을 작성하는 것은 어렵지 않다. `import "C"` 만으로 충분하다. 더해서 사용할
함수들과 구조체, enum 등이 정의된 헤더파일을 주석 형태로 위에 적어주면 완벽하다. 우리의 경우
[Go][golang] 쪽에서 호출해야 하는 함수는 bcm2835 라이브러리에서 초기화 함수인 `bcm2835_init()`,
위에서 작성한 DHT API에서 `dht_init()`, `dht_read()` 를 호출할 예정이다.

```go
// #cgo CFLAGS: -I.
// #include <bcm2835.h>
// #include <dht.h>
import "C"
```

첫 줄은 [cgo][cgo] 컴파일 시 C 코드를 컴파일할 컴파일러에게 전달하는 CFLAGS 옵션을 지정하고 있다.
헤더파일과 go 소스파일, C 코드 모두가 같은 디렉토리에 있기 때문에 이렇게 했다. 특이한 점은 헤더를
include했지만 소스코드는 특별히 처리하지 않았는데, 같은 디렉토리에 있는 C 파일은 자동으로 같이 빌드
해 주기 때문에 지정할 필요가 없기 때문이다. 만약 다른 디렉토리에 있는 코드를 가져오고 싶다면 별도의
빌드 시스템을 이용할 필요가 있다. 이건 의도된 설계인데, 이유가 궁금하면
[관련된 github 이슈][please-bazel]를 참조하기 바란다. 좀 더 자세한 내용은 [cgo][cgo] 패키지 설명을
참조하면 CFLAGS부터 LDFLAGS, pkg-config 등의 환경을 셋업하는 방법에 대해 찾아볼 수 있다.

맨 처음엔 자동으로 bcm2835 라이브러리를 초기화시키고 싶으므로 [`init()`][go-init] 함수를 정의해보자.

```go
func init() {
  if bcm2835_init() == 0 {
    panic("bcm2835_init() has been failed")
  }
}
```

이제 간단한 인터페이스를 정의해보자.

```go
type cDHT interface {
  func Read() (uint32, error)
}
```

실제 코드에는 디버깅을 위한 로그레벨을 조절하는 함수가 하나 더 있지만, 필요없는 함수이므로 언급하지
않았다.

이제 이 것을 구현하는 구현체를 만들어볼텐데, 새로운 `struct`를 만들지 않고 작성해보자.

```go
func newDHT(pin int) *C.struct_dht_sensor {
  dht := C.struct_dht_sensor{}
  C.dht_init(&dht, C.int(pin))
  return &dht
}

func (dht *C.struct_dht_sensor) Read() (uint32, error) {
  output := C.uint32_t(0)
  ret := C.dht_read(dht, &output)
  if ret != 0 {
    return uint32(output), nil
  }
  return 0, fmt.Errorf("Unexpected behavior on the DHT sensor at %v", dht.pin)
}
```

주의해서 볼 부분은, C에서 정의된 struct를 접근할 땐 이름의 앞에 `struct_`가 붙어있어야 한다는 것과
C의 int형 인자값을 받을 때도 `C.int()`로 감싸야 한다는 것 정도가 되겠다. 마찬가지로 [Go][golang]의
정수형으로 반환하고 싶을 때도 감싸줘야한다.

[please-bazel]: https://github.com/golang/go/issues/12953
[go-init]: https://golang.org/doc/effective_go#init

#### Parse DHT11 Response

앞에서 정의한 인터페이스는 의도적으로 private으로 정의했다. 실제로 외부에 보여줄 인터페이스는 정확히
온도와 습도 정보를 담은 구조체를 반환하도록 하고 싶기 때문이다.

```go
type Result struct {
  Humidity float32
  Temperature float32
}

type DHT interface {
  Read() (Result, error)
}

type dht11 struct {
  cdht cDHT
}

func NewDHT11(pin int) DHT {
  return &dht11{cdht: newDHT(pin)}
}

func (dht *dht11) Read() (Result, error) {
  output, err := dht.cdht.Read()
  if err != nil {
    return Result{}, err
  }
  return Result {
    Humidity:    float32((output >> 24) & 0xff),
    Temperature: float32((output >> 8) & 0xff),
  }
}
```

DHT11 센서는 첫 8비트가 습도, 세 번째 8비트가 온도를 나타낸다. 두 번째와 네 번째는 소숫점 이하
수치라고 하는데, 정확히 설명된 자료를 찾지 못했다. `Result`를 포인터로 반환하지 않은 이유는, 32비트
실수형 자료 두 개라 용량이 많이 크지 않고 64 비트 환경에서는 심지어 한번에 전달이 가능하기 때문이다.

#### Build HTTP Server

간단하게, HTTP 요청이 오면 센싱한 뒤 결과를 JSON으로 인코딩해서 돌려줄 것이다.

{{< gist-it "gwangyi/gondogye" "main" "server/server.go" >}}

그러고 나면 이 서버를 실행시키는 간단한 main 함수를 작성하면 끝이다.

{{< gist-it "gwangyi/gondogye" "main" "main.go" >}}

플래그를 사용해 웹서버를 바인드하는 아이피나 포트, 그리고 센서가 연결된 GPIO 포트를 지정할 수 있게
했다.

## Conclusion

이제 C 코드와 [Go][golang] 코드가 적당히 섞인 간단한 API 서버를 만들 수 있게 되었다. 덤으로 bcm2835
라이브러리의 간단한 사용법을 익혔고, 간단한 gpio 활용 기능도 작성할 수 있게 되었다.

사실 다 만들고 나서 도커로 싸던 중에, 이걸 러스트로 만들었으면 더 재밌었을거라는 생각이 문득 들어
현자타임이 왔었다. 그래도 기왕 만든 것이고 의미가 있을거라 생각해서 작성 과정을 글로 남겨보았는데,
생각보다 너무 길어 분량 조절을 망친 것 같다.
