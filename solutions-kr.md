---
layout: page
title: 자동화와 Make
subtitle: 해답
minutes: 0
---

## 수업 02-makefiles

> ## 신규 규칙 두개 작성하기 {.challenge}
>
> `last.dat` 파일에 대한 새로운 규칙을 작성하는데, `books/last.txt` 파일에서 생성된다.
>
> `dats` 규칙을 상기 대상을 포함하도록 갱신하라.
>
> `analysis.tar.gz` 에 대한 새로운 규칙을 작성하는데,
> 데이터 파일을 보관하도록 압축파일이 생성된다.
> 규칙은 다음 작업이 필요하다:
> 
> * 세가지 `.dat` 파일 각각에 의존성을 갖는다.
> * `tar -czf analysis.tar.gz isles.dat abyss.dat last.dat`  동작을 호출한다.
> 
> `analysis.tar.gz` 파일을 제거하도록 `clean`을 갱신한다.

~~~ {.bash}
# Count words.
.PHONY : dats
dats : isles.dat abyss.dat last.dat

isles.dat : books/isles.txt
    	python wordcount.py books/isles.txt isles.dat

abyss.dat : books/abyss.txt
    	python wordcount.py books/abyss.txt abyss.dat

last.dat: books/last.txt
    	python wordcount.py books/last.txt last.dat

analysis.tar.gz: isles.dat abyss.dat last.dat
    	tar -czf analysis.tar.gz isles.dat abyss.dat last.dat

.PHONY : clean
clean :
    	rm -f *.dat
    	rm -f analysis.tar.gz
~~~

## Lesson 03-variables
> ## Update dependencies {.challenge}
> 
> What will happen if you now execute:
> 
> ~~~ {.bash}
> $ touch *.dat
> $ make analysis.tar.gz
> ~~~
> 
> 1. nothing
> 2. all files recreated
> 3. only .dat files recreated
> 4. only analysis.tar.gz recreated

* * *
4.) only `analysis.tar.gz` recreated. 

You can check that `*.dat` is being expanded in the target of the rule for
    `analysis.tar.gz` by echoing the value of the automatic variable `$^` 
    (all dependencies of the current rule).

~~~ {.bash}
analysis.tar.gz: *.dat
    @echo $^
    tar -czf $@ $^
~~~
The rules for `*.dat` are not executed because their corresponding `.txt` files
haven't been modified.

If you run:

~~~ {.bash}
    $ touch *.dat
    $ touch books/*.txt
    $ make analysis.tar.gz
~~~

You will find that the `.dat` files as well as `analysis.tar.gz` are recreated.

* * *



> ## Rewrite `.dat` rules to use automatic variables {.challenge}
>
> Rewrite each `.dat` rule to use the automatic variables `$@` ('the
> target of the current rule') and `$<` ('the first dependency of the
> current rule').

~~~ {.bash}
# Count words.
.PHONY : dats
dats : isles.dat abyss.dat last.dat

isles.dat : books/isles.txt
    	python wordcount.py $< $@

abyss.dat : books/abyss.txt
    	python wordcount.py $< $@

last.dat: books/last.txt
    	python wordcount.py $< $@

analysis.tar.gz: *.dat
    	tar -czf $@ $^

.PHONY : clean
clean :
    	rm -f *.dat
    	rm -f analysis.tar.gz
~~~


## Lesson 06-variables

> ## Use variables {.challenge}
>
> Update `Makefile` so that the `%.dat` and `analysis.tar.gz` rules
> reference the variables `COUNT_SRC` and `COUNT_EXE`.

~~~ {.bash}
# Count words.
COUNT_SRC=wordcount.py
COUNT_EXE=python $(COUNT_SRC)

.PHONY : dats
dats : isles.dat abyss.dat last.dat

%.dat : books/%.txt COUNT_SRC
    $(COUNT_EXE) $< $*.dat

# Generate archive file.
analysis.tar.gz : *.dat COUNT_SRC
    tar -czf $@ $^

.PHONY : clean
clean :
        rm -f *.dat
        rm -f analysis.tar.gz
~~~

## Lesson 08-variables

> ## Extend the Makefile to create JPEGs {.challenge}
>
> Add new rules, update existing rules, and add new macros to:
> 
> * Create `.jpg` files from `.dat` files using `plotcount.py`.
> * Add the script and `.jpg` files to the archive.
> * Remove all auto-generated files (`.dat`, `.jpg`,
>   `analysis.tar.gz`). 

~~~{.bash}
 # config.mk
 # Count words script.
COUNT_SRC=wordcount.py
COUNT_EXE=python $(COUNT_SRC)

# Plot word counts script.
PLOT_SRC=plotcount.py
PLOT_EXE=python $(PLOT_SRC)
~~~

~~~{.bash}
# Makefile
include config.mk

TXT_FILES=$(wildcard books/*.txt)
DAT_FILES=$(patsubst books/%.txt, %.dat, $(TXT_FILES))
JPG_FILES=$(patsubst books/%.txt, %.jpg, $(TXT_FILES))

# Count words.
%.dat : books/%.txt $(COUNT_SRC)
    $(COUNT_EXE) $< $*.dat

# Plot word counts.
%.jpg : %.dat $(PLOT_SRC)
    $(PLOT_EXE) $*.dat $*.jpg

.PHONY : clean
clean :
    rm -f $(DAT_FILES)
    rm -f $(JPG_FILES)
    rm -f analysis.tar.gz

.PHONY : dats
dats : $(DAT_FILES)

.PHONY : jpgs
jpgs : $(JPG_FILES)

analysis.tar.gz : $(DAT_FILES) $(JPG_FILES) $(COUNT_SRC) $(PLOT_SRC)
    tar -czf $@ $^

.PHONY : variables
variables:
    @echo TXT_FILES: $(TXT_FILES)
    @echo DAT_FILES: $(DAT_FILES)
    @echo JPG_FILES: $(JPG_FILES)
~~~

