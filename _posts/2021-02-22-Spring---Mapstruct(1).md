---
date: 2021-02-22 07:00:39
layout: post
title: "Spring-Mapstruct(1)"
subtitle:
description:
image:
optimized_image:
category:
tags:
- spring boot
- jpa
- mapping
author: newvelop
paginate: false
---
modelMapper보다 좀 더 사용하기 좋다고 판단한 Mapstruct의 사용법을 정리하고자 한다.

### 설정
메이븐에 의존성 설정을 해준다.
```
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>1.4.2.Final</version>
    </dependency>
```

그리고 컴파일 시점에 매핑하는 코드를 컴파일해서 생성하기 때문에 메이븐 컴파일러 플러그인이 추가되어있지 않으면 추가를 하고, 어노테이션 프로세서 설정을 해준다.
```
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.6.2</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
            <annotationProcessorPaths>
                <path>
                    <groupId>org.projectlombok</groupId>
                    <artifactId>lombok</artifactId>
                    <version>1.18.16</version>
                </path>
                <!-- This is needed when using Lombok 1.18.16 and above -->
                <path>
                    <groupId>org.projectlombok</groupId>
                    <artifactId>lombok-mapstruct-binding</artifactId>
                    <version>0.2.0</version>
                </path>
                <!-- Mapstruct should follow the lombok path(s) -->
                <path>
                    <groupId>org.mapstruct</groupId>
                    <artifactId>mapstruct-processor</artifactId>
                    <version>1.4.2.Final</version>
                </path>
            </annotationProcessorPaths>
        </configuration>
    </plugin>
```

* 위의 코드를 보면 mapstruct뿐 아니라 lombok관련 설정이 들어있음을 확인할 수 있다. lombok을 사용하지 않을 경우는 상관없으나, lombok을 프로젝트에서 사용할 경우, mapstruct에서 get/set에 컴파일 하기 전에 접근할 경우, symbol not found가 뜨기 때문에 mapstruct가 컴파일 되기전 lombok이 수행될 수 있게 설정해야한다.


### 일반적인 코드
ModelMapper와 마찬가지로 자바에서 객체 간 매핑에 대한 코드를 자동으로 생성해주는 매핑 라이브러리이다. 다른 점이라면, Annotation Processor를 사용하여 런타임이 아니라 컴파일 시, 매핑코드를 생성해준다.

일반적인 Dto와 Entity를 예시로 삼아서 매핑하는 방법을 소개한다.

User.java
```
@Data
@Entity(name =  "I_USER")
@EntityListeners(RefreshHelper.class)
@Accessors(chain = true)
public class User {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    private Long userUid;

    @Column(unique = true, length = 16, updatable = false, nullable = false)
    private String userId;

    @Column(length = 16, nullable = false)
    private String userName;

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Column(nullable = false, length = 256)
    private String password;

    @ManyToMany(fetch = FetchType.LAZY, cascade = { CascadeType.PERSIST, CascadeType.MERGE })
    @JoinTable(name = "user_user_group", joinColumns = @JoinColumn(name="user_uid"), inverseJoinColumns = @JoinColumn(name="user_group_id"))
    @JsonIgnoreProperties({"users"})
    private Set<UserGroup> userGroups;
}
```

UserDto.java
```
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
@Accessors(chain = true)
public class UserDto extends FailMessageDto{
    private Long userUid;
    private String userId;
    private String userName;
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    private String password;

    private Set<Long> userGroupIds;
}
```

위와 같은 예시코드가 있으며 UserGroup이란 클래스와 다대다 관계를 형성하고 있다고 해보자.
이를 서로 매핑하려면 메소드를 하나 만들어주고, 속성별로 복사하는 코드를 작성해야한다.
Mapstruct를 사용할 경우, Mapper 인터페이스를 생성하고, 객체 매핑하는 규칙을 작성하면 이에 걸맞는 매핑 코드를 자동으로 생성해준다.

UserMapper.java 선언부
```
@Mapper(imports = {//여기서 사용할 클래스들을 작성}, nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE  //null 무시할 정책 사용할 경우)
public interface UserMapper {
}
```

