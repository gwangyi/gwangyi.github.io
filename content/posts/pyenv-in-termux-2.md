---
title: pyenv를 termux에서 써보자 (2)
date: 2023-04-03
tags:
  - pyenv
  - termux
  - bash
  - script
  - python
series: pyenv-in-termux
isCJKLanguage: true
---

## Introduction

원래는 [이전 포스트][previous]를 작성한 뒤에 바로 이어서 2편을 쓰려고 했는데, 이직과 해외 적응, 기타등등의 여러가지 일들이 있다 보니 아주 잊어버리고 있었다. 그러다 며칠 전에 기념할만한 [첫 이슈][first-issue]가 잊고 있었던 글을 다시 기억나게 해 줘서 다시 테스트해본 뒤에 장장 4년의 기간을 뛰어넘어 예고했던 내용들을 다룰 수 있게 되었다.

[이전 포스트][previous]에서는 '일단' [pyenv][pyenv]를 설치하고 실행은 해 봤지만 아마 모두들 컴파일에 실패했을 것이다. 이 포스트에서 다룰 내용은, 실패 내용을 하나씩 짚어보며 트러블슈팅을 해볼 것이다.

### tl;dr

성격 급하신 분을 위한 명령어를 먼저 보여드리자면 다음과 같은 명령으로 빌드할 수 있다. python 버전이 바뀌면 바뀐 버전으로 고쳐줄 필요가 있고, pyenv 설치는 따로 해 줄 필요가 있다.

```bash
pkg in build-essential gdbm libandroid-posix-semaphore libandroid-support \
    libbz2 libcrypt libexpat libffi liblzma libsqlite ncurses ncurses-ui-libs \
    openssl readline zlib xorgproto patchelf
CONFIGURE_OPTS="--with-system-ffi --with-system-expat --with-ensurepip"
CONFIGURE_OPTS+=" --enable-loadable-sqlite-extensions"
CONFIGURE_OPTS+=" ac_cv_func_shm_open=yes"
CONFIGURE_OPTS+=" ac_cv_func_shm_unlink=yes"
export CONFIGURE_OPTS
wget -O - "https://api.github.com/repos/termux/termux-packages/contents/packages/python?ref=4f87b638eb93d17b89c499c3ac7d4034d5e1ac5f"    | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q | \
         sed "s%@TERMUX_PREFIX@%$PREFIX%g; s%^+++ [^/ ]*%+++ .%" | \
         CFLAGS="--target=$(gcc -dumpmachine | sed "s/-android[0-9]\\+/-android$(getprop ro.build.version.sdk)/") -I$PREFIX/include" \
         LDFLAGS="-L$PREFIX/lib" \
         LN="ln -s" \
         LIBCRYPT_LIBS="-lcrypt" \
         LD_LIBRARY_PATH="$HOME/.pyenv/versions/3.11.2/lib" \
         pyenv install -p -v 3.11.2
patchelf --set-rpath $HOME/.pyenv/versions/3.11.2/lib:$PREFIX/lib $HOME/.pyenv/versions/3.11.2/bin/python3
```

[previous]: {{< ref "pyenv-in-termux.md" >}}
[first-issue]: https://github.com/gwangyi/gwangyi.github.io/issues/1
[pyenv]: https://github.com/pyenv/pyenv

## Goal

이 문서의 목적은, [pyenv][pyenv]를 이용해 이 포스팅 당시의 최신 [CPython][cpython] 버젼인 3.11.2 버전을 최대한 [Termux][termux]에서 지원하는 바이너리 버전의 Python 패키지와 비슷하게 소스로부터 빌드하는 것이다. 문서를 잘 따라가면, 마지막에는 실행 가능한 Python 바이너리를 얻을 수 있을 것이다.

[cpython]: https://github.com/python/cpython
[termux]: https://termux.com/

### Non-goal

모든 버전의 Python을 빌드할 수 있도록 하는 것은 이 문서의 범위를 벗어난다. 물론 트러블슈팅 과정을 적절히 잘 변형시켜 적용한다면 다른 버전의 Python, 더 나아가서 CPython을 제외한 다른 구현체를 빌드해 볼 수도 있겠지만 그 부분은 여러분의 숙제로 남기도록 하겠다.

## `CONFIGURE_OPTS`

