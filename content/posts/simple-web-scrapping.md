---
title: "Simple Web Scrapping"
date: 2021-03-12T22:11:55+01:00
tags:
  - bash
  - web
  - telegram
isCJKLanguage: true
---

## Introduction

인터넷이 세상을 지배한다고 해도 과언이 아닌 시대를 살아가고 있다. 옛날에는 종이로 우편함으로 보내지던 고지서들이 이메일을 거쳐 메신저로 전달되고 있고, 예약이나 질의도 직접 찾아가는 방법 외에도 인터넷을 통해 다양한 방법으로 달성할 수 있는 시대가 되었다.

기술에 대한 신뢰가 별로 없는 이 독일 땅에서도, 코로나 판데믹이라는 전무후무한 재해로 인해 어쩔 수 없이 비대면 업무 처리를 위한 전산화가 조금씩 도입되고 있는 중이다.

최근 인후통, 콧물, 코막힘 등의 증상이 있어서 아이들과 함께 근처의 이비인후과에 들렀다. 독일어가 미진한 관계로 예약을 잘못 잡았는지 코로나 검사만 해 주고 돌려보내길래 검사 번호가 붙어 있는 엽서 크기의 접수증 세 장을 들고 털레털레 집으로 돌아왔다. 이제 채취한 검체를 연구소로 보내서 검사한 뒤 결과를 알려주겠다고 했는데, 아날로그를 매우 사랑하는 독일이지만 다행히 이 검사 결과는 웹사이트와 스마트폰 앱을 통해 조회가 가능하므로 정식 결과서를 받기 전에 미리 확인할 수 있다고 했다.

결과가 나오는데는 최대 3일 정도 걸린다고 했으니 3일 뒤에 확인해보는 방법도 있겠지만, 우리는 한민족, 빨리빨리의 민족이 아닌가. 3시간 간격으로 한번씩 홈페이지에 들어가서 결과를 확인하려니 귀찮기도 하고 재밌는 아이디어가 생각나서 웹 크롤러를 만들어보기로 했다. 귀찮고 반복적인 일은 최대한 자동화하기, 개발자의 황금률이다.

## Goal

* 이 결과는 길어야 3일 안에는 나온다. 다르게 말하면, 이 웹 스크래퍼는 3일 안에 완성되어야 의미가 있다는 것이다. 아마 3일까지는 안 걸릴 것이기 때문에, 길어야 두어시간, 기대하기로는 몇 분 정도 대충 끄적여서 완성해야 쓸 데가 있다.
* 테스트 결과는 간단하게 `처리중`, `양성`, `음성` 셋 중 하나로 나올 것이다. 처리중 상태에서 다른 상태로 바뀌면 즉시 어떤 방법으로든 알람을 보내서 알 수 있도록 하는 것이 목표다.
* 혹시 모르지만, 너무 많은 요청을 보내서 결과 조회 서버를 터뜨리거나 내가 차단되어서는 안될 것이다.

## Non-goals

* 실시간으로 결과를 받아 볼 필요는 없다. 물론 가능하다면 좋겠지만, 30초 일찍 알았다고 해서 내가 취할 수 있는 선택지가 바뀌는 부분이 없기 때문이다.
  사실 결과를 빨리 받아보고 싶었던 이유는 오늘중으로 소아과를 방문하고 싶었기 때문인데, 이 소아과는 예전에 들렀을 때 병원인 주제에 코로나 의심증상이 조금이라도 있으면 낫고 나서 오라는 이상한 소리를 하면서 돌려보냈었던 곳이기 때문에 결과가 필요했다.

## Background

