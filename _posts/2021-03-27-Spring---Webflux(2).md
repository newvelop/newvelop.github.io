---
date: 2021-03-27 07:00:39
layout: post
title: "Spring-Webflux(2)"
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
Webflux에서 사용하는 Project Reactor에서, 데이터를 다룰수 있는 Flux, Mono, Reactive Stream에 대해 알아본다.

### Flux
이전의 포스트에서 Publisher는 데이터를 제공하는 개념이라고 했다.
이 Publisher가 데이터를 제공할 시 onNext를 통해 하나씩 읽어들여서 데이터를 반환하는데 이 때 데이터가 0개 이상의 N개 데이터 일 경우, 이 데이터를 다루는 객체를 Flux라고 칭한다.

![scrrensh](../assets/img/2021-03-27-Spring---Webflux(2)/flux.PNG)

### Mono
Publisher가 제공하는 데이터가 0~1개일 경우, 이 데이러를 다루를 객체를 Mono라고 칭한다.

![scrrensh](../assets/img/2021-03-27-Spring---Webflux(2)/mono.PNG)

### 코드 기반 개념 학습
gradle 기반으로 Webflux 테스트를 하고자 할 경우, 하단의 코드와 같은 의존성 설정을 한다
```
buildscript {
    ext {
        springBootVersion = '2.1.1.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group '본인프로젝트 정보'
version '본인프로젝트 정보'
sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

test {
    exclude 'com/learnreactivespring/fluxandmonoplayground/**'
}

dependencies {
    implementation('org.springframework.boot:spring-boot-starter-data-mongodb-reactive')
    implementation('org.springframework.boot:spring-boot-starter-webflux')
    compileOnly('org.projectlombok:lombok')
    testImplementation('org.springframework.boot:spring-boot-starter-test')
    testImplementation('de.flapdoodle.embed:de.flapdoodle.embed.mongo')
    testImplementation('io.projectreactor:reactor-test')
}
```

#### 일반적인 Flux 및 Mono 예제
String 데이터를 Publisher가 제공한다 생각하고, String들을 Flux로 다룬다고 할경우, 하단과 같이 코드를 작성한다.

```
    Flux<String> stringFlux = Flux.just("Spring", "Spring boot", "Reactive Spring");
```

여기에서 각 데이터에 concat 처리를 하고 싶을 경우,

```
    Flux<String> stringFlux = Flux.just("Spring", "Spring boot", "Reactive Spring");
        .concatWith(Flux.just("붙일 단어"));
```
이렇게 처리하면 된다.

Mono의 경우, 
```
    Mono<String> stringMono = Mono.just("Spring");
```
이런식으로 코드를 작성하면 된다.

위와 같은 케이스들은 Publisher의 데이터를 가지고 있는 객체를 생성한 것이고, Subscriber를 따로 구현을 안했기 때문에 실제 동작하는 부분은 현 코드로 확인할 수 없다.

만약 현 단계에서 동작 테스트를 하고싶을 경우, test 폴더에 클래스들을 생성하고
```
@Test
    public void fluxTestElementsWithError() {
        Flux<String> stringFlux = Flux.just("Spring", "Spring boot", "Reactive Spring")
                .concatWith(Flux.error(new RuntimeException("exception")))
                .log();

        StepVerifier.create(stringFlux)
                .expectNext("Spring")
                .expectNext("Spring boot")
                .expectNext("Reactive Spring")
                .expectErrorMessage("exception")
                .verify();
    }
```
이런식으로 코드를 작성하면 StepVerifier의 verify 메소드가 subscribe 역할을 하기 때문에 동작을 하게 된다. 위의 코드는 exception이 발생하기 때문에 verify를 호출한 것이고, exception이 없을 경우는 verifyComplete을 호출한다.

.log()함수는 데이터가 처리되는 과정을 로깅하는 메소드 이기때문에 실제 돌아가는 과정을 보고싶을 경우 호출하면 된다.

#### 여러 데이터 포맷에서의 Flux 및 Mono 예제
위의 챕터에선 단순 String 배열을 처리하는 예제를 생성했는데, Stream, Supplier, Collection에서 생성하는 예제를 살펴본다.

우선 Collection에서 Flux를 생성하려면

```
Flux<String> nameFlux = Flux.fromIterable('컬랙션 객체');
```
이런식으로 생성하면된다.

stream에서 생성하고 싶은 경우는
```
Flux<String> nameFlux = Flux.fromStream(names.stream());
```
이런식으로 생성한다.

Supplier에서 Mono로 처리하고 싶을 경우,
```
Mono<String> stringMono = Mono.fromSupplier(stringSupplier);
```
이렇게 처리하면 된다.

#### 데이터 필터링 및 병합에서의 Flux 및 Mono 예제
데이터들을 필터링하고 싶거나 여러 데이터를 병합하고 싶을 수 있다. 그에 대한 예제를 살펴보자

먼저 필터링의 경우,
```
 Flux<String> nameFlux = Flux.fromIterable(names)
                .filter(s -> s.startsWith("a"));
```
이런 식으로, stream의 filter 메소드를 호출하여 처리하면 된다.

