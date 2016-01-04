---
layout: page
title: 자동화와 Make
subtitle: 패턴 규칙
minutes: 0
---

> ## 학습 목표 {.objectives}
>
> * Make 패턴 규칙을 작성한다.
> * Make 와일드-카드 `%`를 대상과 의존성에 사용한다.
> * Make 특수 변수 `$*`을 동작에 사용한다.
> * 규칙에는 Make 와일드-카드 사용을 회피한다.

Makefile에는 여전히 반복되는 콘텐츠가 있다.
텍스트 파일과 데이터 파일 명칭을 제외하고 각 `.dat` 파일에 대한 규칙은 동일한다.
이러한 규칙을 단일 [패턴 규칙](reference.html#pattern-rule)(pattern rule)으로 
교체할 수 있는데, 패턴 규칙을 사용해서 `books/` 디렉토리에 `.txt` 파일을 임의 `.dat` 파일로 
빌드한다:

~~~ {.make}
%.dat : books/%.txt wordcount.py
        python wordcount.py $< $*.dat
~~~

`%`는 Make [와일드-카드](reference.html#wild-card)(wild-card)다.
`$*`은 특수 변수로, 특수변수가
[스템](reference.html#stem)(stem)을 규칙이 매칭하는 것으로 치환한다.

만약 Make를 다시 실행하면,

~~~ {.bash}
$ make clean
$ make dats
~~~

다음 결과를 얻게 된다:

~~~ {.output}
python wordcount.py books/isles.txt isles.dat
python wordcount.py books/abyss.txt abyss.dat
python wordcount.py books/last.txt last.dat
~~~

> ## Make 와일드-카드 사용하기 {.callout}
>
> Make `%` 와일드-카드는 대상과 해당 의존성에만 사용될 수 있다.
> 동작에는 사용될 수가 없다.
> 하지만, 동작에서 `$*`을 사용할 수 있는데, 스템이 규칙이 매칭하는 것으로 치환한다.

Makefile은 이제 훨씬 더 짧아졌고, 깨끗해졌다:

~~~ {.make}
# Count words.
.PHONY : dats
dats : isles.dat abyss.dat last.dat

%.dat : books/%.txt wordcount.py
	python wordcount.py $< $*.dat

# Generate archive file.
analysis.tar.gz : *.dat wordcount.py
	tar -czf $@ $^

.PHONY : clean
clean :
        rm -f *.dat
        rm -f analysis.tar.gz
~~~