[크롬 개발자 도구][chrome-devtools]는 재밌는 기능을 가지고 있다. 특히 웹 스크래핑을 시도할 때 꽤나 유용할만한 기능이 몇 가지 있는데, 대부분 개발자 도구의 [Network][chrome-devtools-network] 탭에서 발견할 수 있다. 기록된 요청에 오른쪽 버튼을 눌러 컨텍스트 메뉴를 열면 [Copy][chrome-devtools-network-copy] 메뉴가 있다. 하위 메뉴를 보면 똑같은 복사인데 엄청 많은 수의 하위 메뉴들이 있는 것을 볼 수 있다. 이 중에 Copy as cURL이 특히 좋은데, 이걸 선택하면 기록된 요청과 동일한 요청을 보내는 [curl][curl] 명령을 클립보드에 넣어준다. 그대로 다른 터미널에 붙여넣고 실행하면, 동일한 요청 헤더와 쿠키가 그대로 보내지기 때문에 간단한 자동화에 매우 큰 도움이 된다. 쿠키가 그대로 보내진다는 부분에서, 세션도 동일한 세션을 쓰기 때문에 별도의 로그인 절차를 따라할 필요가 없다는 것은 덤이다.

[jq][jq]라는 명령이 있다. 아마도 JSON Query의 줄임말이라고 생각하는데, 간단히 말해서 JSON을 표준 입력으로 받아 자체적인 query 문법을 통해 원하는 요소만 골라내서 보여주는 기능을 하는, 일종의 [sed][sed]나 [grep][grep] 같은 툴이다. 이런 비슷한 툴이 html이나 xml에도 있을 것이라고 생각하고 간단히 찾아봤더니 Go로 작성된 [pup][pup]를 찾을 수 있었다.  [pup][pup]은 [CSS selector][css-selector]를 이용해 html 문서를 필터링할 수 있는 프로그램이다.

[telegram-send][telegram-send]라는 파이썬 프로그램이 있다. CLI 환경에서 동작하는 봇 클라이언트인데, 설정파일을 생성해 주면 커맨드 라인에서 간단하게 봇을 통해 메시지를 보낼 수 있다. [텔레그램 봇][telegram-bot]은 [BotFather][botfather] 봇을 통해서 생성하고 관리할 수 있다. [문서][telegram-bot]를 보고 생성 방법을 숙지해 두자.

[chrome-devtools]: https://developers.google.com/web/tools/chrome-devtools
[chrome-devtools-network]: https://developers.google.com/web/tools/chrome-devtools/network
[chrome-devtools-network-copy]: https://developers.google.com/web/tools/chrome-devtools/network/reference#copy
[curl]: https://curl.se/
[jq]: https://stedolan.github.io/jq/
[sed]: https://www.gnu.org/software/sed/manual/sed.html
[grep]: https://man7.org/linux/man-pages/man1/grep.1.html
[pup]: https://github.com/ericchiang/pup
[css-selector]: https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors
[telegram-send]: https://github.com/rahiel/telegram-send
[telegram-bot]: https://core.telegram.org/bots
[botfather]: https://telegram.me/BotFather

## Design

흐름은 대략적으로 다음과 같다.

1. 검사 결과 페이지를 요청한다.
2. 결과 페이지의 dom 구조를 분석하여 결과 부분을 찾는다.
3. 양성이든 음성이든 나오면 메시지를 보낸다.
4. 나오지 않았다면 일정 시간을 기다린 후 1부터 다시 시작한다.

### Querying the result page

코로나 검사 결과를 조회해볼 수 있는 사이트는 <https://www.doctorbox.de/covid19.jsp>였다. 접속해 보면, 접수번호와 생년월일을 입력해서 코로나 검사 결과를 볼 수 있는 형태로 구성되어 있다.
{{< figure-rel src="/img/www.doctorbox.de_covid19.jsp.png" >}}

여기에 연월일을 입력하면 검사 결과가 나타난다.
{{< figure-rel src="/img/www.doctorbox.de_covid19-result.jsp.png" >}}

