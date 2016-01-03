---
layout: page
title: 자동화와 Make
subtitle: 자동 변수
minutes: 0
---

> ## 학습 목표 {.objectives}
>
> * Make 자동변수를 사용해서 Makefile에 중복을 제거한다.
> * `$@`을 사용해서 현재 규칙에 있는 대상을 참조한다.
> * `$^`을 사용해서 현재 규칙에 있는 의존성을 참조한다.
> * `$<`을 사용해서 현재 규칙에 있는 첫째 의존성을 참조한다.
> * 의존성에 배쉬(bash) 와일드카드가 왜 문제를 일으키는지 설명한다.

앞선 수업 말미 연습문제를 푼 후, Makefile 은 다음과 같아 보인다:

~~~ {.make}
# Count words.
.PHONY : dats
dats : isles.dat abyss.dat last.dat

isles.dat : books/isles.txt
        python wordcount.py books/isles.txt isles.dat

abyss.dat : books/abyss.txt
        python wordcount.py books/abyss.txt abyss.dat

last.dat : books/last.txt
        python wordcount.py books/last.txt last.dat

# Generate archive file.
analysis.tar.gz : isles.dat abyss.dat last.dat
        tar -czf analysis.tar.gz isles.dat abyss.dat last.dat

.PHONY : clean
clean :
        rm -f *.dat
        rm -f analysis.tar.gz
~~~

Makefile에 중복이 엄청 많다.
예를 들어, 텍스트 파일과 데이터 파일 명칭이 Makefile 전역에 걸쳐 여러 곳에서 반복되고 있다.
Makefile은 코드의 한 형태로, 어떤 코드에서나 그렇듯이 반복되는 코드는 문제가 될 소지가 있다.
예를 들어, Makefile의 일부에서 데이터 파일 명칭을 바꾸고 나서 다른 곳에 명칭을 바꾸는 것을 잊곧 한다.
반복되는 코드를 제거해 나가자.

`analysis.tar.gz` 규칙에서, 데이터 파일과 파일 저장소 아카이브 명칭이 중복된다:

~~~ {.make}
analysis.tar.gz : isles.dat abyss.dat last.dat
        tar -czf analysis.tar.gz isles.dat abyss.dat last.dat
~~~

먼저 문서저장소 아카이브 명칭을 살펴보면, 
동작에 나온 문서저장소 명칭을 `$@`으로 교체할 수 있다:

~~~ {.make}
analysis.tar.gz : isles.dat abyss.dat last.dat
        tar -czf $@ isles.dat abyss.dat last.dat
~~~

`$@`은 Make [자동 변수](reference.html#automatic-variable)(automatic variable)로,
'현재 규칙에 대한 대상'을 의미한다.
Make가 실행되면, Make가 해당 변수를 대상 명칭으로 교체한다.

동작에 나온 의존성을 `$^`으로 교체할 수 있다:

~~~ {.make}
analysis.tar.gz : isles.dat abyss.dat last.dat
        tar -czf $@ $^
~~~

`$^`은 또다른 자동변수로, '현재 규칙에 대한 모든 의존성'을 의미한다.
다시 한번, Make가 실행될 때, Make가 해당 변수를 의존성으로 교체한다.

텍스트 파일을 갱신하고, 위에서 작성한 규칙을 재실행한다:

~~~ {.bash}
$ touch books/*.txt
$ make analysis.tar.gz
~~~

다음에 산출 결과가 나와 있다:

~~~ {.output}
python wordcount.py books/isles.txt isles.dat
python wordcount.py books/abyss.txt abyss.dat
python wordcount.py books/last.txt last.dat
tar -czf analysis.tar.gz isles.dat abyss.dat last.dat
~~~

의존성 목록에 배쉬(bash) 와일드-카드 문자를 사용할 수 있다:

~~~ {.make}
analysis.tar.gz : *.dat
        tar -czf $@ $^
~~~

텍스트 파일을 갱신하고, 위에서 작성한 규칙을 재실행한다:

~~~ {.bash}
$ touch books/*.txt
$ make analysis.tar.gz
~~~

위와 동일한 결과를 얻게 된다.

이제 데이터 파일을 삭제하고, 규칙을 재실행한다:

~~~ {.bash}
$ make clean
$ make analysis.tar.gz
~~~

산출 결과는 다음과 같다:

~~~ {.error}
make: *** No rule to make target `*.dat', needed by `analysis.tar.gz'.  Stop.
~~~

`*.dat` 패턴과 매칭되는 어떤 파일도 없어서,
`*.dat` 가 파일명 그자체로 사용되었는데,
해당 파일과 매칭되는 파일이 없어서, 어떤 규칙도 동작하지 않게 되어, 오류가 발생된다.
먼저 `.dat` 파일을 명시적으로 다시 빌드하는게 필요하다:

~~~ {.bash}
$ make dats
$ make analysis.tar.gz
~~~

> ## 의존성 갱신하기 {.challenge}
> 
> 지금 다음을 실행하면 어떤 일이 발생할까:
> 
> ~~~ {.bash}
> $ touch *.dat
> $ make analysis.tar.gz
> ~~~
> 
> 1. 아무 일도 없다.
> 2. 모든 파일이 다시 생성된다.
> 3. 단지 .dat 파일만 다시 생성된다.
> 4. 단지 analysis.tar.gz 압축파일만 다시 생성된다.

위에서 살펴봤듯이, `$^`은 '현재 규칙에 대한 모든 의존성'을 의미한다.
`analysis.tar.gz` 압축 파일에 대해서는 잘 동작하는데, 
이유는 해당 동작이 모든 의존성을 문서저장소 콘텐츠처럼 동일하게 처리하기 때문이다.

하지만, 일부 규칙에 대해, 첫째 의존성을 다르게 처리하고 싶을 수 있다.
예를 들어, `.dat` 파일에 대한 규칙은 첫째 의존성(만을) 사용해서 `wordcount.py` 에 
입력 파일로 적용코져 한다.
만약 추가적인 의존성을 추가하면(곧 그렇듯이), `wordcount.py`에 입력 파일로 전달되면 안된다. 
왜냐하면, 호출될 때 입력 파일만 호명되길 기대하기 때문이다.

Make는 이런 목적을 위한 자동변수가 있는데 `$<`으로, '현재 규칙에 대한 첫째 의존성'을 의미한다.

> ## 자동 변수를 사용하도록 `.dat` 규칙을 다시 작성하시오. {.challenge}
>
> 자동 변수  `$@`('현재 규칙에 대한 대상')와 `$<` ('현재 규칙에 대한 첫째 의존성')을 사용하도록 `.dat` 규칙을 다시 작성하시오.
