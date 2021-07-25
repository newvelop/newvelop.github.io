---
date: 2021-07-26 07:00:39
layout: post
title: "Spring-Object and dependency"
subtitle:
description:
image:
optimized_image:
category:
tags:
- Spring
- Toby의 스프링
author: newvelop
paginate: false
---
토비의 스프링을 읽고 이해한 내용을 바탕으로 요약한다.

### 오브젝트와 의존관계
#### DB 커넥션 기능 구현을 토대로 하는 스프링의 구조 이해

DB를 연결하고 쿼리를 날려서 데이터를 가져오는 기능을 구현하면 이 모든 기능을 한 메소드에 넣고, 설정 값을 넣어서 특정 타입의 DB에 연결하는 코드를 구현 할 수 있다. 하지만 이렇게 구현을 하게 된다면 DB를 다른 제품을 사용한다거나 설정값이 달라지거나 했을 경우 영향을 받는 부분이 상당히 많아지기 때문에 유지보수 측면으로 봤을때 성능이 나쁜 코드가 된다. 이를 기능별로 분리해서 설계하는 방법을 살펴본다.

![screensh](../assets/img/2021-07-26-Spring---Object-and-dependency/dao-firstcode.png)

상단의 코드는 토비의 스프링 서적에서 작성된 코드이며, 사용자의 추가와 조회의 기능을 수행하는 메소드 두개가 구현되어있으며, 각 메소드에선 커넥션을 연결하고 쿼리를 수행하는 코드까지 한번에 구현되어있다.

프로그래밍의 기초 개념 중, 관심사의 분리라는 개념이 있는데 관심이 같은 것들끼리는 하나의 객체로, 또는 친한 객체로 모이게 하며 관심이 다른 것들은 가능한 따로 떨어져 서로 영향을 주지 않도록 분리하게 설계를 한다는 개념이다. 이 관점으로 봤을때 위의 코드는 다양한 기능이 한 곳에 담겨 있는 나쁜 코드라고 볼 수 있다.

해당 DAO의 관심사항은 크게 세가지로 나눌 수 있다.
1. DB와의 연결
2. 쿼리 수행
3. 리소스 관리

이 관심사를 기준으로 코드를 분리하는 방법을 살펴본다

##### 커넥션 분리

일단 커넥션 기능을 분리하는 관점에서 코드를 분리하자면 상단과 같이 구현을 할 수 있다.

![screensh](../assets/img/2021-07-26-Spring---Object-and-dependency/connection-separate.png)

하지만 이렇게 구현했을 경우, 다른 방법으로 커넥션을 가져와야한다면, 코드 수정이 불가피한 코드가 된다. 따라서 이를 연결하는 방법을 추상화하는 방법으로 제공을 하면 getConnection 부분은 그대로 가져가고 사용자가 필요한 방식으로 구현해서 클래스 구현을 완료하면 된다.

![screensh](../assets/img/2021-07-26-Spring---Object-and-dependency/connection-abstract.png)
![screensh](../assets/img/2021-07-26-Spring---Object-and-dependency/connection-implement.png)

이런 식으로 구현을 하면 UserDao의 조회나 저장 방식과는 상관없이 connection 부분을 이용할 수 있으며, 각 사용자의 방식에 따라 connection을 구현하여 사용하면 된다. 이렇게 기능의 일부를 추상 메소드나 오버라이딩이 가능한 메소드로 만든뒤 서브클래스에서 구현하여 사용하는 방법을 템플릿 메소드 패턴이라고 한다. 그리고 또한 UserDao의 서브 클래스인 NUserDao와 DUserDao에서 Connection 객체를 어떻게 생성할지를 판단하고 생성하는 구조이기 때문에 팩토리 메소드 패턴을 적용한것이라고 볼 수 있다.

* 템플릿 메소드 패턴 : 알고리즘의 구조를 메소드에 정의하고 하위 클래스에서 알고리즘 구조의 변경없이 알고리즘만 재정의 하는 패턴이라고 보면 된다.

![screensh](../assets/img/2021-07-26-Spring---Object-and-dependency/template-method-pattern.png)

