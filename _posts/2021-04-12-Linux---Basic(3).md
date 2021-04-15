---
date: 2021-04-12 02:29:39
layout: post
title: "Linux-Basic(3)"
subtitle:
description:
image:
optimized_image:
category:
tags:
- linux
author: newvelop
paginate: false
---
리눅스의 여러가지 기술에 대해 정리해본다.

### 와일드 카드
패턴 매칭에 사용되는 문자나 스트링이다.
글로브는 와일드 카드를 이용해서 여러 개의 디렉토리나 파일을 한번에 작업하는 것인데 *.txt를 예로 들자면, *은 와일드 카드이고, *.txt는 *이라는 와일드 카드를 이용해서 txt파일을 전부 선택하는 글로브라고 할 수 있다.

- \* : 0개 이상의 문자열을 의미한다
- ? : 정확히 아무글자나 한문자를 의미한다.
- [] : [] 사이에 있는 문자중 아무글자나 한글자가 매칭되면 맞는 패턴을 의미한다.
  - ex) ca\[nt\]* 일 경우, can, cat, candy, catch 등이 매칭이 된다. 
  - \[!\] : 괄호안에 있는 문자에 해당하지 않는 글자일 경우 맞는 패턴을 의미한다.
    - \[!ae\]일 경우, baseball이나 cricket은 되지만, apple은 되지 않는다.
- range : [] 사이에 '-'을 이용하여 range를 사용할 수 있다
  - \[a-e\] : a부터 e의 알파벳을 의미
- 이름 있는 문자 클래스 : 정확한 명칭을 통해 해당하는 문자 클래스를 이용할 수 있다.
  - \[\[:alpha:\]\] : 알파벳
  - \[\[:alphanum:\]\] : 알파벳이나 숫자
  - \[\[:digit:\]\] : 숫자
  - \[\[:lower:\]\] : 알파벳중 소문자
  - \[\[:upper:\]\] : 알파벳중 대문자
  - \[\[:space:\]\] : 공백
- \ : 특수문자 escape 해주는 문자
  - ex) ?일 경우 아무문자 한글자를 뜻하지만, \?일 경우 ?문자를 뜻함

### I/O 리다이렉트
기본적인 Input/output 정보는 다음과 같다
I/O 명/약어/파일 디스크립터
- Standard Input / stdin / 0
- Standard Output / stdout / 1
- Standard Error / stderr / 2

#### redirection
- \> : standard output을 file로 내보내고, file은 덮어쓰기가됨
- \>> : standard output을 file로 내보내고, file은 이어붙이기가 됨
- < : 파일을 standard input으로 내보냄
- & : 파일 디스크립터를 리다이렉션에 이용하기 위한 문자
  - 2 > &1 : stderr를 stdout에 합침
  - 2 > file : stderr을 file에 출력
- /dev/null : 아무것도 없는 곳. 출력을 버릴 때 사용
  - 2>/dev/null : stderr를 버림
  - rm test > /dev/null 2>&1 : rm test의 stdout을 /dev/null에 버리며, stderr를 stdout으로 넘기기 때문에 결국 stderr와 stdout을 버리는 것이다.

### 파일 비교
- diff {파일1} {파일2} : 파일1과 2를 비교한 결과를 표시
  - ex) 3c3 : 파일1의 3번째 줄과 2의 3번째 줄이 change 됐음을 의미
  - action : A(dd)C(hange)D(elete)가 있음
- sdiff {파일1} {파일2} : 파일1을 왼쪽으로, 파일2를 오른쪽으로 표시하여 달라진 부분 표시
- vimdiff {파일1} {파일2} : vim에서 파일들의 diff 표시
  - ctrl w w : 다음 창으로 이동

### 파일 내에서 검색 및 내용 처리
- grep : 패턴매칭을 통해 맞는 라인을 표시
  - -i : 대소문자 무시
  - -c : 파일에 몇번 나오는지 표시
  - -n : 매칭된 라인 넘버 표시
  - -v : 패턴에 안맞는 라인 찾기
- file {파일명} : 해당 파일의 파일 타입 표시
- strings : 객체나 바이너리파일의 내용물을 문자열로 표시
- | : 이전 것들을 '|'의 뒤에 있는 것으로 전달하는 기호
  - ex) cat file | grep {패턴} : 파일 내용을 토대로 grep을 실행
- cut {file} : 파일 내용 자르기
  - -d {구분자} : 해당 구분자를 통해 파일을 필드로 분할
  - -f N : N 번째 필드를 표시
- more {파일명} : 파일의 내용을 읽는 명령어. 긴파일을 나눠서 읽을때 사용한다고 함
- less {파일명} : 파일의 내용을 읽는 명령어. more과 비슷하나 이전 내용으로도 이동할 수 있는 차이점이 있다고 함

참고
- linux-administration-bootcamp 강의