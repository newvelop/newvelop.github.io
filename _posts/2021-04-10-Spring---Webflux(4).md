---
date: 2021-04-10 07:00:39
layout: post
title: "Spring-Webflux(4)"
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
Webflux에서 사용하는 Project Reactor에서, Flux를 기반으로 에러 핸들링, 백 프레셔 등등에 대해 알아본다

### 에러 핸들링
데이터 스트림을 다루다 보면 중간에 에러가 발생할 수 있다. 이와 관련하여 몇가지 처리방법을 알아본다.

#### onErrorResume
에러가 발생할 경우, 에러를 처리한 이후에 다시 이어갈 flux를 반환하여 스트림을 이어가는 메소드이다.

```
Flux<String> stringFlux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("exception")))
    .concatWith(Flux.just("D"))
    .onErrorResume((e) -> {
        System.out.println("exception is " + e);
        return Flux.just("default", "default1");
    });
```

위와 같이 코드를 작성했을 경우, exception이 ABC 처리 이후에 발생을 하는데 exception이 onErrorResume에서 잡혀서, 에러 관련 출력을 한후 다시 새로운 flux를 반환하여 스트림을 이어가 결국에 5개의 데이터를 반환하게 된다.

#### onErrorReturn
에러가 발생되었을 경우 값 반환하고 스트림 끝내는 처리방법이다.

```
Flux<String> stringFlux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("exception")))
    .concatWith(Flux.just("D"))
    .onErrorReturn("test");
```
ABC를 반환하다 에러가 발생했을 경우, test를 반환하고 스트림이 종료된다.

#### onErrorMap
에러가 발생했을 경우, 이를 새로운 예외로 매핑해서 처리하는 기법이다.
```
Flux<String> stringFlux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("exception")))
    .concatWith(Flux.just("D"))
    .onErrorMap((e) -> new CustomException(e));
```

해당 방법은 어플리케이션에서 비즈니스 예외 처리를 정의해놨을 경우, 매핑해서 사용하면 될 것이다.

#### retry
에러가 발생했을 경우, 스트림을 지정된 횟수만큼 시도하는 방법이다

```
 Flux<String> stringFlux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("exception")))
    .concatWith(Flux.just("D"))
    .retry(2);
```
이렇게 처리할 경우, 결국에 ABC를 기존 말고 두번 더 시도해서 총 9개의 문자열이 출력된다.

#### retryBackoff
에러가 발생했을 경우, 지정된 횟수만큼 시도를 하는 것인데, 시도하는 텀을 두고 수행하는 방법이다. 

```
Flux<String> stringFlux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("exception")))
    .concatWith(Flux.just("D"))
    .onErrorMap((e) -> new CustomException(e))
    .retryBackoff(2, Duration.ofSeconds(2));
```

에러가 발생했을 경우, 바로 처리가 안되는 경우가 많기 때문에 텀을 두고 처리하면 좋을 것 같다.

### 시간을 이용한 생성 및 지연 방법
데이터를 무제한으로 넣어서 flux를 생성하는 방법이 있다.

```
Flux<Long> infiniteFlux = Flux.interval(Duration.ofMillis(200))
    .log();
```
이렇게 생성을 할 경우, 0.2초 간격으로 0부터 1씩 증가하는 데이터를 꾸준히 생성한다. 이때 주의할 점이 테스트를 할때,

```
infiniteFlux
    .subscribe((element) -> System.out.println("value is : " + element));
```
이렇게 처리할 경우 값이 출력이 안되는데 이는 비동기/논블로킹 방식이기 때문에, 호출했던 스레드가 먼저 subscribe 시켜놓고 종료되어버려서 데이터 스트림이 생성되고 반환되는 것을 확인할 수 없다.

```
Thread.sleep(3000);
```
위와 같이 스레드를 대기시켜 살려놓는 방법으로 테스트를 해볼 수 있다.

뿐만 아니라, 지정된 개수만큼 값을 생성할 수 있는데, 이는 take() 함수를 flux에서 호출해주고 인자로 원하는 개수를 넘겨주면 된다.

### 백프레셔
백프레셔라 함은 publisher가 subscriber에게 데이터 반환하는 수를 제한하는 것을 의미한다.

![screensh](../assets/img/2021-04-01-Spring---Webflux(3)/nobackpressure.PNG)

일반적으로 백프레셔를 설정하지 않으면, 위와 같이 모든 데이터를 차례로 반환한다.

![screensh](../assets/img/2021-04-01-Spring---Webflux(3)/backpressure.PNG)

백프레셔를 설정해서 반환하는 개수를 지정하여 그 개수씩 반환을 받을 수 있다.

```
Flux<Integer> finiteFlux = Flux.range(1, 10)
        .log();

finiteFlux.subscribe((element) -> System.out.println("element is " + element)
        , (e) -> System.err.println("exception is : " + e)
        , () -> System.out.println("done")
        , (subscription -> subscription.request(2)));
```
첫 번째 방법으로는 subscription에 직접 request 개수를 지정하는 방법이 있고, 두 번째로는

```
Flux<Integer> finiteFlux = Flux.range(1, 10)
        .log();
finiteFlux.subscribe(new BaseSubscriber<Integer>() {
    @Override
    protected void hookOnNext(Integer value) {
        request(1);
        System.out.println("value : " + value);
        if (value == 4) {
            cancel();
        }
    }
});
```

BaseSubscriber를 구현하여, hookOnNext메소드를 구현하고 여기에 개수를 지정해서 날려주는 방법이 있다.


### HOT/COLD publisher
스트림에 hot과 cold publihser가 있다고 한다. 
```
Flux<String> stringFlux = Flux.just("A", "B", "C", "D", "E", "F")
        .delayElements(Duration.ofSeconds(1));

stringFlux.subscribe(s -> System.out.println("subscriber 1 : " + s));  //A부터 시작
Thread.sleep(2000);

stringFlux.subscribe(s -> System.out.println("subscriber 2 : " + s));  //A부터 시작
Thread.sleep(4000);
```

같은 flux를 두 번 subscriber를 해도 위와 같이 구현했을 경우는, 새로운 publish가 발생하여 두 경우 다 A부터 시작하는 스트림을 받게 된다.

```
Flux<String> stringFlux = Flux.just("A", "B", "C", "D", "E", "F")
        .delayElements(Duration.ofSeconds(1));

ConnectableFlux<String> connectableFlux = stringFlux.publish();
connectableFlux.connect();

stringFlux.subscribe(s -> System.out.println("subscriber 1 : " + s));  //A부터 시작
Thread.sleep(2000);

connectableFlux.subscribe(s -> System.out.println("subscriber 2 : " + s));  //A부터 시작
Thread.sleep(4000);
```

반면 위와 같이 ConnectableFlux를 이용하여 구현했을 경우는, 현재 publish중인 데이터 스트림을 동시에 이용하여 현재 진행중인 데이터를 동시에 받게 된다.

![screensh](../assets/img/2021-04-01-Spring---Webflux(3)/hotpublisher.PNG)

hot publisher의 구조는 위의 그림과 같다.

참고
- build-reactive-restful-apis-using-spring-boot-webflux 강좌
- https://www.slideshare.net/Trayan_Iliev/spring-5-webflux-advances-in-java-2018