* 팩토리 메소드 패턴 : 팩토리 메서드 패턴(Factory method pattern)은 객체지향 디자인 패턴이다. Factory method는 부모(상위) 클래스에 알려지지 않은 구체 클래스를 생성하는 패턴이며. 자식(하위) 클래스가 어떤 객체를 생성할지를 결정하도록 하는 패턴이기도 하다. 부모(상위) 클래스 코드에 구체 클래스 이름을 감추기 위한 방법으로도 사용한다.
![screensh](../assets/img/2021-07-26-Spring---Object-and-dependency/factory-method-pattern.png)

하지만 이렇게 분리를 한다고 하더라도, UserDao에 


참고
- 토비의 스프링
- https://yaboong.github.io/design-pattern/2018/09/27/template-method-pattern/
- https://ko.wikipedia.org/wiki/%ED%8C%A9%ED%86%A0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C_%ED%8C%A8%ED%84%B4















```
dependencies {
	implementation('org.springframework.boot:spring-boot-starter-data-mongodb-reactive')
	implementation('org.springframework.boot:spring-boot-starter-webflux')
	compileOnly('org.projectlombok:lombok')
	testImplementation('org.springframework.boot:spring-boot-starter-test')
	testImplementation('de.flapdoodle.embed:de.flapdoodle.embed.mongo')
	testImplementation('io.projectreactor:reactor-test')
}
```
위와 같은 식으로 의존성을 연결하여 구현 및 테스트를 할 수 있게 설정을 한다.

그리고 연결을 위한 DB의 URL 및 포트를 설정해야하는데 이는 applciation.yml 혹은 application.properties에 설정을 한다. 본인은 이번에 yml을 사용하도록 한다.

```
    spring:
    profiles:
        active: nonprod
    ---
    spring:
    profiles: dev
    data.mongodb:
        host: localhost
        port: 27017
        database: local
    ---
    spring:
    profiles: nonprod
    data.mongodb:
        host: localhost
        port: 27017
        database: local
    ---
```
이런 식으로 yml에 profile별 연결 설정을 ---으로 구분을 해주고 각 profile별 연결설정을 해주면 연결은 자동으로 한다.

#### DB 연결을 위한 클래스 생성
DB와 주고받을 데이터를 담을 클래스를 생성한다.

```
@Document
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Item {

    @Id
    private String id;
    private String description;
    private Double price;
}
```
2,3,4 번재의 어노테이션의 lombok의 어노테이션으로, 해당 라이브러리를 사용하지 않을 경우, getter setter 및 생성자를 따로 정의 해주면 된다.


```
public interface ItemReactiveRepository extends ReactiveMongoRepository<Item,String> {

}
```
그리고 위와 같이 데이터를 주고받는 작업을 수행할 repository를 생성한다.

#### Controller 생성
API를 구현하기 위한 클래스를 생성한다.
```
@RestController
@Slf4j
public class ItemController {
}
```

그리고 데이터의 CRUD를 수행할 Repository의 의존성을 주입한다. 원래는 service 클래스를 따로 만들어 비즈니스로직을 서비스에서 수행을 하는 것이 좋은 코드이나 이번엔 실행하는 것이 목적이기 때문에 그냥 구현을 하도록 한다.

```
private final ItemReactiveRepository itemReactiveRepository;

    public ItemController(ItemReactiveRepository itemReactiveRepository) {
        this.itemReactiveRepository = itemReactiveRepository;
    }
```

그리고 아래와 같이 endpoint로 사용할 string 들을 상수들로 해서 변경에 대응할 수 있도록 선언한다.
```
public class ItemConstants {
    public static final String ITEM_END_POINT_V1 = "/v1/items";
}
```

```
    @GetMapping(ItemConstants.ITEM_END_POINT_V1)
    public Flux<Item> getAllItems() {
        return itemReactiveRepository.findAll();
    }

    @GetMapping(ItemConstants.ITEM_END_POINT_V1 + "/{id}")
    public Mono<ResponseEntity<Item>> getOneItem(@PathVariable String id) {
        return itemReactiveRepository.findById(id)
                .map(item -> new ResponseEntity<>(item, HttpStatus.OK))
                .defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }
```

