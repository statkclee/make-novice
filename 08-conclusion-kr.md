---
layout: page
title: 자동화와 Make
subtitle: 결론
minutes: 0
---

> ## 학습 목표 {.objectives}
>
> * Make 같은 자동 빌드 도구의 장점.

Make 같은 자동 빌드 도구는 여러가지 방식으로 도움을 줄 수 있다.
반복되는 명령어를 자동화해서, 명령어를 수작업으로 실행할 때,
범하게 되는 오류위험을 줄여 주고, 시간을 절약해서 도움이 된다.

생성할 파일이 어떤 방식으로 변경될 때,
자동적으로 생성되는 산출물(데이터 파일 혹은 그래프)만 다시 생성하게 함으로써
시간을 절약해준다.

대상, 의존성, 동작 개념을 통해,
코드, 스크립트, 도구, 구성(configuration), 원데이터, 
파생된 데이터, 그래프, 논문 사이 의존성을 기록하는 문서화 형태로 역할을 수행한다.

> ## Makefile을 연장해서 JPEG 생성하기 {.challenge}
>
> 새로운 규칙을 추가하고, 기존 규칙을 갱신하고, 매크로를 새로 추가하여 다음 작업을 수행한다:
> 
> * `plotcount.py` 프로그램을 사용해서 `.dat` 파일에서 `.jpg` 파일을 생성한다.
> * 스크립트와 `.jpg` 파일을 문서보관 아카이브에 추가한다.
> * 자동생성된 모든 파일(`.dat`, `.jpg`, `analysis.tar.gz`)을 제거한다. 

이미지에 대한 기능을 추가한 후,
`analysis.tar.gz`에 명기된 대상을 빌드하는데 관여된 의존성을 도식화했는데, 
Makefile에서 구현된 사항이 다음 그림에 나와 있다:

![이미지가 추가된 후 analysis.tar.gz 의존성](img/08-conclusion-challenge.png "analysis.tar.gz dependencies once images have been added")