병합의 경우는 여러가지 방법이 있는데 먼저 단순 병합의 경우는,
```
    Flux<String> flux1 = Flux.just("A", "B", "C");
    Flux<String> flux2 = Flux.just("D", "E", "F");

    Flux<String> mergedFlux = Flux.merge(flux1, flux2).log();
```
이런 식으로 merge 메소드를 호출해서 처리하면 된다.

여기서 각 데이터의 지연을 설정하여 천천히 데이터를 병합하고 싶을 경우,
```
    Flux<String> flux1 = Flux.just("A", "B", "C").delayElements(Duration.ofSeconds(1));
    Flux<String> flux2 = Flux.just("D", "E", "F").delayElements(Duration.ofSeconds(1));

    Flux<String> mergedFlux = Flux.merge(flux1, flux2).log();
```
이렇게 delayElements메소드를 호출하면 되는데 여기서 결과가 조금 특이하다.
동일한 지연 시간을 설정했기 때문에 FLUX 에서 데이터가 섞여서 들어오는데 A,D,B,E,C,F 이런 식으로 들어올 때가 있고, D,A,E,B,F,C 이렇게 들어올 때가 있다. 즉 순서가 보장되지 않으며 FLUX 들에서 섞여서 들어온다.

concat을 통해서 이어붙이는 경우도 있는데,
```
    Flux<String> flux1 = Flux.just("A", "B", "C").delayElements(Duration.ofSeconds(1));
    Flux<String> flux2 = Flux.just("D", "E", "F").delayElements(Duration.ofSeconds(1));

    Flux<String> mergedFlux = Flux.concat(flux1, flux2).log();
```
이런 식으로 처리하면 된다. 이 경우는 merge와 다른 결과가 나타나는데 결론적으로는 무조건 A,B,C,D,E,F 가 보장이 된다. 왜냐하면 concat으로 인해서 앞에 flux가 먼저 처리되기 때문이다.

zip이란 것도 있는데 zip은 각 원소별로 처리하는 메소드라고 보면 된다.
```
    Flux<String> flux1 = Flux.just("A", "B", "C");
    Flux<String> flux2 = Flux.just("D", "E", "F");

    Flux<String> mergedFlux = Flux.zip(flux1, flux2, (t1, t2) -> t1.concat(t2)).log();
```
이런 식으로 처리를 할 경우, AD, BE, CF 이게 결과로 들어오게 된다. 즉 각 원소별 zip을 할 수 있는 것이다.

#### 데이터 변환에서의 Flux와 Mono 예제
java stream을 사용한 사람들은 알겠지만 stream을 이용하여 변환 처리를 하고 다시 묶는 경우가 있다. 이에 대한 예제를 Flux와 Mono에서 살펴본다.

먼저 단순하게 map을 사용하여 
```
Flux<Integer> nameFlux = Flux.fromIterable(names)
                .map(s -> s.length())
                .filter(v -> v > 1)
```
이렇게 데이터 형 변환도 할 수 있고, 변환한 데이터에 대한 필터링을 적용시킬 수 도있다.

또한 stream에 대한 반복 연산을 할 수 있는데
```
Flux<Integer> nameFlux = Flux.fromIterable(names)
                .map(s -> s.length())
                .filter(v -> v > 1)
                .repeat(1)
```
이렇게 repeat을 호출하면, 기존 결과 + repeat 횟수만큼의 길이가 된다.

Flatmap을 또한 사용할 수 있다. Flatmap과 map의 차이점을 살펴보면, map은 데이터 stream의 값 하나에 대한 처리를 하는 것인 반면에, Flatmap의 경우는 값 하나가 들어올 경우, 이 값을 가장 작은 stream으로 반환하는 형태이다.

간단한 예를 들자면, 
```
        String[][] sample = new String[][]{
                {"a", "b"}, {"c", "d"}, {"e", "a"}, {"a", "h"}, {"i", "j"}
        };
        Arrays.stream(sample)
                .flatMap(array -> Arrays.stream(array));
```
이런 식으로 코드를 구현하면, 제일 큰 Arrays.stream이 반환하는 것은 Stream&lt;String[]&gt; 이 아니라
Stream&lt;String&gt; 형태로 풀어서 반환을 한다.

thread를 써서 병렬 처리를 하고싶을 경우, 

```
 Flux<String> nameFlux = Flux.fromIterable(Arrays.asList("A", "B", "C", "D", "E", "F"))
                .window(2)
                .flatMap((s) ->
                    s.map(this::convertToList).subscribeOn(parallel()))
                    .flatMap(s -> Flux.fromIterable(s));
```
이런 식으로 처리를 하면 된다(convertToList는 본인이 구현한 메소드)
근데 이런식으로 하면 순서 보장이 되질 않는다. (Thread로 인해)

순서 보장을 원한다면 flatMapSequential 메소드를 flatMap 대신 호출한다.

참고
- build-reactive-restful-apis-using-spring-boot-webflux 강좌