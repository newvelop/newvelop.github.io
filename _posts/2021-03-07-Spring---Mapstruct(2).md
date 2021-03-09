---
date: 2021-02-22 07:00:39
layout: post
title: "Spring-Mapstruct(2)"
subtitle:
description:
image:
optimized_image:
category:
tags:
- spring boot
- jpa
- mapstruct
author: newvelop
paginate: false
---
첫 번째 포스팅에서 다뤘던 Mapstruct에서의 cycle 해결관련하여 해당 방법이 해결하는 이유를 탐색한다.

### 코드 해석
#### Cycle 해결 코드
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

위의 코드를 살펴보면 source에서 dest로 매핑을 하기위한 작업 전에 사전작업으로 추가한 코드이며, source를 키로 하여 mapping 할 target을 value로 저장하며, source를 키로 하여 map에서 객체를 가져오는 메소드가 구현되어있음을 확인할 수 있다.

#### 예시 DTO와 ENTITY
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

FailMessageDto.java
```
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
public class FailMessageDto {
    protected String message;
}
```
DTO와 ENTITY의 예시는 상단과 같다.

#### Mapper를 기반으로 생성된 코드 및 해석
Mapper interface를 기반으로 생성된 mapping 코드를 살펴본다.

UserMapperImple.java
```
 @Override
    public UserDto userToUserDto(User user, CycleAvoidingMappingContext context) {
        UserDto target = context.getMappedInstance( user, UserDto.class );
        if ( target != null ) {
            return target;
        }

        if ( user == null ) {
            return null;
        }

        UserDto userDto = new UserDto();

        context.storeMappedInstance( user, userDto );

        userDto.setUserGroups( userGroupSetToUserGroupDtoSet( user.getUserGroups(), context ) );
        userDto.setUserUid( user.getUserUid() );
        userDto.setUserId( user.getUserId() );
        userDto.setUserName( user.getUserName() );
        userDto.setPassword( user.getPassword() );
        userDto.setEmail( user.getEmail() );
        userDto.setContact( user.getContact() );
        userDto.setUserLang( user.getUserLang() );
        userDto.setUserStatusCode( user.getUserStatusCode() );
        userDto.setUpdateTime( user.getUpdateTime() );
        userDto.setInsertTime( user.getInsertTime() );

        return userDto;
    }

    protected Set<UserGroupDto> userGroupSetToUserGroupDtoSet(Set<UserGroup> set, CycleAvoidingMappingContext context) {
        Set<UserGroupDto> target = context.getMappedInstance( set, Set.class );
        if ( target != null ) {
            return target;
        }

        if ( set == null ) {
            return null;
        }

        Set<UserGroupDto> set1 = new HashSet<UserGroupDto>( Math.max( (int) ( set.size() / .75f ) + 1, 16 ) );
        context.storeMappedInstance( set, set1 );

        for ( UserGroup userGroup : set ) {
            set1.add( userGroupToUserGroupDto( userGroup, context ) );
        }

        return set1;
    }
```

위와 같이 코드가 생성이 되는데 userToUserDto메소드에선 User 엔티티가 들어오면 먼저 map을 조회한다. 해당 key를 가진 객체가 존재할 경우 객체를 반환하고, 없을 경우 현재 user 엔티티를 map에 저장하고 mapping 로직을 거친다. UserGroup 엔티티를 매핑을 하기 위해서 다시 userGroupSetToUserGroupDtoSet을 호출하는데 이때도 키를 기반으로 map에 데이터가 있는지 확인을 한다. 처음 userGroup의 경우, 데이터가 없기 때문에 다시 mapping 로직을 거치는데 이때 User와의 양방향 관계를 형성하고 있기 때문에 userToUserDto메소드를 호출한다.

두번째로 userToUserDto가 호출되었을때, 처음 호출했을시 map에 user 엔티티를 저장했기 때문에 조회가 되고, 매핑로직을 거치지 않고 반환이 되면서 스택오버플로우가 발생하지 않고 해결이 된다.


#### IdentityHashMap 사용 이유
여기서 map을 이용할때 왜 일반 HashMap이 아니라 IdentityHashMap를 사용하는지 궁금할 것인데, 이는 해당 객체의 특성으로 인해 달라진다. HashMap의 경우, 키를 통해 값을 가져올 때 키의 동일함을 비교하기위해, equals 혹은 hashcode 메소드를 이용하여 비교를 한다. 근데 문제점은 N:1 혹은 N:M 관계를 맺고있는 양방향 엔티티의 경우 속성을 계속 참조하다보면 순환 참조가 발생하게된다. 해당 부분은 1편을 참고하길 바란다. 
반면 IdentityHashMap은 equals나 hashcode 메소드를 사용하는게 아닌 '=='연산자를 이용하여 비교하기 때문에 단순 객체의 주소를 비교하여 key 의 동일성을 판단한다. 따라서 속성 검사를 하지 않기 때문에 순환 참조를 회피하고 매핑을 완료할 수 있게 된다.


참고
- https://javarevisited.blogspot.com/2013/01/difference-between-identityhashmap-and-hashmap-java.html#axzz6oZRiYGMJ