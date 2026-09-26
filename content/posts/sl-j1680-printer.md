---
title: SL-J1680 프린터를 네트워크 프린터로 만들기
date: 2026-09-19T23:54:00+0900
tags:
    - raspberry-pi
    - sl-j1680
    - cups
---

## Introduction

[SL-J1680][sl-j1680] 프린터는 몇만 원 정도에 살 수 있는 가성비 높은 프린터 중 하나다. 심지어 스캐너도 붙어 있는 복합기라 활용성은 배가 된다. 다만, 네트워크 프린트 기능이 없고 윈도 드라이버밖에 지원되지 않아 용처가 조금 제한되는 문제가 있긴 하다.

예전에 쓰던 HP DeskJet 프린터는 네트워크 프린트와 스캔 모두 잘 됐기 때문에 편리하게 잘 쓰고 있었는데, [다른 포스트](/posts/pdf-rotate/)에서 언급한 것과 같이 바꾸게 되면서 아쉬움을 많이 느끼던 차였다. 특히 데스크탑이 집에 없어서 노트북에 연결해 써야 하는데, 노트북은 들고다닐 일이 많아서 매일 꽂고 빼는게 너무 귀찮고, 공유해 쓰더라도 노트북이 켜져 있어야 하니 될 일이 아니었다. 하지만, 집에는 [Raspberry Pi][rpi] 보드가 하나 홈 서버로 돌고 있었고 거기 꽂아서 HP DeskJet의 능력을 다시 살려내봐야겠다는 생각이 들었다.

일반적인 리눅스 배포판에는 [CUPS][cups-overview]라는 서버가 설치되어 있고 이 서비스가 프린팅을 담당하도록 구성되어 있다. 특기할 만한 부분은 애플이 최초 개발자로부터 [CUPS][cups-overview]를 인수해서 맥의 기본 프린팅 서비스로 도입했었다는 부분이다. 뭐든 디테일에 집착하는 애플답게 [CUPS][cups-overview]는 macOS 뿐만 아니라 iOS의 기본 프린팅 서비스로 도입되며 무선 프린트 기능인 [AirPrint][airprint]와 드라이버 설치 없이 인쇄가 가능하게 하는 [IPP everywhere][ipp-everywhere] 기능을 지원한다. 즉, [SL-J1680][sl-j1680] 프린터를 잘 설정하면 공짜로 [AirPrint][airprint]를 지원하는 와이파이 지원 프린터가 된다는 것이다!

[sl-j1680]: https://www.samsung.com/sec/support/model/SL-J1680/
[rpi]: https://www.raspberrypi.com/
[cups-overview]: https://www.cups.org/doc/overview.html
[airprint]: https://support.apple.com/en-us/102895
[ipp-everywhere]: https://www.pwg.org/ipp/everywhere.html

## Setup

그러면 이를 이루기 위해 지금 가지고 있는 것과 뭐가 더 필요한지를 잘 따져볼 필요가 있다. SL-J1680 프린터는 삼성 프린터인데, 이쪽에 관심이 있는 사람이라면 [삼성의 프린터 사업부가 2017년에 HP에 매각된 사실][samsung-hp-printer]을 잘 알고 있을 것이다. 그 이후로 HP 제품에 브랜드만 삼성을 달고 나오는 프린터들이 있는데, SL-J1680이 그런 프린터 중 하나다. 그러다 보니 삼성 드라이버가 지원되지 않는 OS 사용자는 해당하는 HP 프린터 드라이버를 억지로 설치하면 사용할 수 있게 되는 경우가 많다. 사실 업무용 맥북을 주로 사용하던 시기에도 비슷한 방식으로 드라이버를 설치해 썼었다. 그래서 SL-J1680에 해당하는 드라이버는 [HP DeskJet 2130 Series][hp-deskjet-2130]이다. 사진을 보면 알겠지만 똑같이 생겼다.

<div style="max-width: 108%; margin-left: -3.8%; display: flex;">
    <div style="width: 50%; display: flex; align-items: center;"><img src="/img/sl-j1680.png" alt="SL-J1680"/></div>
    <div style="width: 50%; display: flex; align-items: center;"><img src="/img/hp-deskjet-2130.png" alt="HP DeskJet 2130"/></div>
</div>

왼쪽이 삼성, 오른쪽이 HP 프린터다. 그럼 이제 이걸 가지고서 [CUPS][cups-overview]에 등록해 보도록 하자.

