---
layout: page
title: 자동화와 Make
subtitle: 변수
minutes: 0
---

> ## 학습 목표 {.objectives}
>
> * Makefile에 변수를 사용한다.
> * 변수에 값을 할당한다.
> * 참조 변수.
> * 구성(configuration)과 계산(computation)을 결합하지 않는(decoupling) 것의 
>    잇점을 설명한다.

지금까지의 노력에도 불구하고, Makefile에는 여전히 중복된 콘텐트, 즉 `wordcount.py`
스크립트 명칭이 존재한다. 해당 스크립트의 이름을 바꾸면, 
여러 곳에서 Makefile을 갱신해야만 된다.

스크립트 명칭을 보관하는 Make [변수](reference.html#variable) (Make 일부 버젼에서는 [매크로](reference.html#macro)라고도 불림)를 소개한다:

~~~ {.make}
COUNT_SRC=wordcount.py
~~~

변수 [할당](reference.html#assignment)(assignment)하는 예제가 위에 나와 있다 -
`COUNT_SRC` 변수에 `wordcount.py` 값이 할당되었다.

`wordcount.py`는 스크립트로, 파이썬에 전달되어 호출된다.
또다른 변수를 도입해서 스크립트 실행을 표현한다:

~~~ {.make}
COUNT_EXE=python $(COUNT_SRC)
~~~

Make가 실행될 때, `$(...)`는 Make에게 변수를 해당 값으로 치환하도록 일러준다.
이것이 변수 [참조](reference.html#reference)(reference)다.
변수값을 사용하고자 하는 어떤 곳에서도, 이런 방식으로 작성하고 참조한다.

여기서는 `COUNT_SRC` 변수를 참조한다.
Make로 하여금 `COUNT_SRC` 변수를 `wordcount.py` 값으로 치환하게 한다.
Make가 실행될 때, `COUNT_EXE`에 값으로 `python wordcount.py`를 할당한다.

이런 방식으로 `COUNT_EXE` 변수를 정의하면 스크립트 실행방식을 쉽게 변경할 수 있게 된다
(예를 들어, 스크립트 구현 언어를 파이썬에서 R로 변경하게 되면).

> ## 변수 사용하기 {.challenge}
>
> `Makefile`를 갱신해서, `%.dat` 와 `analysis.tar.gz` 규칙이
> `COUNT_SRC` 와 `COUNT_EXE` 변수를 참조하게 한다.

Makefile 상단에 변수를 위치시키게 되면, 찾고 변경하기 쉽게 한다는 의미가 된다.
대안으로, 변수 정보만 간직하는 Makefile을 새로 만들어 넣을 수 있다.
`config.mk` 파일을 생성하자:

~~~ {.make}
# Count words script.
COUNT_SRC=wordcount.py
COUNT_EXE=python $(COUNT_SRC)
~~~

다음 명령어를 사용해서, `Makefile` 내부로 Makefile을 가져올 수 있다:

~~~ {.make}
include config.mk
~~~

Make를 재실행해서, 모든 것이 여전히 잘 동작하는지 살펴보자:

~~~ {.bash}
$ make clean
$ make dats
$ make analysis.tar.gz
~~~

Makefile 구성정보와 작업을 수행하는 부분, 즉 규칙을 구분했다.
스크립트 명칭 혹은 실행방식을 변경하려면, 
`Makefile`에 있는 소스코드가 아니라, 단지 구성파일만 편집하면 된다.
이런 방식으로 코드를 구성과 결합시키지 않는 것이 좋은 프로그래밍 관례다.
더 모듈화 되고, 유연해지며, 재사용가능한 코드가 되기 때문이다.
