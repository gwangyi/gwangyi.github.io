---
title: cffi를 써 봅시다
date: 2018-12-08 01:01:00
tags:
  - python
  - c
  - ffi
  - cffi
isCJKLanguage: true
---

## Introduction

Python이 강력한 언어이긴 하지만, python만으로 모든 문제를 해결하기는 어렵다. Python의 기준 구현체인 CPython은 C/C++이나 Go 처럼 네이티브 코드를 생성하는 것이 아니라 바이트코드를 생성한 뒤 이를 하나하나 해석하는 인터프리터라 속도가 느리기도 하고, 기존에 python으로 구현되지 않은 라이브러리를 이용해야 할 경우도 종종 발생한다.

일반적인 경우, 속도가 필요한 부분은 C나 어셈블리어 등을 이용하여 작성하고, 비교적 속도에 민감하지 않은 부분은 생산성이 높은 고수준 언어를 이용해서 작성한 뒤 이를 이어붙이는 기법을 많이 사용한다. 대부분의 고수준 언어는 그 자체가 C/C++로 구현된 경우가 많아, 전체 프로젝트에 해당 언어의 처리 엔진을 심어버리는 방식으로 처리하기도 하지만, 반대로 C로 작성된 모듈을 해당 고수준 언어로 작성된 함수인 것처럼 이어붙이는 기능을 제공하는 경우도 있다.

때로는 HW를 세세하게 조작하고 싶을 때도 있을 것이다. 마침 나도 컴퓨터 부품을 만드는 회사를 다니는지라, HW를 직접 조작한다고 말하기에는 많이 부족하지만 어쨌든 커널과 이야기해야 할 일이 많다. 물론 이런 부분들도 python만 이용해서 어떻게 해볼 수도 있지만, 프로그래밍 세계에서 매우 중요한 격언인 “*바퀴의 재발명*”을 피하기 위해서라도 C로 작성된 모듈을 가져다 쓰는게 효율적인 해답이 되는 경우가 많다.

이러한 **이종 언어간 인터페이스**를 이용해 둘 이상의 언어를 이어붙이면, 하나의 언어로는 표현하기 힘든 부분까지도 표현할 수 있게 돼 프로그래밍 언어의 표현력을 한층 상승시킬수 있게 된다. 이번 포스팅에서는 그 중에서도 python과 C 언어를 이어붙일 수 있도록 도와주는 [cffi][cffi]에 대해 소개하고자 한다.

[cffi]: https://cffi.readthedocs.io/en/latest/

## Background

**이종 언어간 인터페이스**는 대부분의 고수준 언어들이 고민하는 문제인 바, [FFI][ffi]라는 개념으로 정의되어 있다. 단어나 개념만 두고 봤을 때는 서로 다른 두 언어를 이어붙이기만 한다면 다 [FFI][ffi]라 부를 수 있겠지만, 보편적으로는 어떤 언어와 C 언어 사이의 인터페이스로 정의되곤 한다. C 자체가 어셈블리에 한없이 가까운 -- 누군가의 표현에 따르면 *함수 기능이 잘 지원되는 어셈블리어* -- 언어인지라 C에서 쓰이기 위해 준비되어야 하는 요소들이 다른 언어들에 비해 매우 적어서인 듯하다. 마치 현실에서의 영어와도 비슷한데, 전 세계 사람들이 영어를 배우기 때문에 독일인과 한국인이 만나 이야기할 때도 영어로 대화하는 것과 비슷하다고 할 수 있을 것이다.

물론 C를 중간 언어로 두지 않고 직접 이어보려는 시도도 있는데, [SWIG][swig] 같은 것을 들 수 있을 것이다. 완전히 C를 배제하는 것은 아니지만, 별도의 간결한 언어로 인터페이스를 작성하여 연결하는 부분을 자동화 함으로써 직접 C를 건드리는 것을 피할 수 있도록 해 주었다. 현실에서의 예를 들자면 에스페란토어와 같은 인공어를 들 수 있을 것인데, 다른 언어끼리의 소통을 위해 중립적인 언어를 새로 디자인했다는 점에서 비슷하다.