아마 제일 먼저 만날 에러는 `getrandom()` 함수를 찾을 수 없다는 에러이겠지만, 일단 그걸 다루기 전에 먼저 `CONFIGURE_OPTS`를 살펴보는 것이 좋겠다.

[pyenv][pyenv]는 Python 소스 코드를 내려받고 빌드하는 과정을 [python-build][python-build]라는 플러그인을 통해 처리하고 있다. 문서를 잘 읽어보면, [이전 포스팅][previous]에서 다뤘던 [patch][python-build-patch] 넣는 법도 찾아볼 수 있다. 우리가 지금 보려고 하는 부분은 그 윗 문단인 [Special environment variables][python-build-env] 부분이다.

[Termux의 Python 패키지][termux-python]를 보면 [`build.sh`][termux-python-build] 파일이 보인다. 여기서 `TERMUX_PKG_EXTRA_CONFIGURE_ARGS`에 들어가 있는 옵션들은 패키지 빌드 과정에서 `./configure` 명령에 패러미터로 전달되는 옵션들이다. 잘 보면 `--`로 시작하는 플래그도 보이고, `ac_cv_func_`로 시작하는 `configure` 과정에서 체크하는 함수 지원 여부를 강제로 지정하기 위한 옵션들도 보인다. 이런 명령을 [python-build][python-build]가 `configure`에게 전달하게 할 때 쓰는 환경변수가 `CONFIGURE_OPTS`이다.

일단 `ac_cv_func_` 종류는 강제하기에는 조금 찝찝하니, 일단 플래그 종류만 뽑아서 넘겨주도록 하자.

```bash
export CONFIGURE_OPTS="--with-system-ffi --with-system-expat --without-ensurepip --enable-loadable-sqlite-extensions"
```

[python-build]: https://github.com/pyenv/pyenv/blob/master/plugins/python-build/README.md
[python-build-patch]: https://github.com/pyenv/pyenv/blob/master/plugins/python-build/README.md#applying-patches-to-python-before-compiling
[python-build-env]: https://github.com/pyenv/pyenv/blob/master/plugins/python-build/README.md#special-environment-variables
[termux-python]: https://github.com/termux/termux-packages/tree/master/packages/python
[termux-python-build]: https://github.com/termux/termux-packages/blob/master/packages/python/build.sh

## Setting Target to precise Android API level

컴파일을 해 보면 이제 다음과 같은 에러를 만나게 될 것이다.

```
Python/bootstrap_hash.c:116:17: error: call to undeclared function 'getrandom'; ISO C99 and later do not support implicit function declarations [-Werror,-Wimplicit-function-declaration]
            n = getrandom(dest, n, flags);
                ^
Python/bootstrap_hash.c:120:17: error: call to undeclared function 'getrandom'; ISO C99 and later do not support implicit function declarations [-Werror,-Wimplicit-function-declaration]
            n = getrandom(dest, n, flags);
                ^
2 errors generated.
```

이런 일이 안 생기게 하려고 만드는게 `./configure` 스크립트인데 [Termux][termux] 환경에서는 짜잔, 그런데 그게 실제로 일어났습니다!

왜 이런 일이 생기냐 하면, [`getrandom()`][getrandom] 함수가 정의된 데를 보면 `#if __ANDROID_API__ >= 28`으로 둘러쌓여 있기 때문이다. 그리고 여러분의 핸드폰은 아마도 Android API 28 이상을 돌리고 있을 것이다.

[Termux][termux] Issue 중에 이 문제를 다루고 있는 이슈가 여러 개가 이미 있다. 예를 들면 [이 것][android-api-issue]이 바로 그 중 하나인데, [Termux][termux] 메인테이너들은 이 API level이 Termux 앱 자체가 지원하는 최소 API level인 24를 갖도록 하는 것을 방침으로 삼은 듯하다. 그러다보니 컴파일러의 타겟은 API level 24가 되어 24 이후 API들이 컴파일 과정에서 사라지는 것이 원인이다.

