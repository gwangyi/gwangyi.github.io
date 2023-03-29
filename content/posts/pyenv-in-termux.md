---
title: pyenv를 termux에서 써보자 (1)
date: 2019-01-12
tags:
  - pyenv
  - termux
  - bash
  - script
  - python
series: pyenv-in-termux
isCJKLanguage: true
---

__수정__
- 오타 수정: `xarg` -> `xargs`
- 필요한 패키지 설치 항목 추가

## Introduction

[Termux][termux]는 훌륭한 유사-리눅스 환경을 안드로이드 장치 위에서 제공해준다. 단순한 유틸리티 뿐만 아니라, 컴파일러와 오픈소스 라이브러리의 소스코드까지도 함께 제공받을 수 있다. 그러한 배경에서, 안드로이드 장치 위에서 파이썬을 설치해서 다양한 장난감을 만들어 보는 것 또한 즐거운 일일 것이다.

[Termux][termux]는 컴파일러를 제공하는 것처럼 파이썬 또한 미리 빌드한 바이너리 형태로 제공하고 있다. 하지만 설치할 수 있는 파이썬 패키지는 2.x중에서 하나, 3.x중에서 하나 뿐으로 그나마도 교체해가면서 쓰기에는 불편하기 짝이 없다. 일반적인 경우에야 하나만 있어도 별 일이 없겠지만, 우리는 항상 만의 하나를 가정하고 살아야 하므로 파이썬 버전 관리자를 사용해 보도록 하자. 이 포스팅에서 다루고자 하는 파이썬 버전 관리자는 [pyenv][pyenv]로, 파이썬 버전 뿐만 아니라 [Virtualenv][venv]까지도 관리해줄 수 있어 매우 편리하다.

[termux]: https://termux.com/
[pyenv]: https://github.com/pyenv/pyenv
[venv]: https://virtualenv.pypa.io/en/latest/

## Background

어떤 프로그래밍 언어의 구현체가 있다면, 그 또한 하나의 프로그램이며, 버그나 불편함 등으로 인하여 정기적으로 수정되고 재배포되는 것은 필언적이다. 심지어 C 언어와 같은 역사가 오래된 언어들은, 느슨한 형태의 표준 명세를 컴파일러를 만드는 회사마다 조금씩 다른 형태로 구현해버려 특정 컴파일러를 염두해 두고 작성한 소스코드는 다른 컴파일러에서는 컴파일이 안되거나 오동작까지 하기도 한다.

파이썬은 기준이 되는 구현체가 있고 [BDFL][BDFL] - 지금은 사임했지만 - 에 의해 중앙집권적으로 표준이 관리되어 왔기 때문에 앞서 언급한 C와 같은 파편화는 없지만, 그럼에도 불구하고 다양한 feature가 추가되는 과정에서 호환이 깨지거나, 기준 구현체가 아닌 독특한 구현체를 필요로 할 경우가 종종 있을 수 있다.

거기에 더해서, [node.js][nodejs]와는 달리 기본적으로 모든 패키지는 전역적인 형태로 설치되며, 사용자 단위로 설치한다 해도 해당 사용자 입장에서는 전역적으로 설치되기 때문에 패키지들의 버전 관리가 어려운 점도 있다. 이를 해결하기 위해 [Virtualenv][venv] 프로젝트도 있으나, virtualenv들을 관리하고 activate/deactivate 하는 작업이 지난하고 귀찮다는 단점이 있다.

이전부터 버전간 호환성이 나쁘기로 유명했던 [ruby][ruby]의 경우, 꽤 빠른 시기부터 다양한 버전의 [ruby][ruby] 구현체를 설치하고 이를 교체해가며 쓸 수 있는 프로젝트들이 제안되어 왔다. [RVM][rvm]이 매우 유명하며, 여기로부터 영향을 받아 [node.js][nodejs] 진영에서는 [nvm][nvm]이 만들어졌다. [RVM][rvm]의 [한계][why-rbenv]를 개선하기 위한 [rbenv][rbenv] 프로젝트도 있다.

