---
layout: page
title: 자동화와 Make
subtitle: 데이터와 코드에 걸린 의존성
minutes: 0
---

> ## 학습 목표 {.objectives}
>
> * 출력 파일은 입력 파일에 대한 제품일 뿐만 아니라,
>   출력파일을 만들어 내는 코드 혹은 스크립트에 대한 제품이기도 하다.
> * 거짓 의존성을 인지하고 회피한다.

지금까지 작성한 Makefile은 다음과 같다:

~~~ {.make}
# Count words.
.PHONY : dats
dats : isles.dat abyss.dat last.dat

isles.dat : books/isles.txt
	python wordcount.py $< $@

abyss.dat : books/abyss.txt
	python wordcount.py $< $@

last.dat : books/last.txt
	python wordcount.py $< $@

# Generate archive file.
analysis.tar.gz : *.dat
	tar -czf $@ $^

.PHONY : clean
clean :
        rm -f *.dat
        rm -f analysis.tar.gz
~~~

데이터 파일은 텍스트 파일에 대한 제품이기도 하지만,
텍스트 파일을 처리하고 데이터 파일을 생성하는 `wordcount.py`, 스크립트에 대한 제품이기도 하다.
또한, `wordcount.py` 파일을 데이터 파일에 대한 의존성으로 추가해야만 된다:

~~~ {.make}
isles.dat : books/isles.txt wordcount.py
	python wordcount.py $< $@

abyss.dat : books/abyss.txt wordcount.py
	python wordcount.py $< $@

last.dat : books/last.txt wordcount.py
	python wordcount.py $< $@
~~~

`wordcount.py` 프로그램을 편집한 척하고, Make를 재실행하면, 

~~~ {.bash}
$ touch wordcount.py
$ make dats
~~~

다음 결과를 얻게 된다:

~~~ {.output}
python wordcount.py books/isles.txt isles.dat
python wordcount.py books/abyss.txt abyss.dat
python wordcount.py books/last.txt last.dat
~~~

`wordcount.py` 파일을 `.dat` 파일에 의존성으로 추가한 후,
`analysis.tar.gz`에 명기된 대상을 빌드하는데 관여된 의존성을 도식화했는데, 
Makefile에서 구현된 사항이 다음 그림에 나와 있다:


![wordcount.py 파일을 의존성으로 추가한 후에, analysis.tar.gz 의존성](img/04-dependencies.png "analysis.tar.gz dependencies after adding wordcount.py as a dependency")

> ## `.txt` 파일은 `wordcount.py` 파일에 의존성을 갖지 않는가? {.callout}
>
> `.txt` 파일은 입력 파일이며 어떤 의존성도 갖지 않는다.
> 입력파일이 `wordcount.py` 파일에 의존성을 만드려면,
> [거짓 의존성](reference.html#false-dependency)(false dependency)
> 도입이 필요하다.

분석 스크립트를 문서기록보관 아카이브에도 추가하자:

~~~ {.make}
analysis.tar.gz : *.dat wordcount.py
        tar -czf $@ $^
~~~

Make를 재실행하면,

~~~ {.bash}
$ make analysis.tar.gz
~~~

다음 결과를 얻게 된다:

~~~ {.output}
tar -czf analysis.tar.gz abyss.dat isles.dat last.dat wordcount.py
~~~