그런데 왜 `./configure`에서는 지원하는 것으로 판정이 되었는가? 하는 의문이 들 것이다. 이건 `./configure`에서 함수 지원 여부를 확인하는 방법에서 생기는 문제점인데, 함수의 존재 여부를 판단하는데 있어 해당하는 심볼의 존재 여부만 체크하기 때문이다. 우리가 만난 `getrandom()` 함수를 예를 들면, 정상적으로 쓸 때는 `#include <sys/random.h>`로 헤더파일을 불러와서 prototype을 가져오지만 `./configure`에서는 구체적인 prototype에는 관심이 없고 적절한 헤더파일이 어느 것인지를 체크하지 않기 때문에 대충 `char getrandom()`으로 정의한 다음에 컴파일 되는지만 체크하는 것이다. 그러면 우리 시스템은 일단 최신 API를 지원하기 때문에 심볼이 있어 컴파일은 됐지만 실제 코드에서는 prototype 정의가 배제되어서 컴파일 오류가 발생하는 것이다.

다행히 이 `__ANDROID_API__`는 컴파일러에 전달하는 옵션으로 지정해줄 수 있다. 일종의 cross-compile이 되는 셈이라 `--target` 플래그를 주면 된다. Android의 기본 컴파일러인 clang은 이 플래그로 전달되는 triplet에서부터 Android API level을 추론하도록 되어 있다([참고][android-clang-target-flag]). 다음 명령어를 통해서 현재 기본값으로 사용되고 있는 target이 무엇인지 확인해볼 수 있다([참고][gcc-developer-options]).

```bash
gcc -dumpmachine
# aarch64-unknown-linux-android24
```

내 핸드폰에서는 위와 같은 결과가 나온다. 여기서 첫 번째는 CPU architecture, 두 번째는 vendor, 세 번째와 네 번째는 합쳐서 OS를 의미한다. 때때로 OS는 두 부분으로 나뉘어지기도 하는데 Termux clang이 바로 그런 케이스가 되겠다. 여기서 네 번째 부분으로부터 Android API level을 가져온다.

```bash
gcc -dumpmachine | sed "s/-android[0-9]\\+/-android$(getprop ro.build.version.sdk)/"
# aarch64-unknown-linux-android33
```

이걸 이제 컴파일러에 전달하면 된다. 전통적인 `CFLAGS` 환경변수를 사용할 수 있다.

```bash
wget -O - "https://api.github.com/repos/termux/termux-packages/contents/packages/python?ref=4f87b638eb93d17b89c499c3ac7d4034d5e1ac5f"    | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q | \
         sed "s%@TERMUX_PREFIX@%$PREFIX%g; s%^+++ [^/ ]*%+++ .%" | \
         CFLAGS="--target=$(gcc -dumpmachine | sed "s/-android[0-9]\\+/-android$(getprop ro.build.version.sdk)/")" \
         pyenv install -p -v 3.11.2
```

이제 새로운 에러를 만날 수 있다.

```
ln: failed to create hard link 'libpython3.11.so' => 'libpython3.11.so.1.0': Permission denied
```

[getrandom]: https://android.googlesource.com/platform/prebuilts/ndk/+/dev/platform/sysroot/usr/include/sys/random.h#67
[android-api-issue]: https://github.com/termux/termux-packages/issues/2469
[android-clang-target-flag]: https://android.googlesource.com/platform/ndk/+/refs/heads/ndk-release-r21/docs/BuildSystemMaintainers.md#Target-Selection
[gcc-developer-options]: https://gcc.gnu.org/onlinedocs/gcc/Developer-Options.html

## Soft-link vs Hard-link

TLDR; Hard-link 대신 Soft-link를 사용하도록 `LN="ln -s"`를 주면 된다.

일반적인 Linux File System에는 두 가지 종류의 link가 있다. File system 메타데이터 차원에서 동일한 파일 내용을 가리키는 두 개의 파일 엔트리로 구현되는 hard link와 다른 파일을 가리키는 특수한 파일로서의 soft link가 그 것이다. Hard link는 원본과 링크 둘 사이에 차이점이 없어 일반적인 파일로 인식되며 둘 중 어느 하나가 지워져도 다른 하나가 동작하지만, Soft link는 원본과 링크는 구별되며, 원본이 지워지면 링크도 동작하지 않는 차이점이 있다.

Hard link는 [link system call][man-link-2]을 통해 생성하는데, Soft link는 [symlink system call][man-symlink-2]으로 생성한다. 그런데 [An droid M부터는 link syscall이 차단되었다][no-link-syscall].