이번 포스팅에서 다루고자 하는 [pyenv][pyenv]는 이름으로부터 [rbenv][rbenv]로부터 영향을 받은 프로젝트임을 알 수 있다. [pyenv][pyenv]는 기본적으로 다양한 파이썬 구현체를 설치하는 과정을 자동화하여 루트 권한 없이도 간단하게 다양한 구현체의 파이썬을 설치할 수 있게 해 주고, 사용자 전역, 디렉토리, 셸 세션 단위로 사용할 파이썬 버전을 지정해줄 수 있는 기능을 갖고 있다. 또한 간단한 확장을 설치하면 [virtualenv][venv]도 다른 파이썬 버전들을 다룰때와 같이 간단하게 생성, 활성화, 비활성화 시킬 수 있어 편리함을 증진시켜준다.

[pyenv][pyenv]와 비슷한 용도로 사용할 수 있는 [pipenv][pipenv]라는 프로젝트도 있다. [Pipenv][pipenv]는 [node.js][nodejs]와 비슷하게 프로젝트 단위 - 실질적으로는 디렉토리 단위 - 로 가상환경을 생성하고 관리할 수 있게 해 준다. 버전도 지정이 가능하지만, 설치되지 않은 파이썬 버전을 지정한 경우 [pyenv][pyenv]를 통해 버전 관리를 하게 된다. 또, [pipenv][pipenv]는 shim 관리를 해주지 않아 [pipenv][pipenv]를 통해 스크립트를 실행해야하는 차이도 있다.

[pyenv][pyenv]가 좋은 툴인 것은 확실하지만, [Termux][termux] 위에서 사용할 때는 몇 가지 문제가 발생한다. 표준 리눅스 환경을 가정하고 빌드 스크립트가 구성되어 있어, 파이썬 설치시 다양한 문제점이 발생한다. 게다가, 안드로이드에서 기본적으로 설치되어 있는 [BusyBox][busybox]는 표준 GNU 유틸리티와 미묘한 차이가 존재하여, 빌드 스크립트의 오동작이 잦아 여러 문제점을 일으켜 빌드하는 과정이 순탄치가 못하다.

이번 포스팅의 목표는, [pyenv][pyenv]를 [Termux][termux] 위에 설치한 뒤 최신 CPython 구현체를 설치하여 실행해 보는 것을 목표로 한다.

[BDFL]: https://en.wikipedia.org/wiki/Benevolent_dictator_for_life
[nodejs]: https://nodejs.org/en/
[ruby]: https://ruby-lang.org/
[rvm]: https://rvm.io/
[nvm]: https://github.com/creationix/nvm
[why-rbenv]: https://github.com/rbenv/rbenv/wiki/Why-rbenv%3F
[rbenv]: https://github.com/rbenv/rbenv
[pipenv]: https://github.com/pypa/pipenv
[busybox]: https://busybox.net/

## Installation

[pyenv][pyenv]는 다양한 방법으로 설치해볼 수 있지만, 이 중에서 가장 편리한 것은 [pyenv-installer][pyenv-installer]를 사용하는 것이다. 그냥 스크립트만 실행하면 [pyenv][pyenv] 설치가 한방에 끝난다.

```bash
curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
```

관리자 권한이 필요 없고, 관리자 권한으로 실행하지 않기를 강력하게 추천하는 바이다. 실행이 끝나면, [pyenv][pyenv]를 정상적으로 사용하기 위한 추가 작업의 내용을 간단하게 설명해주므로 따라서 설정해주도록 하자. [bash][bash]를 사용중이라면 `$HOME/.bashrc`에, [zsh][zsh]를 사용중이라면 `$HOME/.zshrc`에 [pyenv][pyenv] 명령을 등록하는 명령을 추가하자. 다음과 비슷한 코드를 입력해야 할 것이다. 만약 [pyenv][pyenv] 버전이 많이 달라져서 안내된 명령이 아래 명령과 다르다면, 안내된 대로 수정하는 것이 좋다.

