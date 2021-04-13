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
Webflux에서 Rest API를 구현하는 방법을 알아본다.

### @RestController를 이용하여 만들기
일반적으로 Spring boot에서 컨트롤러를 만드는 @RestController 어노테이션을 사용하여 non blocking api를 만들어본다.

```
@GetMapping("/flux")
public Flux<Integer> getFlux() {
    return Flux.just(1,2,3,4)
            .log();
}
```
위와 같이 flux를 이용하여 반환하는 get방식의 메소드를 구현할 경우, 1,2,3,4를 브라우저에서 올바르게 반환한다. 하지만 아래와 같이

```
@GetMapping("/flux/delay")
public Flux<Integer> getDelayFlux() {
    return Flux.just(1,2,3,4)
            .delayElements(Duration.ofSeconds(1))
            .log();
}
```
delay를 줄 경우, 브라우저에선 비동기로 실행되는 것이 아니라 동기로 실행되어 모든 요소들을 받을 때까지 기다렸다가 렌더링을 하기 때문에 4초 흐른 뒤, 1,2,3,4를 반환한다.

이를 요소 하나씩 받을때 마다 렌더링을 하고자 할 경우

```
@GetMapping(value = "/flux/stream", produces = MediaType.APPLICATION_STREAM_JSON_VALUE)
public Flux<Long> getFluxStream() {
    return Flux.interval(Duration.ofSeconds(1))
            .log();
}
```
이런 식으로 MediaType을 stream 타입으로 부여하여 반환하면, 브라우저에선 스트림으로 인식하여 값이 들어올때마다 표시한다.

### @RestController 테스트
@RestController를 이용하여 구현한 클래스를 테스트하는 법을 설면한다

```
@RunWith(SpringRunner.class)
@WebFluxTest
public class FluxAndMonoControllerTest {
}
```
Junit 4를 사용할 경우, 위와 같이 @RunWith 어노테이션과 @WebFluxTest 어노테이션을 사용하여 클래스를 작성한다.

```
    @Autowired
    WebTestClient webTestClient;

    @Test
    public void flux_approach1() {
        Flux<Integer> integerFlux = webTestClient.get().uri("/flux")
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .exchange()
                .expectStatus().isOk()
                .returnResult(Integer.class)
                .getResponseBody();

        StepVerifier.create(integerFlux)
                .expectSubscription()
                .expectNext(1, 2 ,3, 4)
                .verifyComplete();
    }
```

그 다음 @Autowired 어노테이션을 통해 WebTestClient 객체 의존성 주입을 한다. 그리고 @Test 어노테이션을 통해 테스트 하고자 하는 메소드임을 표시하고, 그안에서 WebTestClient를 이용하여 실제로 uri를 실행하는 코드를 작성한다. 그리고 responseBody를 받아 기대에 맞는 body가 들어왔는지 검증한다.

* 주의사항 : test 클래스는 Springboot의 Main 클래스가 담긴 패키지 하위에 존재해야 실행된다. 위치가 잘못되었을 경우 에러를 발생시킴

```
    @Test
    public void fluxStream() {
        Flux<Long> longStreamFlux = webTestClient.get().uri("/flux/stream")
                .accept(MediaType.APPLICATION_STREAM_JSON)
                .exchange()
                .expectStatus().isOk()
                .returnResult(Long.class)
                .getResponseBody();

        StepVerifier.create(longStreamFlux)
                .expectNext(0L, 1L, 2L)
                .thenCancel()
                .verify();
    }
```
위에서 구현한 stream을 반환하는 메소드 테스트를 하고 싶을 경우, Mediatype을 바꿔주고 테스트 하면 된다.


### RouterFunction을 사용할 경우
Webflux에선 RouterFunction을 이용하여 컨트롤러 매핑을 이용하지 않고, 직접 라우팅을 할 수 있다. 

```
@Component
public class SampleHandlerFunction {
    public Mono<ServerResponse> flux(ServerRequest serverRequest) {
        return ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .body(
                        Flux.just(1,2,3,4)
                        .log(), Integer.class
                );
    }

    public Mono<ServerResponse> mono(ServerRequest serverRequest) {
        return ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .body(
                        Mono.just(1)
                                .log(), Integer.class
                );
    }
}
```
위와 같이 커스텀 클래스를 만들어서 컴포넌트로 등록하고, ServerRequest를 매개변수로 하여 request를 처리하는 메소드를 각각 구현한다.

```
@Configuration
public class SampleRouterFunctionConfig {
    @Bean
    public RouterFunction<ServerResponse> route(SampleHandlerFunction sampleHandlerFunction) {
        return RouterFunctions
                .route(GET("/functional/flux").and(accept(MediaType.APPLICATION_JSON)), sampleHandlerFunction::flux)
                .andRoute(GET("/functional/mono").and(accept(MediaType.APPLICATION_JSON)), sampleHandlerFunction::mono);
    }
}
```
그 다음 위와 같이 Config파일을 생성하여 @Configuration 어노테이션을 통해 설정으로 등록하고, RouterFunction 객체를 반환하는 메소드를 빈으로 등록하여 RouterFunction에 각각 url의 라우트 정보를 등록하여 메소드로 매핑을 해주면, 해당 url이 들어왔을때 매핑된 메소드를 실행되게해준다.

### RouterFunction 테스트
RouterFunction을 이용하여 구현한 메소드를 테스트 하고 싶을 경우, 컨트롤러 테스트와는 조금 구현 방식이 다르다.

```
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureWebTestClient
public class SampleHandlerFunctionTest {
}
```
우선 위와 같이 클래스를 작성하는데, 컨트롤러 테스트와는 다르게 @WebFluxTest를 사용하지 않은 이유는 해당 어노테이션이 컨트롤러 기반으로 테스트하는 어노테이션으로 WebTestClient에 의존성을 주입해주는데, 이 경우는 컨트롤러가 없기 때문에 의존성 주입에 실패를 하게된다. 대신 @AutoConfigureWebTestClient 어노테이션을 사용하여, @Bean으로 등록한 RouterFunction에 대한 의존성 처리를 해서 WebTestClient를 주입해준다.


```
    @Autowired
    WebTestClient webTestClient;

    @Test
    public void flux_approach1() {
        Flux<Integer> integerFlux = webTestClient.get().uri("/functional/flux")
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .exchange()
                .expectStatus().isOk()
                .returnResult(Integer.class)
                .getResponseBody();

        StepVerifier.create(integerFlux)
                .expectSubscription()
                .expectNext(1, 2 ,3, 4)
                .verifyComplete();
    }
```

나머지 구현 방식은 컨트롤러 테스트와 같다.

참고
- build-reactive-restful-apis-using-spring-boot-webflux 강좌