`Makefile` 내용을 잘 살펴보면, 링크 생성을 위해 `LN` 환경변수를 사용하는 것을 볼 수 있다. 이것 역시 `configure` 스크립트에서 받아 넣어주는 부분이므로, `pyenv install` 명령을 줄 때 환경변수에 보태 주면 링크를 위해 `ln` 대신 soft link를 만들어주는 `ln -s` 명령을 사용하게 할 수 있다.

```bash
wget -O - "https://api.github.com/repos/termux/termux-packages/contents/packages/python?ref=4f87b638eb93d17b89c499c3ac7d4034d5e1ac5f"    | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q | \
         sed "s%@TERMUX_PREFIX@%$PREFIX%g; s%^+++ [^/ ]*%+++ .%" | \
         CFLAGS="--target=$(gcc -dumpmachine | sed "s/-android[0-9]\\+/-android$(getprop ro.build.version.sdk)/")" \
         LN="ln -s" \
         pyenv install -p -v 3.11.2
```

이러면 일단 빌드는 되는데, 반쪽짜리로 빌드가 된다. 예를 들면 `readline` 라이브러리가 정상적으로 링크되지 않아서 프롬프트에서 화살표 키를 인식하지 못한다거나, `ctypes`를 사용할 수 없다거나 하는 일이 생긴다. 왜 그런가 하면, 로그를 잘 살펴보면 다음과 같은 에러를 발견할 수 있을 것이다.

```
...

*** WARNING: renaming "_ctypes" since importing it failed: dlopen failed: cannot locate symbol "ffi_type_pointer" referenced by "/data/data/com.termux/files/usr/tmp/python-build.20230330154219.32391/Python-3.11.2/build/lib.linux-aarch64-3.11/_ctypes.cpython-311.so"...

The necessary bits to build these optional modules were not found:
_crypt                _curses               _curses_panel
_posixshmem           _tkinter              nis
readline              spwd
To find the necessary bits, look in setup.py in detect_modules() for the module's name.


Following modules built successfully but were removed because they could not be imported:
_ctypes

...
```

참고로 여기서부터는 _일단은_ 빌드가 완료되었기 때문에 파이썬 소스트리는 자동으로 삭제되어 내용을 볼 수 없다. 꼼수로 다시 `LN=` 부분을 지우면 빌드에 실패하면서 소스트리가 남고, `Makefile`을 수정해서 `LN=ln -s`로 고쳐준 다음 `make` 명령을 통해 테스트를 해볼 수 있다.

[man-link-2]: https://man7.org/linux/man-pages/man2/link.2.html
[man-symlink-2]: https://man7.org/linux/man-pages/man2/symlink.2.html
[no-link-syscall]: https://code.google.com/archive/p/android-developer-preview/issues/3150

## readline and curses

`readline`과 `curses`는 프롬프트로 쓰는 인터프리터에게 매우 중요한 라이브러리들이다. 이게 없으면 방향키로 커서를 옮길 수도 없고 히스토리를 사용할 수도 없다...

메시지에 나온 대로 `setup.py`의 `detect_modules()`를 살펴보자. 참고로 `setup.py`는 3.12부터는 [삭제될 예정][no-setup-py]이다.

```python
        # Determine if readline is already linked against curses or tinfo.
        if sysconfig.get_config_var('HAVE_LIBREADLINE'):
            if sysconfig.get_config_var('WITH_EDITLINE'):
                readline_lib = 'edit'
            else:
                readline_lib = 'readline'
            do_readline = self.compiler.find_library_file(self.lib_dirs,
                readline_lib)
            if CROSS_COMPILING:
                ret = run_command("%s -d %s | grep '(NEEDED)' > %s"
                                % (sysconfig.get_config_var('READELF'),
                                   do_readline, tmpfile))
```

위 코드는 `detect_readline_curses()`의 일부이다. `lib_dirs`가 살짝 의심스럽기 때문에 이걸 찾아가 보면 다음 부분이 나온다.

```python
        if not CROSS_COMPILING:
            self.lib_dirs = self.compiler.library_dirs + system_lib_dirs
            self.inc_dirs = self.compiler.include_dirs + system_include_dirs
```