```bash
export PATH="~/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

`.bashrc` 파일을 수정했으므로 셸을 다시 열도록 하자.

추후에 [pyenv][pyenv]를 업그레이드 할 일이 생기면 다음 명령으로 업그레이드할 수 있다.

```bash
pyenv update
```

[pyenv-installer]: https://github.com/pyenv/pyenv-installer
[bash]: https://www.gnu.org/software/bash/
[zsh]: https://www.zsh.org/

## Install Python

[pyenv][pyenv] 설치는 간단하지만, 이를 통해 파이썬을 빌드하는 것은 간단하지 않다. 앞에서 말했듯 [Termux][termux] 환경은 완전한 리눅스 환경이 아니라서 하드코딩된 디렉토리 패스와 같은 것들이 빌드 과정에서 문제를 지속적으로 일으킨다. 그러나 [Termux][termux]에 파이썬이 포팅되지 않은 것이 아니므로, [Termux][termux]의 패키지를 어떻게 관리하고 있는지 먼저 찾아볼 필요가 있다.

### Install dependencies

[Termux][termux]의 패키지는 [termux-packages][termux-packages] 리포지터리를 통하여 관리되고 있다. 해당 깃헙 리포지터리에 가 보면 현재 [Termux][termux]에서 지원하는 다양한 패키지들에 대한 패치와 빌드 스크립트를 확인할 수 있다. 그 중에 [파이썬 패키지][termux-python]를 보면 [Termux][termux] 환경에서 파이썬을 빌드하기 위해 필요한 부분들을 확인할 수 있다.

먼저 `TERMUX_PKG_DEPENDS` 항목을 보면 termux의 python 패키지가 의존하는 패키지 목록이 나온다. [termux 정책 변화][termux-no-more-develsplit]로 인해 별도의 `-dev` 패키지가 없어졌기 때문에, 목록에 있는 패키지를 먼저 설치해주자.

```bash
pkg in build-essential gdbm libandroid-posix-semaphore libandroid-support libbz2 libcrypt libexpat libffi liblzma libsqlite ncurses ncurses-ui-libs openssl readline zlib
```

[termux-no-more-develsplit]: https://github.com/termux/termux-packages/pull/4071

### Obtain Patch

[pyenv][pyenv]는 빌드 과정이 모두 자동화되어 있어 중간에 소스코드를 수정할 시간은 없다. 대신, [patch][patch] 명령을 이용하여 패치를 적용하는 기능을 가지고 있다. 일반적인 [pyenv][pyenv]를 이용한 파이썬 설치는 `pyenv install 3.11.2` 와 같이 진행되지만, 패치를 먹이는 작업을 함께 수행한다면 다음과 같이 명령하면 된다.

```bash
cat python.patch | pyenv install -p 3.11.2
```

`-p` 옵션을 주게 되면, 표준 입력으로부터 패치를 받아 적용하는 과정을 추가적으로 거치게 된다. 이 때 [patch][patch] 명령에는 `-p 0` 옵션으로 전달되기 때문에 패치 파일 내부의 파일 주소를 적절하게 조정해 줄 필요가 있다. 이전에는 환경변수로 [patch][patch]의 옵션을 지정할 수 있었던 것 같지만, 지금은 그렇지 않다.

여기까지 내용을 요약하면 다음과 같다.

1. [termux-packages][termux-packages]의 [파이썬 패키지][termux-python]의 패치 목록을 취득
2. 취득한 패치 목록의 내용을 모두 이어붙임
3. 이어붙인 패치를 `pyenv install -p` 명령의 표준 입력으로 인가

이를 위해 필요한 것은

1. 리포지터리의 디렉토리 내용 조회
2. 조회된 내용으로부터 패치 내용 취득

두 가지 기능이 필요할 것이다. 각각은 다음과 같이 얻어낼 수 있다.

1. GitHub API 이용 [Contents/Get contents][github-api-get-contents]
2. 1의 결과는 json 형태이므로 이를 [jq][jq]로 파싱
3. 결과를 [xargs][xargs]에 전달하여 각각 내려받은 후 표준 출력으로 내보내기

http 다운로드를 위해서는 [curl][curl]이나 [wget][wget]을 쓸 수 있을텐데, 개인적으로 [wget][wget]이 더 편하므로 `pkg in wget`을 통해 설치하도록 하자. wget은 [BusyBox][busybox]에 내장되어 있기는 하나, 내장된 버전은 https를 처리하지 못하므로 따로 설치하는 것이 좋다.
[jq][jq]는 [Termux][termux]에서 미리 빌드된 패키지로 설치 가능하다. `pkg in jq` 로 설치해두자.

```bash
pkg in wget jq
```

먼저 json 파일의 구조를 살펴보기 위해 다음과 같이 실행해보자.

```bash
wget -O - https://api.github.com/repos/termux/termux-packages/contents/packages/python -q
```

특정 커밋이나 태그, 브랜치를 지정하고 싶으면 python 뒤에 `?ref=...` 형태로 지정하면 된다. 이 때 결과물은 다음과 비슷할 것이다.

```json
[
  {
    "name": "Lib-subprocess.py.patch",
    "path": "packages/python/Lib-subprocess.py.patch",
    "sha": "bea4bbf23358e4b6d64a59f659b206d3201c06e2",
    "size": 663,
    "url": "https://api.github.com/repos/termux/termux-packages/contents/packages/python/Lib-subprocess.py.patch?ref=master",
    "html_url": "https://github.com/termux/termux-packages/blob/master/packages/python/Lib-subprocess.py.patch",
    "git_url": "https://api.github.com/repos/termux/termux-packages/git/blobs/bea4bbf23358e4b6d64a59f659b206d3201c06e2",
    "download_url": "https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/Lib-subprocess.py.patch",
    "type": "file",
    "_links": {
      "self": "https://api.github.com/repos/termux/termux-packages/contents/packages/python/Lib-subprocess.py.patch?ref=master",
      "git": "https://api.github.com/repos/termux/termux-packages/git/blobs/bea4bbf23358e4b6d64a59f659b206d3201c06e2",
      "html": "https://github.com/termux/termux-packages/blob/master/packages/python/Lib-subprocess.py.patch"
    }
  },
  ...
]
```

디렉토리이기 때문에 배열 형태로 반환되며, 각 아이템은 디렉토리에 있는 파일들에 대한 정보가 들어있다. 여기서 `name` 속성이 파일 이름이며, `download_url` 속성의 값으로 요청하면 파일의 내용을 얻어올 수 있다. 패치를 모두 엮기 위해서는,

1. `name`이 `.patch`로 끝나는 아이템들을 골라낸 뒤
2. 아이템에서 `download_url` 을 추출한다.

실제 [termux 패키지 빌드 스크립트][termux-packages]를 보면 `.patch` 파일과 `.patch32|64` 파일을 코어 bit수에 맞추어 선택하여 모두 사용하지만, 파이썬에는 `.patch` 파일만 있어 이것만 사용하였다.

[jq][jq]의 쿼리를 작성하려면,

1. 모든 아이템에 대해 반복하기 위해 첫 번째는 `.[]`이 된다.
2. 이 중에 특정 조건을 만족하는 아이템만 뽑아야 하므로 `|`로 `select()`를 연결해야 된다.
3. `select()`의 조건은 `name` 속성을 추출한 뒤 `endswith()`로 맨 뒷 부분을 확인해야 한다. 즉, `.name|endswith(".patch")`가 조건이 된다.
4. 필터링 한 후에는 그 안에서 `download_url`을 추출해야 하므로 `|`로 `.download_url`을 연결하면 최종적으로 쿼리가 완성된다.

완성된 최종 쿼리는 다음과 같다.

```bash
wget -O - https://api.github.com/repos/termux/termux-packages/contents/packages/python -q | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url'
```

결과는 다음과 같이 나올 것이다.

```json
"https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/Lib-subprocess.py.patch"
"https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/Lib-tmpfile.py.patch"
"https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/Modules-socketmodule.c.patch"
"https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/_cursesmodule.c.patch"
"https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/configure.patch"
"https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/cryptmodule.c.patch"
"https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/fix-paths.patch"
"https://raw.githubusercontent.com/termux/termux-packages/master/packages/python/setup.py.patch"
```

이제 이 각각의 url으로 wget을 실행해주면 연속해서 표준 출력으로 출력할 수 있다. [xargs][xargs]를 이용하자.

```bash
wget -O - https://api.github.com/repos/termux/termux-packages/contents/packages/python | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q
```

결과는 다음과 비슷하게 나올 것이다.

```diff
diff -u -r ../Python-3.7.1/Lib/subprocess.py ./Lib/subprocess.py
--- ../Python-3.7.1/Lib/subprocess.py   2018-10-20 06:04:19.000000000 +0000
+++ ./Lib/subprocess.py 2018-10-20 20:17:50.157206343 +0000
@@ -1389,9 +1389,7 @@
                 args = list(args)

             if shell:
-                # On Android the default shell is at '/system/bin/sh'.
-                unix_shell = ('/system/bin/sh' if
-                          hasattr(sys, 'getandroidapilevel') else '/bin/sh')
...
```

여기서 문제는, 패치 적용시 `@TERMUX_PREFIX@`와 같은 교체해야하는 스트링이 조금 포함되어 있다는 것과, Termux에서 사용하는 패치는 `-p 1` 옵션을 상정하고 만들어졌다는 점이다. 이를 해결하기 위해서는 [sed][sed] 를 사용하면 편리하다.

```bash
wget -O - https://api.github.com/repos/termux/termux-packages/contents/packages/python | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q | \
         sed "s%@TERMUX_PREFIX@%$PREFIX%g; s%^+++ [^/ ]*%+++ .%"
```

빌드 스크립트에는 `@TERMUX_HOME@` 등의 치환자가 더 있지만, 파이썬에서는 사용하지 않기 때문에 `@TERMUX_PREFIX@`만 변경했고, 바뀌어야되는 값은 환경변수 `PREFIX`이다. 그리고 패치의 대상 파일 경로는 `+++`으로 시작하는 행에 나타나므로, 첫 번째 디렉토리 부분을 `./`으로 교체하여 `-p 1` 옵션을 흉내내었다.
이제 이 것을 `pyenv install -p`에 집어넣어 설치해보면, 뭔가가 진행되다 실패하는 것을 볼 수 있다. 패치에 실패하는 것은 [BusyBox][busybox]의 `patch` 애플릿의 기능이 딸려서이므로, `pkg in patch`를 통해 제대로 된 GNU patch 툴을 설치한 뒤에 재시도하면 된다.

```bash
wget -O - "https://api.github.com/repos/termux/termux-packages/contents/packages/python?ref=4f87b638eb93d17b89c499c3ac7d4034d5e1ac5f"    | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q | \
         sed "s%@TERMUX_PREFIX@%$PREFIX%g; s%^+++ [^/ ]*%+++ .%" | \
         pyenv install -p -v 3.11.2
