---
date: 2021-03-26 07:00:39
layout: post
title: "Spring-Webflux(1)"
subtitle:
description:
image:
optimized_image:
category:
tags:
- spring boot
- webflux
author: newvelop
paginate: false
---
Spring5에서 사용할 수 있는 Webflux란 무엇이며, 이것이 나타나게 된 계기는 무엇인가에 대해서 살펴보도록한다.

### 기존의 모델
10~15년 전까지는 모노톨릭 서버 구조로, 모든 request를 한 서버에서 처리를 하는 구조를 가지고 있었다.
하지만 처리해야될 데이터가 많아지면서 한 곳에서 모든 서비스 로직을 처리하는 것이 아니라 서비스 별로 어플리케이션을
따로 만들고 서로 통신을 통해 request 를 처리하는 Micro Service Architecture가 등장했다.

Spring을 이용하여 request를 처리하는 서비스를 만들고자 했을 때는 아래와 같은 그림의 처리구조가 일반적이다.

![screensh](../assets/img/2021-03-26-Spring---Webflux(1)/Spring-architecture.PNG)

그림과 같이 Request가 들어오면, tomcat 같은 웹서버가 요청을 받고 filter단에서 한번 거른 후, DispatcherServlet을 통해 처리할 controller를 선별하여 service 단으로 처리를 하는데 DB와의 통신을 하거나 다른 서비스에 api콜을 하여 처리하는 구조이다.

이렇게 되면 각 Request를 어떤 식으로 처리하는지 궁금한데 해당 구조는 아래의 그림과 같다.

![screensh](../assets/img/2021-03-26-Spring---Webflux(1)/request-thread.PNG)

Request마다 thread를 할당하여 request를 처리하는 구조로, request 수만큼 thread가 증가하기 때문에 서버의 메모리 여유에 따라서 서버에 장애가 올 수 있는 구조이다. 물론 모든 서버의 구조가 다 이렇지만, thread의 수가 request 수만큼 비례하여 커지는 부분은 대형 서비스에선 문제라고 볼 수 있다.

따라서 이러한 문제를 해결하기 위해 나온 방법이 async와 non blocking 방식을 이용하는 Webflux 이다.

### Synchronous/Asynchronous와 Blocking/Non Blocking의 개념 정리
개발에서 비동기라는 말을 자주 쓰는데 이 의미에 대해서 잘 모르고 사용하는 경우가 종종 있다. 비동기 프레임워크인 Webflux를 사용함에 앞서 개념을 정리하고 들어가고자 한다.

#### Synchronous
메소드 A가 B를 호출한다고 해보자. 메소드 A에서 B의 결과 반환값을 계속 기다리면서 신경쓰고 있는 성질을 Synchronous라고 한다.

#### Asynchronous
메소드 A가 B를 호출한다고 했을 때, B가 반환을 하든 말든 메소드 A에서 신경쓰고 처리하는 부분이 없는 성질을 Asynchronous라고 한다

#### Blocking
블로킹은 호출된 메소드 기준으로 봐야한다. 메소드 A가 메소드 B를 호출한다고 했을 때, 메소드 A가 다른 일을 하지 못하고, 본인 처리할 때까지 기다리게 하는 개념이라고 보면된다.

#### Non-Blocking
메소드 A가 메소드 B를 호출한다고 했을때, 메소드 A가 다른 일을 할 수 있도록 놓아주는 개념이라고 보면된다.

이렇게만 말해선 사실 잘 모를거같은데 실제 비유하는 걸로 정리를 해보고자 한다.

- Sync + Blocking : 친구 A와 B가 메신저를 통해 메시지를 한다고 가정을 했을때, '밥 먹었어?' 라고 A가 보냈으면 B가 답장을 할 때까지 친구 A는 안절부절 못하면서 기다리고 딴 일을 못하는 경우라고 보면된다.

- Sync + Non-Blocking : 친구 A가 친구 B에게 메시지를 보냈을때, A는 메시지가 오기까지 다른 일을 할 수 있지만, 계속 오는지 핸드폰을 보고 있는 상태라고 보면 된다.

- Async + Blocking : 친구 A가 친구 B에게 메시지를 보냈을 때, A는 메시지가 오든 말든 신경을 쓰진 않지만, 메시지를 대기하는것 말고는 아무것도 할 수 없는 상태라고 보면된다.

- Async + Non-Blocking : 친구 A가 친구 B에게 메시지를 보냈을 때, A는 메시지 오든 말든 신경안쓰고 있고, TV를 보든 뭘 하든 다른 일을 하고 있는 일반적인 경우라고 보면된다.

즉 우리가 흔히 말하는 비동기라는 성질은 Async + Non-Blocking이라고 보면 된다.


### Publisher/Subscriber
Reactive Programming이라고 하는 비동기 프로그래밍을 하고자 할 경우, Publisher/Subscriber 패턴을 알아야 한다고 한다

- Publisher : 데이터를 제공해주는 친구. DB 등등

- Subscriber : 데이터를 읽고자 하는 친구. DB에 요청하는 서비스 라고 보면 될듯.

![screensh](../assets/img/2021-03-26-Spring---Webflux(1)/pub-sub.PNG)

위의 그림과 같이 구조가 흘러가는데, 먼저 데이터가 필요한 subscriber가 publisher에 subscribe를 호출하면 publisher는 subscriber를 받아들인다. 이게 성사될 경우 subscriber는 필요한 데이터 요청을 하는데 여기서 n의 파라미터를 전달하여 back-pressure(데이터 흐름 제어)를 할 수 있다. 왜 필요하냐면 필요한 것 이상으로 데이터가 생성될 수 있기 때문에 데이터 양을 직접 설정한다고 한다. 이는 나중에 찾아보는 걸로.
이렇게 request를 보내면 non-blocking이기 때문에 subscriber는 딴거 하고 있고, 이러한 동안, publisher는 데이터를 계속 보내는 형태로 보면 된다. 이러다가 필요한 데이터를 전부 보냈을 경우 onComplete으로 끝낸다고 보면 된다.


참고
- build-reactive-restful-apis-using-spring-boot-webflux 강좌