`system_lib_dirs`, `system_include_dirs`는 크로스 컴파일이 아닌 이상 하드코딩된 `/usr/include`나 `/lib` 등을 참조하고 있기 때문에 독특한 prefix를 사용하는 Termux 에서는 잘못된 디렉토리를 참조하게 된다.

그리고 compiler의 `library_dirs`는 `add_ldflags_cppflags()`에서 확장된다.

```python
    def add_ldflags_cppflags(self):
        # Add paths specified in the environment variables LDFLAGS and
        # CPPFLAGS for header and library files.
        # We must get the values from the Makefile and not the environment
        # directly since an inconsistently reproducible issue comes up where
        # the environment variable is not set even though the value were passed
        # into configure and stored in the Makefile (issue found on OS X 10.3).
        for env_var, arg_name, dir_list in (
                ('LDFLAGS', '-R', self.compiler.runtime_library_dirs),
                ('LDFLAGS', '-L', self.compiler.library_dirs),
                ('CPPFLAGS', '-I', self.compiler.include_dirs)):
            env_val = sysconfig.get_config_var(env_var)
            if env_val:
                parser = argparse.ArgumentParser()
                parser.add_argument(arg_name, dest="dirs", action="append")

                # To prevent argparse from raising an exception about any
                # options in env_val that it mistakes for known option, we
                # strip out all double dashes and any dashes followed by a
                # character that is not for the option we are dealing with.
                #
                # Please note that order of the regex is important!  We must
                # strip out double-dashes first so that we don't end up with
                # substituting "--Long" to "-Long" and thus lead to "ong" being
                # used for a library directory.
                env_val = re.sub(r'(^|\s+)-(-|(?!%s))' % arg_name[1],
                                 ' ', env_val)
                options, _ = parser.parse_known_args(env_val.split())
                if options.dirs:
                    for directory in reversed(options.dirs):
                        add_dir_to_list(dir_list, directory)
```

[`sysconfig.get_config_var()`][sysconfig-get_config_var]는 실행중인 Python interpreter가 컴파일 될 당시의 구성을 참고할 때 쓸 수 있는 함수다. 보통은 바이너리 확장을 컴파일 할 때 Python interpreter 본체와 환경 구성을 맞추기 위해서 사용하는 것이다. 우리가 Python을 빌드할 때 prefix 정보를 이 부분으로는 주지 않았기 때문에 바이너리 확장이 하나도 제대로 빌드가 안되는 불상사가 발생한 것이다.

`setup.py`는 python 코드이기 때문에, 잘 이해가 안되는 부분에서 `import pdb; pdb.set_trace()`로 디버거를 붙여 확인해볼 수 있다.

보아하니 이 디렉토리들을 `CPPFLAGS`, `LDFLAGS`에 주어진 `-Ixxx`, `-Lxxx` 플래그로 받아오는 것으로 보인다. 이제 이 옵션까지 줘서 Python을 빌드해 보자.

```bash
wget -O - "https://api.github.com/repos/termux/termux-packages/contents/packages/python?ref=4f87b638eb93d17b89c499c3ac7d4034d5e1ac5f"    | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q | \
         sed "s%@TERMUX_PREFIX@%$PREFIX%g; s%^+++ [^/ ]*%+++ .%" | \
         CFLAGS="--target=$(gcc -dumpmachine | sed "s/-android[0-9]\\+/-android$(getprop ro.build.version.sdk)/") -I$PREFIX/include" \
         LDFLAGS="-L$PREFIX/lib" \
         LN="ln -s" \
         pyenv install -p -v 3.11.2
```

이제 빌드되지 않는 모듈은 다음과 같다.

```
The necessary bits to build these optional modules were not found:
_crypt                _posixshmem           _tkinter
nis                   spwd
To find the necessary bits, look in setup.py in detect_modules() for the module's name.
```

여기서 다시 빌드 로그의 `configure` 부분을 보면 다음 부분을 찾을 수 있다.