```

참고로 `pyenv install`에 `-v` 옵션을 주면 빌드 과정을 볼 수 있다. 일반적인 경우는 굳이 쓸 필요가 없지만, 우리처럼 제대로 안되는 환경에서 어거지로 돌려야 되는 경우에는 에러 메시지를 바로바로 볼 수 있게 해 두는 편이 문제 파악하기에 조금 더 편리하므로 계속 넣도록 하자. 물론 `-v` 옵션을 주지 않았다고 해서 로그를 볼 수 없는 것은 아니다. temp 디렉토리를 찾아보면 `python-build`로 시작하는 디렉토리와 로그 파일을 발견할 수 있다.

그리고 `ref=` 뒤에 오는 해시값은 커밋의 해시값이다. 지금 넣어놓은 값은 [`python: Bump to 3.11.2`][termux-python-3.11.2] 커밋인데, 추후에 새로운 python 버전이 나와도 안정적으로 빌드할 수 있게 하기 위해 붙여두었다. 다른 버전의 python을 빌드하고 싶다면 떼거나 적절한 해시값으로 변경하도록 하자.

이제부터는 플래그를 설정하고, 요구되는 라이브러리를 찾고, 빌드 과정의 소소한 문제들을 찾아가면서 파이썬을 설치할 수 있게 고쳐갈 것이다.

[termux-packages]: https://github.com/termux/termux-packages
[termux-python]: https://github.com/termux/termux-packages/tree/master/packages/python
[termux-python-3.11.2]: https://github.com/termux/termux-packages/tree/4f87b638eb93d17b89c499c3ac7d4034d5e1ac5f
[patch]: http://savannah.gnu.org/projects/patch/
[github-api-get-contents]: https://developer.github.com/v3/repos/contents/#get-contents
[curl]: https://curl.haxx.se/
[wget]: https://www.gnu.org/software/wget/
[jq]: https://stedolan.github.io/jq/
[xargs]: http://man7.org/linux/man-pages/man1/xargs.1.html
[sed]: https://www.gnu.org/software/sed/

## Future work

설치를 다 못했는데 여기서 자르니 매우 찝찝하지만, 분량이 너무 커진 관계로 나머지는 다음 포스팅에 담도록 하겠다.

다음 포스팅에서는 컴파일·링크에 필요한 플래그를 지정하고, 빌드에 필요한 라이브러리 패키지들을 소개하고, 파일 시스템의 한계로 인한 버그를 잡아보도록 하겠다.