Python에서는 [ctypes][ctypes] 모듈이 표준 라이브러리에 포함되어 [FFI][ffi]를 사용할 수 있도록 되어 있다. [ctypes][ctypes]는 상당히 훌륭한 [FFI][ffi] 모듈이기는 하나, 성능 오버헤드가 조금 있고 사용하기가 다소 불편한 감이 있다.
[ctypes][ctypes]를 이용하기 위해서는,

* 먼저 C로 작성된 코드가 shared object나 DLL의 형태로 컴파일되어 있어야 하고,
* 함수와 구조체 등을 [ctypes][ctypes]에 맞는 방식으로 다시 정리해야되며,
* OS에서 제공하는 symbol lookup 방식으로 찾을 수 있는 요소에 한해서만 접근이 가능하다.

첫 번째는 빌드 스크립트 등으로 어떻게든 되는 문제이긴 하지만, 두 번째는 작은 규모의 프로젝트라면 몰라도 많은 수의 함수를 가져와야 되면 손이 매우 많이 가는 일이 되며, 세 번째는 상수 매크로나 간단한 매크로 함수, inline 함수 등은 아예 사용이 불가능하다는 것을 의미하게 된다.

이 세 가지의 문제를 한번에 해결시켜 줄 수 있는 것이 [cffi][cffi]이다.

[ffi]: https://en.wikipedia.org/wiki/Foreign_function_interface
[swig]: http://www.swig.org
[ctypes]: https://docs.python.org/3/library/ctypes.html

## Calling C from Python

[cffi][cffi]의 사용법은 매우 직관적이다. 물론 C를 배운적이 없고 python만 사용해 본 사람이라면 [ctypes][ctypes]가 더 직관적일 수도 있다. [cffi][cffi]가 직관적이라는 부분은, C의 문법으로 C 확장을 정의할 수 있다는 점에서 그렇다. 예를 들어 `puts()` 함수를 [cffi][cffi]로 가져다 쓰려면 다음과 같이 작성하면 된다.

모듈 정의:
```python
from cffi import FFI

ffi = FFI()
ffi.set_source('mylib', r'''
#include <stdio.h>
''')
ffi.cdef(r'''
void puts(const char *);
''')
ffi.compile(verbose=True)
```

사용:
```python
import mylib

mylib.lib.puts(b'Hello, world!')
```

