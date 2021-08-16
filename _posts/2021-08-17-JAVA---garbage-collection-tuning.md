---
date: 2021-08-17 07:00:39
layout: post
title: "JAVA-garbage collection tuning"
subtitle:
description:
image:
optimized_image:
category:
tags:
- JAVA
- GC
- Garbage collection
author: newvelop
paginate: false
---
JAVA에서 garbage collection 튜닝에 대해서 알아본다

### 모니터링
일단 튜닝을 하기 위해선 어떤 문제가 있는지 알아볼 필요가 있다. 따라서 현재 application의 GC 현황을 모니터링 하는 방법을 살펴보면, Oracle JVM 기준으로 jstat을 이용하면 현재 gc 현황을 할 수 있따. jps를 통해서 vmid와 main 메소드 정보를 얻어온뒤, jstat의 인자로 vmid를 넘겨주면 gc 정보를 출력할 수 있다. 이 방법은 커맨드라인을 통해 진행하는 방법이며, VisualVM과 Visual GC플러그인 조합으로 gc 현황을 시각화할 수 있다. 이 방법으로 모니터링 준비는 완료이다.

### 튜닝
모니터링 결과 0.3초 이내로 gc가 수행된다면 튜닝할 필요는 없지만 초단위로 걸린다면 튜닝을 진행해야한다.

참고한 링크에선 이런 식의 지표가 나올 경우, gc 튜닝을 할 필요가 없다고 판정하고 있다.

- Minor GC의처리시간이빠르다(50ms내외).
- Minor GC 주기가빈번하지않다(10초내외).
- Full GC의처리시간이빠르다(보통1초이내).
- Full GC 주기가빈번하지않다(10분에 1회).

이런 상황이 아닐 경우 문제가 있다고 판단하여 gc 튜닝을 진행한다.

#### 알맞는 GC 설정
일단 JAVA 8미만일 경우는 Parallel GC, Parallel Compacting GC, CMS GC 3중에 하나를 선택한다. 3가지 다 돌려보고 그중에 좀더 빨라지는 gc 방식을 선택하면된다. 일반적으로는 CMS가 가장 빠르나, 항상 빠르지는 않다고 한다. Parallel GC는 Full GC 이후 compaction을 진행하기 때문에 한번 진행할때는 느리나, 이후 할당에 빠르다. 반면 CMS는 compaction이 맨처음에는 없기 때문에 더 빠르나, Old의 영역이 많이 남아있는데도 불구하고, 더 이상 배치할 공간이 없을만큼 파편화가 일어났을 경우, Concurrent mode failure을 일으키면서 Compaction이 일어나기 때문에 시간이 더길어진다. 따라서 직접 운영해보고 맞는 방법으로 고치는게 맞다. JAVA 8부터는 G1 GC가 기본 설정이기 때문에 이 GC 까지 포함해서 실행을 해보는 편이 좋다.

#### 메모리 크기 지정
메모리 크기가 크면 GC 발생횟수는 줄어들고 GC 수행시간은 길어지는 반면, 메모리크기가 작으면 GC 수행시간은 줄어들고
GC 발생횟수는 증가한다. 이 메모리가 클때 FULL GC가 일어나면 STOP THE WORLD 시간이 매우 길어지기 때문에 메모리를 적당한 사이즈로 지정할 필요가 있다. FULL GC가 일어났을 때, 기존 어플리케이션이 무조건 사용해야하는 메모리를 측정해서, 이 메모리 + OLD 영역용 + 여유메모리로 지정해서 메모리 할당을 하고 결과를 지켜보면서 맞는 메모리 크기를 설정한다. 또한 YOUNG 영역과 OLD 영역의 비율을 잘 조절해서 실행해야한다. OLD 영역이 크면 FULL GC의 시간이 길어지기 때문에 비율을 잘 조절해서 사용해야한다.

GC튜닝해서 개선되는 지표의 우선순위를 참고사이트에선 다음과 같이 선정하고 있다.
- Full GC 수행시간
- Minor GC 수행시간
- Full GC 수행간격
- Minor GC 수행간격
- 전체 Full GC 수행시간
- 전체 Minor GC 수행시간
- 전체 GC 수행시간
- Full GC 수행횟수
- Minor GC 수행횟수

참고
- https://d2.naver.com/helloworld/37111
- https://d2.naver.com/helloworld/6043