```
checking for stdlib extension module _multiprocessing... yes
checking for stdlib extension module _posixshmem... missing
checking for stdlib extension module fcntl... yes
checking for stdlib extension module mmap... yes
checking for stdlib extension module _socket... yes
checking for stdlib extension module grp... yes
checking for stdlib extension module ossaudiodev... yes
checking for stdlib extension module pwd... yes
checking for stdlib extension module resource... yes
checking for stdlib extension module _scproxy... n/a
checking for stdlib extension module spwd... missing
checking for stdlib extension module syslog... yes
checking for stdlib extension module termios... yes
checking for stdlib extension module pyexpat... yes
checking for stdlib extension module _elementtree... yes
checking for stdlib extension module _md5... yes
checking for stdlib extension module _sha1... yes
checking for stdlib extension module _sha256... yes
checking for stdlib extension module _sha512... yes
checking for stdlib extension module _sha3... yes
checking for stdlib extension module _blake2... yes
checking for stdlib extension module _crypt... missing
checking for stdlib extension module _decimal... yes
checking for stdlib extension module _gdbm... yes
checking for stdlib extension module nis... missing
checking for stdlib extension module _sqlite3... yes
checking for stdlib extension module _tkinter... missing
checking for stdlib extension module _uuid... yes
checking for stdlib extension module zlib... yes
checking for stdlib extension module _bz2... yes
checking for stdlib extension module _lzma... yes
checking for stdlib extension module _ssl... yes
checking for stdlib extension module _hashlib... yes
checking for stdlib extension module _testcapi... yes
checking for stdlib extension module _testclinic... yes
checking for stdlib extension module _testinternalcapi... yes
checking for stdlib extension module _testbuffer... yes
checking for stdlib extension module _testimportmultiple... yes
checking for stdlib extension module _testmultiphase... yes
checking for stdlib extension module _xxtestfuzz... yes
checking for stdlib extension module _ctypes_test... yes
checking for stdlib extension module xxlimited... yes
checking for stdlib extension module xxlimited_35... yes
```

보다시피 `missing`으로 되어 있는 모듈들이 위에 나오는 것들과 겹치는 것을 알 수 있다. `configure` 스크립트를 찾아보면 무슨 조건으로 해당 모듈이 `missing` 취급이 됐는지 확인할 수 있으므로 한번 잘 찾아보자.

[no-setup-py]: https://github.com/python/cpython/commit/81dca70d704d0834d8c30580e648a973250b2973
[sysconfig-get_config_var]: https://docs.python.org/3/library/sysconfig.html#sysconfig.get_config_var

## _posixshmem, spwd, nis

이 모듈들은 결론부터 말하자면 빌드할 수 없다. `_posixshmem`은 `shm_open()`, `shm_unlink()` 두 시스템 콜에 [의존하고 있는데][have_posix_shmem] Android는 정책적으로 이 [시스템 콜을 지원하지 않는다][no-shm]. 비슷하게 `spwd`는 `getspent()`, `getspnam()`에 [의존하고 있는데][module_spwd], Termux에서는 빌드가 안 된다. 그리고 있어도 별 의미가 없을 것 같은데, shadow passwd는 Android에서 쓸 일이 거의 없을 것이다. `nis` 모듈은 Solaris OS의 yellow page와 관련된 모듈이다. 마찬가지로 빌드가 안되는데, `yp_match()`에 의존하고 이건 Termux에서 빌드가 안된다.

`_posixshmem`의 경우는 조금 특이한데, termux package build script를 보면 강제로 `ac_cv_func_` 패러미터를 통해 `yes`로 만들고 넘어간다. 왜 이렇게 하냐면, 패치를 통해 `shm_open()`, `shm_unlink()` 함수를 구현해서 집어넣기 때문이다.

[have_posix_shmem]: https://github.com/python/cpython/blob/v3.11.2/configure#L22877
[no-shm]: https://android.googlesource.com/platform/ndk/+/4e159d95ebf23b5f72bb707b0cb1518ef96b3d03/docs/system/libc/SYSV-IPC.TXT
[module_spwd]: https://github.com/python/cpython/blob/v3.11.2/configure#L24504

## _crypt, _tkinter

두 모듈은 플래그가 덜 주어져서 빌드가 안된 모듈이다. `_crypt`는 `-lcrypt` 옵션이 빠져서 생긴 문제라서 해결이 비교적 간단하다. 사실 원래는 `pkg-config`에서 알아서 찾아주어야 하는 플래그인데 제대로 동작하지 않는 듯하다... `LIBCRYPT_LIBS`에 주어서 빌드할 수 있다.

