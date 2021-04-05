---
date: 2021-04-02 02:29:39
layout: post
title: "Linux-Basic(2)"
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
리눅스의 기본 커맨드중 파일과 관련한 커맨드를 정리해본다.

### 파일 검색
#### find
기본적인 파일을 찾는 커맨드 이다
- find {경로} {표현식} : 해당 경로에 있는 표현식에 맞는 파일들을 찾는다.
  - -name {이름} : 이름에 맞는 파일들을 찾는다
    - -name *v : v로 끝나는 파일들을 찾음
  - -iname {이름} : 대소문자를 무시하고 이름에 맞는 파일들을 찾는다.
  - -mtime : 변경된 시간을 기반으로 파일을 찾는다. +옵션일 경우 해당 +보다 더 뒤에 수정된 파일들을 찾으며, -일 경우 해당 -보다 더 최근에 수정된 파일들을 찾는다.
  - -size : 해당 사이즈에 맞는 파일들을 찾는다. + 옵션일 경우 이상, - 옵션일 경우 이하로 적용한다.
  - -newer {파일명} : 해당 파일보다 최신의 파일들을 찾는다.

find . -exec file {} \; 를 수행할 경우 해당 디렉토리의 모든 파일 대상으로, file 커맨드를 실행하여 해당 파일의 타입을 출력한다.

#### locate
find보다 빠른 수행시간을 가진 커맨드이다. 시스템의 인덱스 기반으로 수행되기 때문에 find보다 빠른 것이며, index는 주기적으로 생성된다. 이는 실시간으로 반영되는 것이 아니기 때문에 바로 얼마전에 생성한 파일을 찾고싶을 경우는 locate이 아니라 find를 사용하는게 좋다.

![screensh](../assets/img/2021-04-02-Linux---Basic(2)/find-locate.PNG)


참고
- linux-administration-bootcamp 강의