1. 필요한 HP 드라이버를 설치한다.
```bash
sudo apt update && sudo apt install -y hplip printer-driver-hpcups
```
2. 드라이버의 URL을 확인한다.
```bash
sudo lpinfo -m | grep -Ei "^drv.*hpcups.*deskjet.*2130"
```
3. 장치의 URL을 확인한다.
```bash
sudo lpinfo -v | grep -Ei "^direct usb://"
```
4. 프린터를 추가한다. 위에서 나온 정보를 활용해서 명령을 만들면 된다.
```bash
sudo lpadmin -p SL-J1680 -E -v usb://Samsung/SL-J1680%20series?serial=xxxxxxxxxx&interface=1 -m drv:///hpcups.drv/hp-deskjet_2130_series.ppd
```

내 경우는 시리얼을 제외하고 위와 같았는데, 명령어를 만드는게 어렵다면 다음 명령으로 한번에 실행할 수 있다. 다만 2, 3번 명령에서 항목이 하나만 나오는지 확인하고 실행하는 것이 좋다.
```bash
sudo lpadmin -p SL-J1680 -E \
    -o printer-is-shared=true \
    -v $(sudo lpinfo -v | grep -Ei '^direct usb://' | awk '{ print $2 }') \
    -m $(sudo lpinfo -m | grep -Ei '^drv.*hpcups.*deskjet.*2130' | awk '{ print $1 }')
```

여기서 `Printer drivrs are deprecated and will stop working in a future version of CUPS.` 라는 경고가 뜰 수 있다. CUPS 3.0부터는 [IPP Everywhere][ipp-everywhere] 네트워크 프린터 지원만 남기고 PPD 방식이 지원되지 않을 예정이라 나오는 문구로, [hplip-printer-app][hplip-printer-app]과 같은 별도의 [IPP Everywhere][ipp-everywhere] 어댑터가 필요해질 것이다. 이건 다음 기회에 다루어 보도록 하자. 짧게 테스트해본 결과 arm64용 docker image는 제공되지 않아서 간단하게 사용할 수 있을 것 같지는 않다..

[samsung-hp-printer]: https://news.samsung.com/kr/%EC%82%BC%EC%84%B1%EC%A0%84%EC%9E%90-%EC%97%90%EC%8A%A4%ED%94%84%EB%A6%B0%ED%8C%85%EC%86%94%EB%A3%A8%EC%85%98-%EA%B3%B5%EC%8B%9D-%EC%B6%9C%EB%B2%94
[hp-deskjet-2130]: https://support.hp.com/kr-ko/drivers/hp-deskjet-2130-all-in-one-printer-series/model/7174551
[hplip-printer-app]: https://github.com/OpenPrinting/hplip-printer-app

## Test print

CLI로 테스트 프린트를 하는 방법도 있겠지만, 설정한 김에 웹 인터페이스를 사용해보도록 하자. CUPS web interface의 포트번호는 631번을 기본값으로 사용한다. 방금 프린터 이름읗 SL-J1680으로 생성했으므로 다음 주소로 프린터 관리 페이지에 접근할 수 있을 것이다.

* http://127.0.0.1:631/printers/SL-J1680

만약 Raspberry Pi에서 접속하는게 아니라면 127.0.0.1 대신 Raspberry Pi의 IP를 사용하자. Maintenance 드롭박스를 눌러서 Print Test Page를 선택하면 바로 인쇄된다.

![Test Page](/img/testpage.png)

이런 페이지가 나왔다면 다 됐다! 위에서 shared 설정도 했으므로 다른 컴퓨터나 심지어 스마트폰에서도 자동으로 프린터가 보일 것이다. 안 보인다면 ipp 방식으로 IP 입력하면 인쇄가 가능하다.

## Conclusion

사실 많은 부분을 [Gemini][gemini] 도움을 받아 수행했다. 시키는 대로 해보고 그 결과에 대한 감상을 공유하는 문서나 다름없다. 그래도 이렇게 기록해 두면, 나중에 다시 할 때 참고할 만한 자료로 쓸 수 있지 않을까?

그리고 다음은 스캐너도 네트워크로 가능하도록 만들어야 한다. 그리고 나서 시간이 남으면 [hplip-printer-app][hplip-printer-app]을 다뤄보도록 하자.