그리고 전체 조회와 id를 통한 조회를 하는 코드를 생성한다.

이렇게 하면 CRUD 중 R작업을 하는 Controller가 완성이 된다.

```
    @PostMapping(ItemConstants.ITEM_END_POINT_V1)
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<Item> createItem(@RequestBody Item item) {
        return itemReactiveRepository.save(item);
    }
```
위와 같은 코드를 추가하여 Item을 생성하는 메소드를 추가한다.

```
    @DeleteMapping(ItemConstants.ITEM_END_POINT_V1 + "/{id}")
    public Mono<Void> deleteItem(@PathVariable String id) {
        return itemReactiveRepository.deleteById(id);
    }
```
id를 받아 delete 하는 메소드 또한 추가한다.

```
    @PutMapping(ItemConstants.ITEM_END_POINT_V1 + "/{id}")
    public Mono<ResponseEntity<Item>> updateItem(@PathVariable String id, @RequestBody Item item) {
        return itemReactiveRepository.findById(id)
                .flatMap(currentItem -> {
                    currentItem.setPrice(item.getPrice());
                    currentItem.setDescription(item.getDescription());
                    return itemReactiveRepository.save(currentItem);
                })
                .map(updatedItem -> new ResponseEntity<>(updatedItem, HttpStatus.OK))
                .defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }
```
id와 request body를 받아 update 하는 메소드 또한 추가한다. id가 잘못들어왔을 경우 notfound status 또한 반환한다.


### Test 코드
해당 컨트롤러가 잘 수행이 되는지 테스트 코드를 작성한다.

```
@SpringBootTest
@RunWith(SpringRunner.class)
@DirtiesContext
@AutoConfigureWebTestClient
@ActiveProfiles("test")
public class ItemControllerTest {
}
```
위와 같이 어노테이션을 달고있는 클래스를 선언한다.

```
    @Autowired
    WebTestClient webTestClient;

    @Autowired
    ItemReactiveRepository itemReactiveRepository;
```

그리고 의존성 주입을 해준다. test를 수행할때 constructor injection을 하여 의존성 주입을 할 경우, 정상동작이 안되기 때문에 일단 어노테이션으로 주입을 해준다.

```
    public List<Item> data() {
        return Arrays.asList(new Item(null, "SAMSUNG TV", 400.0), new Item(null, "LG TV", 300.0), new Item(null, "APPLE TV", 200.0), new Item("1", "TV", 100.0));
    }
```
조회를 테스트하기위한 더미데이터를 생성하는 메소드를 작성한다.

```
    @Before
    public void setUp() {
        itemReactiveRepository.deleteAll()
                .thenMany(Flux.fromIterable(data()))
                .flatMap(itemReactiveRepository::save)
                .doOnNext(item -> {
                    System.out.println(item);
                })
                .blockLast();
    }
```
그리고 모든 테스트 메소드 수행전에 데이터 초기화하는 메소드를 정의하고 @Before 메소드를 통해 수행하게 끔한다.

```
    @Test
    public void getAllItems() {
        webTestClient.get().uri(ItemConstants.ITEM_END_POINT_V1)
                .exchange()
                .expectStatus().isOk()
                .expectHeader().contentType(MediaType.APPLICATION_JSON_UTF8)
                .expectBodyList(Item.class)
                .hasSize(data().size());
    }
```
우선 모든 item을 조회하는 메소드를 테스트하는 메소드를 작성한다. 정상 동작일 경우 http status는 ok가 떨어지고 body의 각각의 타입은 item 클래스 일것이며 사이즈 또한 동일 할 것이다.