#### 생성
UserDto을 기반으로 User 엔티티를 생성하고, 이를 DB에 저장하고 싶을 경우 작성하는 코드이다.
```
@Mappings({
            @Mapping(target = "userGroups", expression = "java(userDto.getUserGroupIds() == null ? null : userDto.getUserGroupIds().stream().map(v -> new UserGroup(v)).collect(Collectors.toSet()))")
    })
    User userDtoToUser(UserDto userDto, @Context CycleAvoidingMappingContext context);
```
context는 차후 설명하도록 하고, mapping rule을 좀 살펴보면 dto와 entity간의 필드명이 같고 타입 또한 같을 경우 알아서 복사를 해주기 때문에 Mapping 규칙을 따로 지정해주지 않는다. 다만 UserGroupIds 속성과 UserGroups를 살펴보면 Class의 Set과 int의 Set으로 다르다. 이럴 경우 본인은 expression을 사용하여 Java 코드로 매핑을 해결하였다.

User에서 UserDto를 만드는 방법은 위의 코드를 반대로 작성하면 된다.

#### 수정
JPA를 이용하여 수정을 해야되는 경우, 있는지 조회를 하고 값을 수정해서 SAVE 하는 로직을 수행한다.
따라서 조회를 하면 User 엔티티가 생성되는데 이때 User 엔티티의 id값은 복사할 필요가 없고 나머지 값들만 복사하면 된다. 이에 대한 규칙을 작성해보면 
```
@Mappings({
            @Mapping(target = "userId", ignore = true)
    })
    void updateUserFromUserDto(UserDto userDto, @MappingTarget User user, @Context CycleAvoidingMappingContext context);
```
위와 같이 코드를 작성할 수 있게된다. 물론 Null 값이 dto에 있을 경우 null 값이 db에 작성되지 않느냐고 할 수 있는데 이미 UserMapper 선언부에 NULL IGNORE 정책을 사용한다고 명시를 해놨기 때문에 문제가 없다.

### Cycle 해결
JPA 엔티티를 DTO로 변경할 때 유의점은 N:1 혹은 N:M 관계를 지니고 있으며 이 관계가 양방향 참조일 경우이다. User로 예시를 들면 User의 UserGroups 필드가 있는데 이 UserGroups의 멤버를 하나씩 살펴보면 UserGroup 객체이며, 이 관계는 본인이 양방향으로 설정해놨기 때문에 Users 필드가 안에 존재한다. 따라서 서로 상호 참조하는 Cycle이 발생하게 되는데, Entity를 DTO로 바꿀시 상호 참조하는 코드를 작성하게 되어 결국 stackOverflow 에러가 발생하게 된다.

이를 해결하는 해결법으로는 하위 필드를 ignore하는 방법이 첫째로 존재한다. 하지만 이 방법 말고 다른 방법을 찾아본 결과 IdentityHashMap을 사용하여 source를 map으로 작성하고 이를 mapping에 사용하면 cycle이 발생하지 않는다고 한다.

상세한 이유는 차후 살펴보며 기술하겠다.

단순히 해결법만 찾아보면

```
/**
 * A type to be used as {@link Context} parameter to track cycles in graphs.
 * <p>
 * Depending on the actual use case, the two methods below could also be changed to only accept certain argument types,
 * e.g. base classes of graph nodes, avoiding the need to capture any other objects that wouldn't necessarily result in
 * cycles.
 *
 * @author Andreas Gudian
 */
public class CycleAvoidingMappingContext {
    private Map<Object, Object> knownInstances = new IdentityHashMap<Object, Object>();

    @BeforeMapping
    public <T> T getMappedInstance(Object source, @TargetType Class<T> targetType) {
        return (T) knownInstances.get( source );
    }

    @BeforeMapping
    public void storeMappedInstance(Object source, @MappingTarget Object target) {
        knownInstances.put( source, target );
    }
}
```
위의 코드를 기반으로 context 클래스를 생성한후, Mapper에 @context 인자로 넘겨주면 cycle이 해결이 된다.