`_tkinter`는 `X11/X.h` 헤더파일이 누락됐기 때문이다. 이건 `xorgproto` 패키지에 있으니 설치해주자.

```bash
pkg in build-essential gdbm libandroid-posix-semaphore libandroid-support \
    libbz2 libcrypt libexpat libffi liblzma libsqlite ncurses ncurses-ui-libs \
    openssl readline zlib xorgproto
CONFIGURE_OPTS="--with-system-ffi --with-system-expat --with-ensurepip"
CONFIGURE_OPTS+=" --enable-loadable-sqlite-extensions"
CONFIGURE_OPTS+=" ac_cv_func_shm_open=yes"
CONFIGURE_OPTS+=" ac_cv_func_shm_unlink=yes"
export CONFIGURE_OPTS
wget -O - "https://api.github.com/repos/termux/termux-packages/contents/packages/python?ref=4f87b638eb93d17b89c499c3ac7d4034d5e1ac5f"    | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q | \
         sed "s%@TERMUX_PREFIX@%$PREFIX%g; s%^+++ [^/ ]*%+++ .%" | \
         CFLAGS="--target=$(gcc -dumpmachine | sed "s/-android[0-9]\\+/-android$(getprop ro.build.version.sdk)/") -I$PREFIX/include" \
         LDFLAGS="-L$PREFIX/lib" \
         LN="ln -s" \
         LIBCRYPT_LIBS="-lcrypt" \
         pyenv install -p -v 3.11.2
```

짜잔, 이제 빌드가 완료되었다! `pyenv shell 3.11.2` 명령으로 활성화해서 한번 테스트 해보자.

그런데, 여기서 끝이 아니다. 만약 termux의 python 패키지를 설치했었다면 지금 설치를 실패할 것이고, 추후에 설치하게 된다면 예상치 못한 부분에서 이상한 에러가 날 가능성이 있다. 이건 libpython 라이브러리가 꼬이기 때문에 생기는 문제다.

## RUNPATH

그런데, 이미 termux에서 python을 설치해 사용중이었다면 다음과 같은 에러를 만날 수 있다.

```
  File "/data/data/com.termux/files/home/.pyenv/versions/3.11.2/lib/python3.11/sysconfig.py", line 531, in _init_posix
    _temp = __import__(name, globals(), locals(), ['build_time_vars'], 0)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ModuleNotFoundError: No module named '_sysconfigdata__linux_'
error: failed to install pip via get-pip.py
```

결론부터 말하자면, 지금 pyenv로 빌드한 libpython3 라이브러리 대신 이미 깔려 있는 termux의 libpython3이 링크되어 들어가면서 생기는 문제다. termux의 python은 `sys.implementation`에 `_multiarch` 속성이 없다. 하지만 우리가 방금 빌드한 python에는 이게 있어서 꼬이는 문제이다. 확인해보고 싶으면 직접 `$PREFIX/tmp/python-build.*/Python-3.11.2`에 들어가서 `make install`으로 `$HOME/.pyenv/versions/3.11.2/`에 설치한 다음 `$HOME/.pyenv/versions/3.11.2/bin/python3`을 실행시킨 뒤 `/proc/self/maps` 내용을 읽어보자.

```bash
cd $(ls -d $PREFIX/tmp/python-build.*/Python-3.11.2 | tail -n 1)
make install
$HOME/.pyenv/versions/3.11.2/bin/python3 -c 'print(open("/proc/self/maps").read())' | grep libpython3

# ... /data/data/com.termux/files/usr/lib/libpython3.11.so.1.0
```

아마 위와 같이 `$PREFIX/lib` 안에 있는 `libpython3.11.so` 를 가리키고 있으면 당첨이다. 이건 Termux의 특수성 때문에 일어나는 일인데, Termux는 표준 Linux 디렉토리를 가지고 있지 않기(사실은 못하기) 때문에 라이브러리 패스를 강제로 지정하고 있기 때문이다. `RUNPATH`라고 하는 ELF 포맷 항목을 `$PREFIX/lib`으로 강제 지정하는 코드가 어딘가에 들어 있는 듯 한데, 어디인지 찾지는 못했다. 확인해보려면 `objdump` 명령으로 확인해볼 수 있다.

```bash
echo "int main() { return 0; }" > test.c
gcc test.c
objdump -x a.out | grep RUNPATH

#   RUNPATH         /data/data/com.termux/files/usr/lib
```