[cffi][cffi]도 [ctypes][ctypes]처럼 이미 빌드된 shared object로부터 runtime binding을 통해 함수 등을 가져올 수도 있지만, 그렇게 쓴다면 굳이 [cffi][cffi]를 쓸 필요성이 없다고 개인적으로 생각하기 때문에 그 방법에 대해서는 따로 기술하지 않도록 하겠다. 궁금하다면 [해당 문서 자료](https://cffi.readthedocs.io/en/latest/overview.html#simple-example-abi-level-in-line)를 참조해서 보라.

`set_source()` 함수의 첫 번째 인자는 `import`를 통해 가져올 모듈의 주소가 된다. 패키지 안으로 넣고 싶다면 `'some.package.module'`과 같이 `.`으로 구분하면 이에 맞추어 디렉토리 구조를 만들면서 모듈을 생성하게 된다. 두 번째 인자는 모듈 안에 넣고자 하는 C 코드를 넣으면 된다. 여기서 정의된 함수나 전역변수 등을 나중에 꺼내 쓸 수 있다. 여기서는 생략되어 있지만 `libraries` 패러미터로 추가 라이브러리(`-lsomething` 등으로 지정하는 라이브러리)를 지정하거나, `library_dir`, `include_dir` 옵션을 통해 library directory나 include directory를 지정해 넣을 수도 있다.

`cdef()` 함수에서는 python에서 가져다 쓰고 싶은 함수, 전역변수 등을 C 언어 문법 그대로 작성해 넣는다. `typedef`나 `struct`, `union` 등도 여기다 넣으면 나중에 사용할 수 있다.

`cdef()`의 재미있는 점은, 여기서는 C 확장 모듈의 인터페이스를 정의하는 것이고, 실제 함수나 전역변수 등은 `set_source()`에서 처리해야한다는 점이다. 다르게 말하면, 실제로는 함수나 전역변수가 아니더라도, 함수나 전역변수처럼 사용할 수 있다면 `cdef()`에서 함수나 전역변수인 것처럼 정의해줌으로써 python에서 함수나 전역변수의 형태로 사용할 수 있다는 것이다. 이렇게 길게 설명하니 말이 어려운데, 예를 들자면 매크로 상수처럼 실제로 변수가 아닌 것도 마치 전역변수인 것마냥 외부로 노출시켜 python에서 가져다 쓸 수 있도록 하는 것이 가능하다는 것이다.

`cdef()`의 다른 특징 하나는 여러번 호출이 가능하다는 것이다. 여러번 호출하면 이전 호출이 사라지는 것은 아니고 정의들이 누적이 된다.

이렇게 정의된 모듈은 `compile()` 함수를 통해서 실제로 python에서 가져다 쓸 수 있는 C extension 모듈로 컴파일된다. 컴파일 된 후에는 해당 모듈은 `ffi` 객체와 `lib` 객체를 노출하는데, `lib` 객체를 통해 `cdef()`에서 노출시킨 함수와 전역변수등을 접근하는 것이 가능하다. `ffi` 객체는 `memmove` 등의 C 메모리를 접근하는 유틸리티 함수와 자료형에 맞는 메모리를 할당받는 `new` 등의 함수를 가지고 있다. 참고로 할당받은 메모리를 해제하는 함수는 따로 없는데, GC에서 해당 객체를 수집할 때 삭제하도록 되어 있기 때문이다.

### Macro constant / Macro function

앞서 `cdef()`에서 매크로도 노출시킬 수 있다고 했었다. 전역 변수를 노출시키게 되면 `setter`와 `getter`를 통해서 노출된 전역 변수를 읽어오거나 바꿀 수 있다. 하지만 Macro constant는 상수로 런타임에 그 값이 바뀔 수 없기 때문에, `const` 한정자를 붙여 정의해줌으로써 가져다 쓸 수 있게 된다.

```python
ffi.set_source('my_lib', r'''
#define VALUE 0
#define STR "Hello!"
```)
ffi.cdef('const int VALUE; const char * STR;')
```

매크로 함수도 일반적인 함수처럼 사용이 가능하다면 마찬가지로 `cdef()`를 통해 외부에 노출시켜줄 수 있다.

```python
ffi.set_source('my_lib', r'''
#define MIN(a, b) ((a) < (b) ? (a) : (b))
```)
ffi.cdef('const int MIN(const int a, const int b)')
```

### Memory allocation

C 자료형을 다루고자 할 때에는 `ffi.new()` 함수를 사용하면 된다. 특이한 점은, 생성할 자료형을 지정할 때 C 문법을 사용한다는 것이다.

```python
ffi.cdef(r'''
struct happy {
    int score;
    int reason;
};
''')
...
happy = ffi.new('struct happy *')
happy.score = 1
happy.reason = 2

buff = ffi.new('char [1024]')
buff[0] = 0x41
buff[1] = 0x42
buff[2] = 0
print(ffi.string(buff))
# ==> b'AB'
```

## Calling Python from C

물론 C로 작성된 코드에서 python 함수를 호출하고 싶을 때도 있을 것이다. python 함수를 C 함수로 싸 주는 래퍼를 실시간으로 생성할 수 있다면 제일 좋겠지만, [cffi][cffi]는 그 방법을 deprecate 시켜버리고 대신 `extern "Python"`이라는 방법을 이용하여 감싸고 있다.

미리 특정 이름을 가진 함수를 만들어 준비시켜 놓고, python에서 그 이름의 함수를 구현하여 이를 [cffi][cffi]를 통해 연결시켜주면, C 함수가 마치 지정한 Python 함수인 것처럼 동작하도록 지정할 수 있다.

```python
ffi.set_source('my_lib', r'''
extern int g(int a);
int f(int a) {
    return g(a) + 5;
}
''')
ffi.cdef(r'''
extern "Python" int g(int a);
int f(int a);
''')

@ffi.def_extern
def g(a):
    return a * 2
```

좀 더 자세한 자료는 [Extern "Python" (new-style-callbacks)](https://cffi.readthedocs.io/en/latest/using.html#extern-python-new-style-callbacks) 문서를 참고하라.

### Embedding

단순한 callback으로만 사용한다면 C에서 Python 코드를 호출한다는 것이 그다지 큰 의미를 갖지 못할 수도 있다. 아예 python 해석기와 python 코드를 shared object로 감싸버려서, 다른 프로그램이 C 함수를 호출하듯이 python 함수를 호출할 수 있게 노출시킬 수도 있다.

이 embedding은 [공식 문서](https://cffi.readthedocs.io/en/latest/overview.html#embedding)에서 간결하고 단순하게 잘 표현해 놓았기 때문에 공식 문서를 한번만 읽어도 감을 잡을 수 있을 것이라고 기대한다.

Embedding은 매우 강력한 기능인데, python 코드를 SystemVerilog의 DPI 인터페이스를 통해 SystemVerilog 시뮬레이터에 물려버릴 수도 있다.

## Cooperation with setuptools

`compile()` 함수를 통해 모듈을 컴파일 할 수도 있지만, [cffi][cffi]의 진가는 [setuptools][setuptools]와의 협업에도 있다. ffi object를 만들고 설정해주는 `.py` 파일을 만든 뒤 `compile()` 호출을 빼 놓자. 그 뒤에 `setup.py`에 `setup_requires` 항목으로 `cffi>=1.0.0`을 추가해주고 `cffi_modules` 항목으로 `ffi_builder.py:ffi_object` 와 같이 만든 `.py` 파일과 ffi object 이름을 넣어주면 `python setup.py` 명령으로 패키징하거나 설치할 때 자동으로 `cffi` 모듈을 컴파일해 줄 것이다.

```python
from setuptools import setup

setup(
    ...
    setup_requires=["cffi>=1.0.0"],
    cffi_modules=["piapprox_build:ffibuilder"], # "filename:global"
    install_requires=["cffi>=1.0.0"],
)
```

`install_requires`가 들어간 이유는, `setup_requires`는 패키지를 빌드할 때만 사용하고 설치가 끝나면 지워지기 때문에, 설치한 후로도 계속해서 의존할 라이브러리는 `install_requires`에 따로 추가해야 하기 때문이다.

[setuptools]: https://setuptools.readthedocs.io/en/latest/

## Future work

Python으로 구현된 C preprocessor인 [pcpp][pcpp]를 소개하고자 한다. [cffi][cffi]와 조합하면 `cdef()`에 전달할 함수 정의 목록을 좀 더 쉽게 얻을 수 있다.

[pcpp]: https://ned14.github.io/pcpp/

## Conclusion

[cffi][cffi]는 C 함수나 변수를 python C 확장의 형태로 감싸주거나, python 함수를 C 함수로 감싸주는 작업을 자동화해주는 편리한 [FFI][ffi] 라이브러리이다. 게다가, C 언어의 문법을 그대로 사용하여 인터페이스를 정의할 수 있어 어렵지 않게 사용법을 익힐 수 있다. 이를 이용하면 python으로 작성 가능한 프로그램의 표현 범위를 크게 늘릴 수 있다.