이제 해볼 것은, 위에서 이야기했던 [curl][curl] command를 [따 보는 것][chrome-devtools-network-copy]이다. 따 보면 다음과 비슷하게 나타날 것이다. 민감할 수 있는 정보는 `.` 으로 치환했다.
```bash
curl 'https://www.doctorbox.de/covid19-result.jsp' \
  -H 'Connection: keep-alive' \
  -H 'Cache-Control: max-age=0' \
  -H 'sec-ch-ua: "Google Chrome";v="89", "Chromium";v="89", ";Not A Brand";v="99"' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'Upgrade-Insecure-Requests: 1' \
  -H 'Origin: https://www.doctorbox.de' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 11_2_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.72 Safari/537.36' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' \
  -H 'Sec-Fetch-Site: same-origin' \
  -H 'Sec-Fetch-Mode: navigate' \
  -H 'Sec-Fetch-User: ?1' \
  -H 'Sec-Fetch-Dest: document' \
  -H 'Referer: https://www.doctorbox.de/covid19.jsp' \
  -H 'Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,de;q=0.6' \
  -H 'Cookie: JSESSIONID=............................; ........................=.............; site-lang=de' \
  --data-raw 'covid19ID=..........&day=..&month=..&year=....' \
  --compressed
```

위 명령의 결과는 다음과 비슷할 것이다.
```html
<!doctype html>
<html class="no-js" lang="de">

    <head>
        <meta charset="utf-8" />
        <meta http-equiv="x-ua-compatible" content="ie=edge">
        ...
    </head>

    <body>
    ...
            <!-- Start Covid -->
            <section id="covid19-2">
                <div class="width-xl grid-padding-t-x">
                    <div class="grid-x">



<div class="grid-x grid-padding-t-m">
    <div class="large-8 large-offset-2 medium-10 medium-offset-1 small-12 text-center normal-title">
        <h4>Auswertung Ihres COVID-19 Testabstriches</h4>
        <h5>Bitte beachten Sie, dass der Online Abruf nur zur Erstinformation dient. Ihren schriftlichen ärztlichen Befund für eventuell notwendige Nachweise erhalten Sie über die DoctorBox App.</h5>
    </div>


    <div class="large-12 medium-12 small-12 text-center negative-title mx-0 my-3 mx-md-5">
        <h1 class="text-uppercase">Ihr Ergebnis ist <span >IN BEARBEITUNG</span></h1>
    </div>
    <div class="large-6 large-offset-3 medium-10 medium-offset-1 small-12 text-center covid-notification-body">
        <!--
        <h4>Sie sind sich nicht sicher, ob Sie Covid19 schon hatten?</h4>
        <h5>Finden Sie es mit einem Covid19 <strong>Antikörpertest</strong> heraus!</h5>
        <div class="mb-5"><a href="aerzte.jsp" class="button" data-localize="mob-menu-button-1">Jetzt bestellen</a></div>
        -->
        <div class="pt-3"><h4>Hilfe zu Ihrem Testabstrich</h4></div>
        <h5 class="mx-3 mx-md-0">Sollte ihr Testergebnis nach mehr als <strong>3 Werktagen nach Testabstrich</strong> nicht vorliegen:</h5>
        <ol class="text-left mx-2 mx-md-5">
            <li>Schreiben Sie eine E-Mail an covid19@doctorbox.eu</li>
            <li>Inkludieren Sie bitte die Barcodenummer, idealerweise ein Bild des Barcodes sowie Ihr Geburtsdatum</li>
            <li>Wir melden uns schnellstmöglich bei Ihnen zurück</li>
        </ol>
    </div>

</div>


                    </div>
                </div>
            </section>
            <!-- Ende Covid -->
...
```

요즘 유행과는 다르게 API 호출후 스크립트로 렌더링 하는 방식이 아니라 문서 자체에 결과가 찍혀 나오는 타입으로 보인다. 덕분에 HTML 구문 분석이 필요하게 되었다.

### Extract current status

대강 살펴봤을 때 `#covid19-2 h1 span` 을 살펴보면 현재 상황을 알 수 있을 것만 같다. 결과가 이미 나온 접수번호로 테스트 했을 때는, 저 자리에 `NEGATIV`가 들어 있는 것을 확인했다. 이제 위에서 찾았던 [pup][pup]을 이용해서 파싱해보자.

이 경우 `pup "#covid19-2 h1 span text{}"` 명령으로 파이프라이닝을 하면 될 것 같다. 때마침 [curl][curl]도 요청한 내용을 기본적으로는 stdout으로 내 보내 주니 궁합이 잘 맞다.

