---
layout: page
title: 자동화와 Make
subtitle: 함수
minutes: 0
---

> ## 학습 목표 {.objectives}
>
> * Make `wildcard` 함수를 사용해서 패턴과 매칭되는 파일 목록을 얻어온다.
> * Make `patsubst` 함수를 사용해서 파일명을 다시 작성한다.

현재 시점엣, Makefile은 다음과 같다:

~~~ {.make}
include config.mk

# Count words.
.PHONY : dats
dats : isles.dat abyss.dat last.dat

%.dat : books/%.txt $(COUNT_SRC)
	$(COUNT_EXE) $< $*.dat

# Generate archive file.
analysis.tar.gz : *.dat $(COUNT_SRC)
	tar -czf $@ $^

.PHONY : clean
clean :
        rm -f *.dat
        rm -f analysis.tar.gz
~~~

Make에는 많은 [함수](reference.html#function)(functions)가 지원되어 더 복잡한 규칙을 작성할 수 있다. 한 사례가 `wildcard`다. `wildcard`는 특정 패턴과 매칭되는 파일 목록을 얻어와서,
변수에 저장할 수 있다. 예를 들어, 모든 텍스트 파일(`.txt`로 끝나는 파일) 목록을 불러와서
Makefile 시작부분에 변수에 불러온 값을 저장한다:

~~~ {.make}
TXT_FILES=$(wildcard books/*.txt)
~~~

변수의 값을 보여주도록 `.PHONY` 대상과 규칙으로 추가한다:

~~~ {.make}
.PHONY : variables
variables:
	@echo TXT_FILES: $(TXT_FILES)
~~~

> ## @echo {.callout}
>
> Make는 동작을 실행할 때 동작을 화면에 출력한다.
> 동작 시작부에 `@`을 사용하면 Make로 하여금 동작을 출력하지 않게 한다.
> 그래서, `echo` 대신에 `@echo`을 사용함으로써, 
> `echo` 실행 결과(변수값을 출력)를 볼 수 있지만, `echo` 명령어 자체는 볼 수 없게 된다.

Make를 실행하면:

~~~ {.bash}
$ make variables
~~~

다음 결과를 얻게 된다:

~~~ {.output}
TXT_FILES: books/abyss.txt books/isles.txt books/last.txt books/sierra.txt
~~~

이제 `sierra.txt`도 포함된 것에 주목한다.

함수를 도입한 후,  `analysis.tar.gz`에 명기된 대상을 빌드하는데 관여된 의존성을 도식화했는데, 
Makefile에서 구현된 사항이 다음 그림에 나와 있다:

![함수를 도입한 후에 analysis.tar.gz 의존성](img/07-functions.png "analysis.tar.gz dependencies after introducing a function")

`patsubst` ('pattern substitution', 패턴 치환)은 

`patsubst` ('pattern substitution') takes a pattern, a replacement string and a
list of names in that order; each name in the list that matches the pattern is 
replaced by the replacement string. Again, we can save the result in a
variable. So, for example, we can rewrite our list of text files into
a list of data files (files ending in `.dat`) and save these in a
variable:

~~~ {.make}
DAT_FILES=$(patsubst books/%.txt, %.dat, $(TXT_FILES))
~~~

We can extend `variables` to show the value of `DAT_FILES` too:

~~~ {.make}
.PHONY : variables
variables:
	@echo TXT_FILES: $(TXT_FILES)
	@echo DAT_FILES: $(DAT_FILES)
~~~

If we run Make,

~~~ {.bash}
$ make variables
~~~

then we get:

~~~ {.output}
TXT_FILES: books/abyss.txt books/isles.txt books/last.txt books/sierra.txt
DAT_FILES: abyss.dat isles.dat last.dat sierra.dat
~~~

Now, `sierra.txt` is processed too.

With these we can rewrite `clean` and `dats`:

~~~ {.make}
.PHONY : clean
clean :
        rm -f $(DAT_FILES)
        rm -f analysis.tar.gz

.PHONY : dats
dats : $(DAT_FILES)
~~~

Let's check:

~~~ {.bash}
$ make clean
$ make dats
~~~

We get:

~~~ {.output}
python wordcount.py books/abyss.txt abyss.dat
python wordcount.py books/isles.txt isles.dat
python wordcount.py books/last.txt last.dat
python wordcount.py books/sierra.txt sierra.dat
~~~

We can also rewrite `analysis.tar.gz` too:

~~~ {.make}
analysis.tar.gz : $(DAT_FILES) $(COUNT_SRC)
	tar -czf $@ $^
~~~

If we re-run Make:

~~~ {.bash}
$ make clean
$ make analysis.tar.gz
~~~

We get:

~~~ {.output}
$ make analysis.tar.gz
python wordcount.py books/abyss.txt abyss.dat
python wordcount.py books/isles.txt isles.dat
python wordcount.py books/last.txt last.dat
python wordcount.py books/sierra.txt sierra.dat
tar -czf analysis.tar.gz abyss.dat isles.dat last.dat sierra.dat wordcount.py
~~~

We see that the problem we had when using the bash wild-card, `*.dat`,
which required us to run `make dats` before `make analysis.tar.gz` has
now disappeared, since our functions allow us to create `.dat` file
names from those `.txt` file names in `books/`.

Here is our final Makefile:

~~~ {.make}
include config.mk

TXT_FILES=$(wildcard books/*.txt)
DAT_FILES=$(patsubst books/%.txt, %.dat, $(TXT_FILES))

.PHONY: variables
variables:
	@echo TXT_FILES: $(TXT_FILES)
	@echo DAT_FILES: $(DAT_FILES)

# Count words.
.PHONY : dats
dats : $(DAT_FILES)

%.dat : books/%.txt $(COUNT_SRC)
	$(COUNT_EXE) $< $*.dat

# Generate archive file.
analysis.tar.gz : $(DAT_FILES) $(COUNT_SRC)
	tar -czf $@ $^

.PHONY : clean
clean :
	rm -f $(DAT_FILES)
	rm -f analysis.tar.gz
~~~

Remember, the `config.mk` file contains:

~~~ {.make}
# Count words script.
COUNT_SRC=wordcount.py
COUNT_EXE=python $(COUNT_SRC)
~~~
