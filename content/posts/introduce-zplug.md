---
title: zplug 소개
date: 2018-12-01 12:01:00
tags:
  - zsh
  - zplug
isCJKLanguage: true
---

## Introduction

일반적으로 리눅스 환경을 처음 접하게 되면 [bash][bash] 셸을 접하는 경우가 대다수이다. 좀 오래된 환경을 사용한다면 가뭄에 콩나듯 [csh][csh]을 볼 수 있는데, 최근에 csh을 접했다면 실제로는 [tcsh][tcsh]일 가능성이 높다. 이러한 셸들은 오래 널리 사용되어 왔기 때문에 방대한 양의 참고자료와 수많은 보조 도구들이 이미 존재하고 있다.

하지만 그러한 자료들로도 부족함을 느꼈던 사람들은 여러가지 대안 셸들을 제안했고, 그 중 [zsh][zsh]는 강력한 interactive 기능과 높은 프로그래머블한 특성을 앞세워 많은 얼리 어답터들을 포섭하는데 성공했다.

높은 프로그래머블 특성은 [zsh][zsh] 위에서 동작하는 수많은 플러그인들을 낳았고, 이런 플러그인을 관리하는 일이 어려워짐에 따라 다양한 플러그인 매니저들이 등장했다. 그 중에서도 이 문서에서는 [zplug][zplug]를 소개하고자 한다.

[bash]: https://www.gnu.org/software/bash/
[csh]: https://en.wikipedia.org/wiki/C_shell
[tcsh]: https://github.com/tcsh-org/tcsh
[zsh]: http://www.zsh.org/
[zplug]: https://github.com/zplug/zplug

## Background

[bash][bash]의 계보나 [tcsh][tcsh]의 계보는 재미있는 옛날 이야기 정도이니 검색해서 다른 곳에서 찾는 것이 빠를 것이라 생각해 생략하도록 하겠다. [bash][bash]는 interactive shell에서 강점을 갖고, [tcsh][tcsh]는 shell scripting에 강점을 갖고 있다는 특징이 있어 양 쪽의 장점을 취하고자 하는 사람들은 항상 있어왔다.

