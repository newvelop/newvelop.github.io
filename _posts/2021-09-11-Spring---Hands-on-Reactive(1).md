---
date: 2021-09-11 07:00:39
layout: post
title: "Spring-Hands on Reactive(1)"
subtitle:
description:
image:
optimized_image:
category:
tags:
- Spring
- Webflux
- Hands on Reactive Programming in Spring 5
author: newvelop
paginate: false
---
Hands on Reactive Programming in Spring 5라는 책을 읽고 내용을 요약한다.

### Microservice Architecture
서비스를 사용자들에게 제공할 때, 특정 이벤트를 제공하면 사람들이 평소보다 많이 몰리게 되는 현상이 발생한다. 그러면 평소보다 많은 트래픽이 들어올 경우에 기존 서버로는 처리가 안될 수 있다.

![screensh](../assets/img/2021-09-11-Spring---Hands-on-Reactive(1)/msa.png)

그래서 위와 같이 같은 MSA 구조에서는 같은 애플리케이션을 더 늘리고 로드밸런싱을 통해서 트래픽 분산을 시킬 수 있고, 카프카 같은 기술을 통해 메시지 브로커를 활용하고 이벤트가 들어올 때 메시지 브로커에 넘겨서 이를 다른 서비스에서 처리하는 구조를 가져갈 수 있다. 이 부분은 MSA의 구조적인 처리이고, 앱 자체에서도 리액티브 프로그래밍을 통해 이벤트 기반 처리하는 방법을 알아본다.

### JVM에서의 Reactive Programming
기본적으로 JVM에서 Reactive System을 구현할 수 있는 프레임워크는 Akka와 Vert.x 시스템이다. 하지만 Akka 현재는 JAVA도 지원을 하지만 초반에는 scala기반의 지원이고, Vert.x같은 경우는 Spring 프레임워크와 경쟁을 하다 시장점유율에서 뒤쳐졌다고 한다.

그리고 현재 Spring 진영에서 인기있는 프로젝트는 Spring Cloud 프로젝트이다. 이 프레임워크는 분산시스템을 쉽게 구축할 수 있게 도움을 주기 때문에 MSA 구조가 유행하고 있는 현재 실용성이 높다. 하지만 이렇게 Spring Cloud를 이용하여 분산시스템을 구축한다고 해도, 전체 구조가 Reactive 하기 위해선, 시스템을 이루는 각각의 컴포넌트들이 Reactive해야한다.

![screensh](../assets/img/2021-09-11-Spring---Hands-on-Reactive(1)/blocking.png)

예를 들어서 위와 같은 구조를 지니고 있을 때, OrderService가 process를 처리하기 위해서, ShoppingCardService의 calculate를 호출한다고 해보자. 그때 calculate라는 메소드의 처리 시간이 매우 오래 걸리고, 프로그램이 reactive하지 않을 경우 process를 처리하기 위해서 calculate를 호출하고, 이 calculate를 처리하기 위해서 thread가 블로킹으로 인해 대기해야하는 현상이 벌어지게된다. 그렇게 되면 process에서 부가적인 처리를 해야함에도 불구하고, calculate를 위해 고스란히 기다려야하는 현상이 발생한다.

이를 해결하기 위해서 비동기 처리를 위한 Future 클래스를 JAVA 5에서부터 제공을 한다.

![screensh](../assets/img/2021-09-11-Spring---Hands-on-Reactive(1)/future.png)

위의 코드에서처럼 calculate가 Future타입으로 반환이 되면, calculate를 호출하고 나서, 그 아래 있는 라인들이 calculate가 끝날때까지 기다리는게 아니라, 실행이 된다. 그리고 get을 호출했을때 calculate가 끝났을 경우 결과값을 가져오고, 안끝났을 경우 calculate가 끝날때까지 blocking을 하게 된다. 

Java 8에서는 이보다 좀 더 고급 처리가 가능한데, CompletionStage를 사용하면, Promise 처럼 값을 가져오고 난 뒤에 추가적인 연산이 가능하다.