이 RUNPATH가 하는 일은 [ld.so][ld-so-8] ELF loader가 shared object 파일을 로드할 때 찾아볼 디렉토리를 실행파일 안에 새겨놓는 역할을 한다. `LD_LIBRARY_PATH`와 비슷한데, 이건 환경변수가 아니라 실행파일 안에 들어가 있다는 것이 차이점이다. Termux의 기묘한 디렉토리 구조를 큰 문제 없이 주입하려면 _조용히_ 이걸 삽입하는게 필요한 모양이다.

이게 골치아픈 점은, 저 RUNPATH를 덮어쓰는 일반적인 방법인 `-Wl,-rpath=` 플래그를 지정해줘도 해결이 안된다는 것이다. 예를 들어 `-Wl,-rpath=$HOME/.pyenv/versions/3.11.2/lib`을 지정해주면 다음처럼 들어가게 된다.

```
   RUNPATH         /data/data/com.termux/files/usr/lib:/data/data/com.termux/files/home/.pyenv/versions/3.11.2/lib
```

대부분의 PATH들이 그렇듯이 RUNPATH도 앞에서부터 찾기 때문에 같은 버전의 python이 이미 있으면 system의 libpython을 로딩하게 되는 불상사가 생긴다.

그래서 나는 다음과 같이 해결을 보았다:

1. `LD_LIBRARY_PATH`를 주어서 `RUNPATH`를 덮어씌운다. `LD_LIBRARY_PATH`가 우선순위가 높다.
2. 빌드가 끝난 뒤 [`patchelf`][patchelf] 명령으로 `RUNPATH` 항목을 고친다.

좀 더 깔끔한 방법이 있을 것 같은데.. 지금 시점에서는 잘 모르겠다.

```bash
pkg in build-essential gdbm libandroid-posix-semaphore libandroid-support \
    libbz2 libcrypt libexpat libffi liblzma libsqlite ncurses ncurses-ui-libs \
    openssl readline zlib xorgproto patchelf
CONFIGURE_OPTS="--with-system-ffi --with-system-expat --with-ensurepip"
CONFIGURE_OPTS+=" --enable-loadable-sqlite-extensions"
CONFIGURE_OPTS+=" ac_cv_func_shm_open=yes"
CONFIGURE_OPTS+=" ac_cv_func_shm_unlink=yes"
export CONFIGURE_OPTS
wget -O - "https://api.github.com/repos/termux/termux-packages/contents/packages/python?ref=4f87b638eb93d17b89c499c3ac7d4034d5e1ac5f"    | \
         jq '.[]|select(.name|endswith(".patch"))|.download_url' | \
         xargs wget -O - -q | \
         sed "s%@TERMUX_PREFIX@%$PREFIX%g; s%^+++ [^/ ]*%+++ .%" | \
         CFLAGS="--target=$(gcc -dumpmachine | sed "s/-android[0-9]\\+/-android$(getprop ro.build.version.sdk)/") -I$PREFIX/include" \
         LDFLAGS="-L$PREFIX/lib" \
         LN="ln -s" \
         LIBCRYPT_LIBS="-lcrypt" \
         LD_LIBRARY_PATH="$HOME/.pyenv/versions/3.11.2/lib" \
         pyenv install -p -v 3.11.2
patchelf --set-rpath $HOME/.pyenv/versions/3.11.2/lib:$PREFIX/lib $HOME/.pyenv/versions/3.11.2/bin/python3
```

이제 진짜 끝! python을 termux에서 직접 빌드해서 사용할 수 있게 해 보았다.

[ld-so-8]: https://man7.org/linux/man-pages/man8/ld.so.8.html
[patchelf]: https://github.com/NixOS/patchelf

## Conclusion

Termux는 여러가지 제약사항으로 인해 일반적인 Linux와 많이 다른 형태를 가지고 있는 환경이다. 이번 프로젝트를 통해서, 대표적인 오픈소스 프로젝트인 python을 Termux 환경 하에서 빌드해보면서 디렉토리 구조, `configure` 스크립트의 동작 방식, 안드로이드의 몇 가지 제약들, 나아가서 Linux ELF loader까지 많은 요소들을 살펴볼 수 있었다.