```
    @Test
    public void getAllItems_approach2() {
        webTestClient.get().uri(ItemConstants.ITEM_END_POINT_V1)
                .exchange()
                .expectStatus().isOk()
                .expectHeader().contentType(MediaType.APPLICATION_JSON_UTF8)
                .expectBodyList(Item.class)
                .hasSize(data().size())
        .consumeWith((response) -> {
            List<Item> items = response.getResponseBody();
            items.forEach((item) -> {
                assertTrue(item.getId() != null);
            });
        });
    }
```
또한 데이터가 올바르게 저장되었을 경우, mongodb 자체에서 id를 발급해주기 때문에 response stream을 consume 해서 각 item들의 id가 null 이 있는지 체크를 해준다.

```
    @Test
    public void getAllItems_approach3() {
        Flux<Item> itemFlux = webTestClient.get().uri(ItemConstants.ITEM_END_POINT_V1)
                .exchange()
                .expectStatus().isOk()
                .expectHeader().contentType(MediaType.APPLICATION_JSON_UTF8)
                .returnResult(Item.class)
                .getResponseBody();

        StepVerifier.create(itemFlux)
                .expectNextCount(data().size())
                .verifyComplete();
    }
```
response의 BODY를 직접 받아서 테스트하는 메소드도 추가한다.

```
    @Test
    public void getOneItem() {
        webTestClient.get().uri(ItemConstants.ITEM_END_POINT_V1.concat("/{id}"), "1")
        .exchange()
        .expectStatus().isOk()
        .expectBody()
        .jsonPath("$.price", 100.00);
    }
```
단일 조회를 했을 때 해당 RESPONSE가 올바른 것을 반환했는지 확인하기 위한 메소드도 추가한다. jsonPath 메소드에서 $.속성명 을 정의해서 해당 속성에 올바른 값이 조회가 되는지 확인을 한다.

```
    @Test
    public void getOneItem_notfound() {
        webTestClient.get().uri(ItemConstants.ITEM_END_POINT_V1.concat("/{id}"), "2")
                .exchange()
                .expectStatus().isNotFound();
    }
```
없는 id를 조회하여 없을 경우 notfound를 잘 반환하는지 테스트하는 메소드 또한 추가한다.

```
@Test
    public void createItem() {
        Item item = new Item(null, "Iphone x", 999.99);
        webTestClient.post().uri(ItemConstants.ITEM_END_POINT_V1)
            .contentType(MediaType.APPLICATION_JSON_UTF8)
        .body(Mono.just(item), Item.class)
        .exchange()
        .expectStatus().isCreated()
        .expectBody()
        .jsonPath("$.id").isNotEmpty()
        .jsonPath("$.description").isEqualTo("Iphone x");
    }
```
위의 메소드를 추가하여, 지정한 데이터를 가진 item을 추가하는지 테스트 한다.

```
    @Test
    public void deleteItem() {
        webTestClient.delete().uri(ItemConstants.ITEM_END_POINT_V1.concat("/{id}"), "1")
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .exchange()
                .expectStatus().isOk()
                .expectBody(Void.class);
    }
```
지정한 id를 삭제하는 메소드 또한 테스트 해본다.

```
@Test
    public void updateItem() {
        String desc = "UPDATE TV";
        double price = 150.0;
        Item item = new Item(null, desc, price);
        webTestClient.put().uri(ItemConstants.ITEM_END_POINT_V1.concat("/{id}"), "1")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .body(Mono.just(item), Item.class)
                .exchange()
                .expectStatus().isOk()
                .expectBody()
                .jsonPath("$.price").isEqualTo(price)
                .jsonPath("$.description").isEqualTo(desc);
    }

    @Test
    public void updateItem_invalidId() {
        String desc = "UPDATE TV";
        double price = 150.0;
        String id = "notfound";
        Item item = new Item(null, desc, price);
        webTestClient.put().uri(ItemConstants.ITEM_END_POINT_V1.concat("/{id}"), id)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .body(Mono.just(item), Item.class)
                .exchange()
                .expectStatus().isNotFound();
    }
```
위와 같이 update를 하거나, 없는 id를 지정하여 업데이트 시도하는 테스트 메소드를 추가한다.

이렇게 CRUD작업을 하는 Controller 구현을 완료해본다.

참고
- build-reactive-restful-apis-using-spring-boot-webflux 강좌