```bash
$ curl ... | pup "#covid19-2 h1 span text{}"
IN BEARBEITUNG
```

### Check the result and send notification

위에서 주어진 접수 번호에 대해 테스트 결과가 어떤지를 얻어올 수 있는 명령어를 만들었다. 이걸 변수에 저장한 뒤 비교해보고 판단해서 메시지를 보내는 것을 어떻게 할지 알아보자.

[Bash][bash] script에는 [Command substitution][bash-cmd-sub]이라는 기능이 있다. 쉽게 표현하자면 어떤 명령을 실행한 뒤 그 결과값으로 치환되는 식을 사용할 수 있다. 즉, 어떤 명령의 실행 결과를 다른 명령의 인자로 전달하거나 환경변수에 저장할 수 있다.

[Bash][bash] script로 의미있는 작업을 하려면 아무래도 어떤 식을 평가하거나, 평가한 값을 바탕으로 분기하는 기능이 반드시 있어야 할 것이다. 이 역할을 하는 명령어가 built-in 명령어인 [test][bash-test] 와 [\[\[][bash-test2]이다. [test][bash-test]는 [Bourne shell(sh)][sh] 계열 셸이 모두 지원하는 명령어지만 기능이 제한적이거나 복잡하고, [\[\[][bash-test2]는 [bash][bash] 기능이라 호환이 안되는 경우도 있지만 좀 더 기능이 편리하다. 여기에 `&&`이나 `||`으로 명령을 연결하면 간단한 `if-else` 느낌으로 쓸 수 있다.

[telegram-send][telegram-send]는 미리 설정을 해 놓은 다음 실행하면 주어진 인자를 미리 설정한 상대방에게 보내주는 간단한 사용법을 지닌 명령어이다. 같이 조합해보면 다음과 같다.

```bash
RESULT=$(curl ... | pup "#covid19-2 h1 span text{}")
[[ "$RESULT" = "IN BEARBEITUNG" ]] || telegram-send "$RESULT" && exit
```

`||` 은 다른 언어에서도 많이 볼 수 있는 논리합 연산자인데, [short-circuit evaluation][sce]을 지원한다. 즉, 앞 부분이 성공이면 뒷 부분은 실행하지 않고 성공으로 취급하고, 앞 부분이 실패라면 뒷 부분을 실행한 뒤 뒷 부분의 성공/실패 여부로 결과를 결정한다. 말인즉슨, `A || B` 는 `if not A then do B`의 의미를 가진다는 것이다. 마찬가지로 `A && B` 는 `if A then do B`가 될 것이다.

이 사실을 이용해서 해석해 보자면, `$RESULT`가 `IN BEARBEITUNG`이 아니라면 `telegram-send`를 수행하고, `telegram-send`가 성공하면 종료하라는 뜻이 되겠다.

[telegram-send][telegram-send]로 메시지를 보내려면 미리 설정을 해야 되는데, 다음처럼 하면 간단하게 설정이 가능하다.

```bash
$ telegram-send -c
Talk with the BotFather on Telegram (https://telegram.me/BotFather), create a bot and insert the token
❯ 0123456789:ABCDEFGHIJKLMNOPQRSTUVWXYZ012345678
Connected with telegram_bot.

Please add telegram_bot on Telegram (https://telegram.me/telegram_bot)
and send it the password: ####

Congratulations gwangyi!
telegram-send is now ready for use!
```

처음 묻는 부분에서는 [BotFather][botfather]를 통해 새로 생성한 봇의 토큰을 넣으면 된다. 토큰은 탈취당한 경우 다른 사람이 만들어 둔 봇으로 보내지는 메시지를 탈취하거나 봇으로 위장해서 메시지를 보낼 수 있기 때문에 안전하게 보관해야 한다.

입력하게 되면 앞에서 생성한 봇의 정보를 간단하게 보여준 뒤 숫자로 된 비밀번호 5자리를 보여준다. 지정한 봇을 텔레그램에서 친구추가 한 뒤 비밀번호 5자리를 입력하면 연결이 완료된다.

다음 명령어를 통해 제대로 설정되었는지 확인할 수 있다.

```bash
$ telegram-send "Hello"
```

명령을 수행한 뒤 텔레그램에서 지정된 봇으로부터 메시지를 수신한 것을 확인하면 다음으로 넘어갈 수 있다.

[bash]: https://www.gnu.org/software/bash/
[bash-test]: https://man7.org/linux/man-pages/man1/test.1.html
[bash-test2]: https://man7.org/linux/man-pages/man1/bash.1.html
[sh]: https://en.wikipedia.org/wiki/Bourne_shell
[sce]: https://en.wikipedia.org/wiki/Short-circuit_evaluation

### Make a loop

이제 할 일은, 결과가 나올 때까지 반복하는 일이다. 복잡하게 구성하면 머리만 아파지니 간단하게 무한루프를 만든 다음 조건이 맞으면 나오도록 만들자.

```bash
while true; do
  RESULT=$(curl ... | pup "#covid19-2 h1 span text{}")
  [[ "$RESULT" = "IN BEARBEITUNG" ]] || telegram-send "$RESULT" && exit
done
```

`while` 명령은 C언어에 있는 `while`과 비슷하게 특정 조건을 만족하는 동안 루프를 돌게 하는데 쓸 수 있다. 다만 두 가지 부분에서 C 언어와 다른데,

1. 중괄호 대신 `do ... done` 으로 블록을 감싼다.
2. `while` 뒤에 오는 명령을 실행시킨 뒤 종료 코드가 `0`일 때 내용을 수행하고 `0`이 아닐 때 밖으로 나간다.

1은 단순히 문법적인 이유이고, 2는 아마도 일반적인 경우 에러 없이 종료된 경우 종료 코드가 `0`인 것이 셸에서 통용되는 원칙이므로 저렇게 한 것 같다. 재미있는 점은 `true`는 일반적인 배포판에서 `/bin/true` 에 있는 독립적인 실행파일로, 실행하면 종료코드 `0`으로 종료하는 기능만 있는 바이너리라는 것이다. 그러면 `while` 조건이 항상 만족될테니, 무한루프를 간단히 작성할 수 있다.

### Full script without private info

```bash
#!/bin/bash

while true; do
  RESULT=$(curl 'https://www.doctorbox.de/covid19-result.jsp' \
    -H 'Connection: keep-alive' \
    -H 'Cache-Control: max-age=0' \
    -H 'sec-ch-ua: "Google Chrome";v="89", "Chromium";v="89", ";Not A Brand";v="99"' \
    -H 'sec-ch-ua-mobile: ?0' \
    -H 'Upgrade-Insecure-Requests: 1' \
    -H 'Origin: https://www.doctorbox.de' \
    -H 'Content-Type: application/x-www-form-urlencoded' \
    -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 11_2_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.72 Safari/537.36' \
    -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' \
    -H 'Sec-Fetch-Site: same-origin' \
    -H 'Sec-Fetch-Mode: navigate' \
    -H 'Sec-Fetch-User: ?1' \
    -H 'Sec-Fetch-Dest: document' \
    -H 'Referer: https://www.doctorbox.de/covid19.jsp' \
    -H 'Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,de;q=0.6' \
    -H 'Cookie: JSESSIONID=............................; ........................=.............; site-lang=de' \
    --data-raw 'covid19ID=..........&day=..&month=..&year=....' \
    --compressed 2> /dev/null | pup '#covid19-2 h1 span text{}')

  echo -n "$RESULT "; date

  [[ "$RESULT" == "IN BEARBEITUNG" ]] || telegram-send "$RESULT" && exit

  sleep 60 
done
```

## Conclusion

세상엔 다양한 워크플로우가 있고, 웬만한 워크플로우 구성 요소들은 나만 필요한게 아니라 다른 누군가도 필요하다. 그리고 그런 경우 대부분은 그 누군가가 만들어둔 것이 있다.

매우 특수한 상황이 아니라면 그런 워크플로우들을 잘 결합해보면 간단한 자동화 프로그램을 만드는 것이 어렵지 않다는 것을 알 수 있는 소 프로젝트였다.