![screensh](../assets/img/2021-09-11-Spring---Hands-on-Reactive(1)/completionstage.png)

위와 같이 calculate 의 연산이 끝난 결과값을 그대로 가져다가 추가적인 연산을 수행할 수 있다.

하지만 이와 같은 기능은 Spring 4에서는 사용이 불가능했는데, 이는 Spring 4가 JAVA 8보다 낮은 버전을 지원하기 때문에 사용이 불가능했던 것이다. 

그러나 여러가지 필요성으로 인해 스프링 진영에서는 Reactive Programming 지원을 해야했다. 프레임워크 없이 멀티스레딩을 고려하여 프로그래밍을 하기엔 개념이 자체적으로 어려운 측면이 있다. 멀티 스레딩을 이용하여 프로그래밍을 하려면, 다른 스레드들과 메모리 공유하는 관점에서 설계를 해야하고, 에러핸들링같은 문제가 있으며 단일 CPU를 사용할 경우, 여러개의 스레드를 사용할 때 컨텍스트 스위칭이 매우 많이 일어나서 레지스터를 불러오고, 메모리 저장된 내역 불러오는 등의 문제가 많이 존재한다. 그리고 자바의 스레드 자체가 메모리를 많이 차지하는데 JVM의 스레드 한개 스택사이즈가 1024KB이다. 즉 64000개의 요청을 동시 처리하려면 64GB의 메모리를 차지해야하기 때문에 비 효율적이고, 또 스레드 풀을 이용하여 스레드 개수를 제한하고 이를 스위칭해서 사용하자니 클라이언트가 너무 많이 대기해야해서, 이러한 문제점으로 인해 스프링에서 Reactive 관점의 기능을 프레임워크 레벨에서 제공하기로 한다.

### Spring에서의 Reactive Programming
스프링에서 사용할 수 있는 Reactive Programming 기술에 대해서 알아본다.

#### Observer 패턴
옵저버 패턴이란 하나의 주체가 여러개의 옵저버들을 들고있고, 이 주체의 상태가 바뀔 경우 옵저버들에게 상태 변화를 알려주는 패턴이다.

![screensh](../assets/img/2021-09-11-Spring---Hands-on-Reactive(1)/newsletter.png)

위와 같은 예시로 설명하자면, 뉴스사에 유저들이 구독을 하면, 뉴스가 발행될때마다 구독한 유저들에게 발행한 뉴스를 전달해주는 개념이다.

이러한 패턴을 구현하기 위해선 Subject에는 3가지의 메소드가 기본적으로 필요하다.

- 옵저버 등록
- 옵저버 해제
- 옵저버에게 이벤트 알리기

이렇게 옵저버 패턴을 구현할 때, 이벤트 알리는 기능을 반복문을 이용하여 처리하게 된다면, 옵저버의 수만큼 반복을 하는 것이기 대문에 이벤트 알리는 기능이 오래 걸릴 수록, 옵저버 수에 비례하여 느려지게 된다. 이를 해결하기 위해 멀티 스레드로 병렬처리하여 이벤트 알림을 주게 된다면, 스레드 풀을 제한하지 않았을 경우 1mb에 해당하는 스레드가 옵저버 수만큼 생성되어 메모리 오버플로우가 발생할 수 있고, 반대로 스레드 풀로 제한을 하게 된다면, 느려지는 현상과 함께 가용 가능한 스레드가 전부 여기에 사용되었을 때 어플리케이션의 다른 기능이 마비 되는 liveness 관점에서의 문제가 발생하게 된다.

따라서 이 패턴은 현대의 multi thread 관점에서는 문제가 있다고 한다.

#### @EventListener를 이용한 Publish Subscribe 패턴
![screensh](../assets/img/2021-09-11-Spring---Hands-on-Reactive(1)/publish-subscribe.png)

Observer 패턴과는 다르게, 




참고
- Hands on Reactive Programming in Spring 5
- http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.137.9454&rep=rep1&type=pdf