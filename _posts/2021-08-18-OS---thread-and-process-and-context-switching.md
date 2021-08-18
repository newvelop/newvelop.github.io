---
date: 2021-08-18 07:00:39
layout: post
title: "OS-thread and process and context switching"
subtitle:
description:
image:
optimized_image:
category:
tags:
- OS
- thread
- process
- context switching
author: newvelop
paginate: false
---
이 포스트에선 OS의 스레드와 프로세스, 그리고 컨텍스트 스위칭과 임계영역등 여러가지 개념에 대해서 살펴본다.

### 프로세스와 스레드

#### 프로세스
프로세스란 메모리에 올라와서 실행되고 있는 프로그램을 의미한다. 또한 OS에서 시스템 자원을 할당받는 작업의 단위다.

![screensh](../assets/img/2021-08-18-OS---thread-and-process-and-context-switching/process.png)

프로세스는 Code, Data, Stack, Heap 영역을 할당받아서 독립적으로 사용한다. 즉 다른 프로세스들과 공유하지 않는다. 또한 프로세스는 최소 1개의 스레드를 가지고 있다.

#### 스레드
스레드는 프로세스의 특정한 수행경로를 의미하며, 프로세스가 할당받은 자원을 이용하는 실행 단위이다.

![screensh](../assets/img/2021-08-18-OS---thread-and-process-and-context-switching/thread.png)

스레드는 프로세스와는 다르게, 스택을 제외한 모든 영역을 공유한다. 즉 프로세스내의 스레드들끼리는 스택을 제외한 데이터를 공유한다는 소리이다.

#### 멀티 프로세스
멀티 프로세스란 하나의 프로그램을 여러개의 프로세스로 구성하여 실행하는 것을 의미한다. 여러 프로세스가 처리하기 때문에 프로세스 하나가 죽는다고 해도 프로그램 자체가 죽지는 않는다. 다만 Context Swithching을 할 때 프로세스끼리는 영역을 공유하지 않기 때문에 모든 상태를 바꿔줘야해서 오버헤드가 심하다. 

#### 멀티 스레드
하나의 응용프로그램을 여러개의 스레드로 구성하고 프로그램을 실행하는 구조를 의미한다. 프로세스보다 스레드가 가볍기 때문에 시스템 자원 소모를 멀티 프로세스에 비해 줄일 수 있고, Context Switching이 더 빠르다. 다만 디버깅이 어렵고, 스레드끼리는 스택을 뺀 자원을 공유하기 때문에 공유자원에 대한 데이터 동기화를 고려하여 설계해야한다.

### mutex & semaphore
스레드의 경우, 스택을 제외한 나머지 영역을 공유하기 때문에 동기화 문제를 해결해야한다고 했다. 이 동기와 문제를 해결하기위한 방법을 소개한다.

#### mutex
자원에 대한 동기화를 하기 위해 사용되는 상호배제 기술이다. 동기화를 하기 위한 임계영역을 정하고, 이 영역에 처음으로 들어오는 스레드는 lock을 건다. 이러면 lock이 풀리기 전까지는 다른 스레드는 대기한다. 그러다가 lock을 건 스레드가 lock을 풀고 나오면 다른 스레드가 lock을 걸고 들어가는 구조이다. 

#### Semaphore
세마포어의 경우도 상호배제를 하는 기술이지만 mutex와는 조금 다르다. mutext는 하나의 스레드만 진입하면서 lock을 거는 구조지만, semaphore의 경우는 여러 스레드가 진입할 수 있다. 예를 들어서 3을 max로 걸었을 경우, 스레드 1,2,3 까지 진입할 수 있다. 3이 들어가는 순간 lock이 걸리게 된다. 그러면 4번째 오는 스레드는 대기를 해야한다. 그러다가 lock을 걸지 않은 스레드1이 작업을 종료하면 lock을 풀어주게 되고 4번째 스레드는 진입할 수 있게 되는 구조이다.

참고
- https://gmlwjd9405.github.io/2018/09/14/process-vs-thread.html
- https://worthpreading.tistory.com/90