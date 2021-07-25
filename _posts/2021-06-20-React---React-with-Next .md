---
date: 2021-06-20 07:00:39
layout: post
title: "React-React with Next"
subtitle:
description:
image:
optimized_image:
category:
tags:
- react
- next
author: newvelop
paginate: false
---
React를 시작하면서 React란 무엇이고, React와 같이 사용할 Next.js는 무엇인지 개념에 대해 검토한다.

### @RestController를 이용하여 만들기
#### MONGO DB연결
DB를 연결하고 CRUD 작업을 하는 REST API를 구현해야한다면, 우선 DB를 연결할때 사용할 클래스가 필요하다. 이 포스팅은 MONGO DB를 이용할 것이기 때문에 먼저 의존성을 설정해준다.

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