[zsh][zsh]는 [bash][bash]를 원형으로 삼아 만들어진 셸로, [bash][bash]와 호환성이 뛰어나고 다양한 부분을 script를 통해 커스터마이즈할 수 있는 특징이 있다. [Shell 별 비교](https://en.wikipedia.org/wiki/Comparison_of_command_shells) 페이지를 보면 다른 셸들과 비교한 표를 볼 수 있다.

아래 내용들은 [awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins) 페이지를 참고하여 작성하였다.

[zsh][zsh]는 다양한 플러그인들이 개발되어 있는데, 이런 쪽으로 제일 기본이 되는 [oh-my-zsh][oh-my-zsh]는 유용한 플러그인을 모아 번들 형태로 제공하고 있다. 이 후에 나오는 대부분의 플러그인 매니저는 [oh-my-zsh][oh-my-zsh]는 기본적으로 지원하는 경우가 대부분이다.

zsh Plugin Manager는 [antigen][antigen]이 가장 기초가 되는 매니저이다. 많은 플러그인 매니저들이 [antigen][antigen]의 개선판이라는 이름을 달고서 나오고 있는 실정이다. [Antigen][antigen] 스스로는 vim+pathogen과 비슷한 컨셉으로 관리한다고 하며, [vim](https://www.vim.org)에 [Vundle](https://github.com/VundleVim/Vundle.vim)이 있다면 [zsh][zsh]에는 [antigen][antigen]이 있다고 표현하고 있다.

[zgen][zgen]은 개인적으로 [antigen][antigen]을 쓰다가 느린 속도 때문에 갈아탔었던 플러그인 매니저이다. 최소한의 오버헤드를 가지도록 설계되어 있으며, 플러그인을 불러오는 모듈을 정적으로 생성해두고 변경이 발생하면 수동으로 업데이트해야되는 구조를 가지고 있다.

[oh-my-zsh]: https://ohmyz.sh/
[antigen]: http://antigen.sharats.me/
[zgen]: https://github.com/tarjoilija/zgen

## Zplug

나는 [zplug][zplug]를 사용하고 있는데, 사용하게 된 이유는 몇 가지가 있다. [zgen][zgen]도 좋기는 하지만, 회사의 특수한 환경에서 사용하기가 조금 불편하여 [zplug][zplug]로 옮겨타게 됐다. [zplug][zplug]의 특장점은 몇 가지가 있다.

1. 빠른 플러그인 설치
  일단 플러그인 설치가 병렬적으로 이루어져 빠르게 설치할 수 있다는 장점이 있다.
2. 직관적인 사용법
  `.zshrc` 파일에 `zplug` 명령으로 플러그인들을 추가할 수 있는데, github에 있는 플러그인이라면 간단하게 추가할 수 있다. [oh-my-zsh][oh-my-zsh]나 [prezto][prezto]같은 플러그인 번들에 포함된 플러그인도 태그를 이용해 간단하게 추가할 수 있다. 이외에도 gist를 비롯한 일반적이지 않은 source를 사용할 수 있도록 커스터마이즈 할 수도 있다.
3. Caching 지원
  캐싱을 통해 플러그인 로딩 속도를 높일 수 있다.

이 외에도 [zplug][zplug] 깃헙 저장소의 README를 보면 다양한 장점들을 제안하고 있는데, 언급되어 있지 않은 최고의 장점이 하나가 더 있어 택하게 되었다. 바로 플러그인의 repository를 관리하는 디렉토리를 별도로 지정할 수 있다는 것. 왜 이 것이 장점이 되냐면, 개인 계정의 용량에 제한이 크게 가해져 있어 공용계정 스토리지를 사용해야 되는 경우 플러그인 저장 장소를 바꿔줌으로써 개인 계정 용량을 관리할 수 있고, 때로는 플러그인 repository 관리용 디렉토리를 아예 공유하여 사용함으로써 환경 관리 및 설정 비용을 최소화할 수도 있기 때문이다.

## How to install zplug

인터넷이 되는 환경에서의 설치는 매우 간단하다.

```bash
curl -sL --proto-redir -all,https https://raw.githubusercontent.com/zplug/installer/master/installer.zsh | zsh
```

되지 않는 경우에는 zplug repository를 다운받아서 옮겨넣는 방식으로 설치할 수 있다.

```
https://github.com/zplug/zplug/archive/master.zip
```

위의 압축파일을 받아 풀면 `zplug-master` 디렉토리가 생길텐데, 이걸 `$HOME/.zplug`로 옮겨놓고 `init.zsh`를 source하면 된다.

```
unzip master.zip
mv zplug-master $HOME/.zplug
echo 'source $HOME/.zplug/init.zsh' >> ~/.zshrc
echo `zplug load' >> ~/.zshrc
```

## Useful plugins

[zsh][zsh] 플러그인 중에서 특별히 추천할만한 플러그인은 다음과 같다.

### Oh-my-zsh

Oh my zsh 전체가 필요한 것은 아니지만, [zgen][zgen]과는 다르게 [zplug][zplug]를 사용할 경우 lib 디렉토리에 들어있는 기능들이 자동적으로 활성화가 안되는 것 같다. 문제는 되지 않는 것이, `zplug load` 명령으로 불러올 수 있다. [zgen][zgen]과의 차이는, [zgen][zgen]은 lib 디렉토리 내의 기능들이 선택의 여지 없이 한번에 활성화되지만 [zplug][zplug]는 활성화를 선택할 수 있다는 점이다.
lib 디렉토리를 포함하여 oh my zsh는 여러 플러그인들을 가지고 있는데, 태그로 `from:oh-my-zsh`를 표시해주면 oh my zsh의 플러그인들을 선택적으로 불러올 수 있다. 그 때 플러그인 이름에는 oh my zsh 리포지터리 기준으로 상대 경로를 지정해주면 된다. 예를 들면 다음과 같다.

```bash
zplug "lib/key-bindings", from:oh-my-zsh

zplug "plugins/git", from:oh-my-zsh
```

lib 디렉토리 안의 플러그인들은 간단하면서 사용성을 크게 향상시켜주는 것들이 많기 때문에 대부분 사용하는 것을 추천한다. 좀 더 자세한 내용은 [github](https://github.com/robbyrussel/oh-my-zsh)를 참조하자.

### Autosuggestion

셸을 쓰다보면 비슷한 명령이 들어간 미묘하게 다른 커맨드들을 쓸 일이 많다. 그 명령이 길기라도 하면 내용이 기억이 안나니 위 아래 화살표로 히스토리를 뒤지고 있거나, `history` 명령에 `grep`을 걸어서 보는 수 밖에 없었다.
[fish][fish] 셸이 등장한 이후, 여기서 선보인 autosuggestion 기능을 다른 셸에 도입하는 시도들도 있었는데, [zsh][zsh]도 당연히 그런 플러그인들이 존재한다.

```bash
zplug "zsh-users/zsh-autosuggestions"
```

[github](https://github.com/zsh-users/zsh-autosuggestions)으로부터 바로 가져오면 되니 특별한 태그가 필요하지 않다. 이걸 사용하면, 사용 기록을 바탕으로 추천해줄만한 전체 명령을 어두운 색으로 미리 표시해줄 것이다.
그런데 그냥 쓰기에는 한가지 단점이 있는데, 바로 [solarized][solarized] 테마를 쓰면 배경색과 추천한 내용의 어두운 색이 겹쳐 글자가 잘 안보인다는 점이다.

```bash
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=23"
```

위 설정값을 `zplug load` 밑에 두면 살짝 밝은 색으로 나와서 보기에 조금 편리해진다.

[fish]: https://fishshell.com

### Syntax highlighter

이 기능도 역시 [fish][fish] 셸에서 유래된 것인데, 셸에 치는 명령어를 컨텍스트에 맞게 색을 칠해서 구분하기 쉽게 해준다. 명령어가 존재하는지 여부나 파일이 있는지, 파일이 디렉토리인지 등의 정보를 커맨드를 치면서 실시간으로 확인할 수 있어 매우 편리하다.

```bash
zplug "zdharma/fast-syntax-highlighting"
```

[github](https://github/zdharma/fast-syntax-highlighting) 저장소를 참조하면 자세한 내용을 알 수 있다. 사족으로, 원래 [zsh-users 에 등록된 highlighter](https://github.com/zsh-users/zsh-syntax-highlighting)을 썼었는데, 이게 더 빠르다고 해서 바꿨다. 바꿔보니 좀 더 다양한 highligting을 지원하는 것 같아 만족스럽다.

### Awesome-zsh-plugins

[zsh][zsh] 플러그인도 역시 [awesome-zsh-plugin](https://github.com/unixorn/awesome-zsh-plugins) 같은 페이지가 있다. 다양한 플러그인들이 모여 있으니 참고해서 좋은 툴을 사용하면 사용성을 많이 증대시킬 수 있을 것이다.

## Offline usage

독특한 사용법 중 하나로 공용 repo를 위에서 언급했었다. 개인적으로 플러그인을 살짝 손볼 필요가 있을 때는 공용 repo를 사용할 수 없겠지만, 인터넷이 안되는 상황이라고 할 경우 기존 방법으로 플러그인을 설치하는 것이 조금 복잡해지는데 공용 repo를 쓰면 귀찮은 것이 어느 정도 해결된다.
외부에서 사용하던 스크립트를 그대로 집어넣을 경우 github에 연결이 안되는 상황이면 플러그인을 설치할 수 없어 제대로 환경이 셋업되지 않는다. 이 때 누군가가 플러그인을 엄선해서 공용 디렉토리에 풀어놓고, 이를 참조하게 하면 인터넷 없이 플러그인을 관리할 수 있을 것이다.
가장 좋은 방법은 그런 공용 디렉토리를 소스로 사용하는 커스텀 태그를 추가하는 것이겠지만, 태그 추가와 같은 없는 기능을 추가하는 것은 아무리 프레임워크가 잘 만들어져 있다고 해도 손이 많이 가는 일이다.
그래서 내가 선택한 방법은, 미리 설치된 플러그인의 repo를 그대로 압축해서 인터넷이 안되는 환경에 옮겨논 후, zplug가 공용 디렉토리를 바라보도록 만들어서 이미 설치된 것으로 간주하게 하는 것이었다.
공용 repo를 바라보게 하는 것은 어렵지 않은데, `source $HOME/.zplug/init.zsh` 앞에 `$ZPLUG_REPO` 환경변수를 해당 공용 repo를 가리키게 하면 된다. 이렇게 하면 또 다른 장점은, 누군가가 공용 repo를 업데이트 해주기만 해도 이를 바라보는 모든 사람이 한번에 업데이트를 받을 수 있다는 점이다.

## Future work

다음에 기회가 된다면, 필요해서 간단하게 자작해봤던 zsh 스크립트에 대해 간단히 소개하는 시간이 있었으면...좋겠지만, 쓸모있는 내용이 될 때까지 정리를 한 뒤에나 할 수 있을 것 같다.

## Conclusion

zplug는 플러그인 관리 구조를 간단하고 단순하게 만들어줘서 다양한 플러그인을 어렵지 않게 관리할 수 있고, 또한 공용 repo 사용을 통해 중앙관리까지 가능하여 매우 편리한 플러그인 매니저다.
