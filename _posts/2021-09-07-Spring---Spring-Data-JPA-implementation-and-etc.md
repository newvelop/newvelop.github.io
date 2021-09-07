---
date: 2021-09-07 07:00:39
layout: post
title: "Spring-Spring Data JPA implementation and etc"
subtitle:
description:
image:
optimized_image:
category:
tags:
- Spring
- Spring Data JPA
- 실전! 스프링 데이터 JPA
- implementation
author: newvelop
paginate: false
---
실전 스프링 데이터 JPA라는 강의를 들으면서 요약한 내용을  정리한다.

### Spring Data JPA 구현체 분석
Spring Data JPA에서 Repository를 이용하기 위해 상속하는 JpaRepository의 구현체인 SimpleJpaRepository를 살펴본다.

findById나 findAll 등을 확인해보면 이전에 정의했던 JPA를 사용하기 위한 구현했던 메소드랑 비슷하게 EntityManager를 사용하여 구현을 해놓을 것을 확인할 수 있다.

그리고 QueryUtils에 미리 정의해서 사용하는 몇몇 쿼리들이 상수문자열로 정의되어있다. 이 SimpleJpaRepository에는 @Repository이 달려있는데 이 어노테이션은 첫 번째로 컴포넌트 스캔의 대상으로 인식하게 하며, 두 번째로는 JPA의 Exception을 Spring의 Exception으로 변환시켜주는 작업을 하게 마킹한다.

그리고 save 메소드가 존재하는데 이 메소드의 동작은, 새로운 엔티티일 경우 persist를, 기존에 있던 엔티티일 경우 merge를 수행한다. 그리고 엔티티가 새로운지를 판단하게 되는 기준은, 엔티티의 id가 객체일 경우 null이면, primitive 타입이면 0일 경우 새로운 엔티티로 판단한다.

만약 기본적인 이 로직대로 하고싶지 않고 본인이 직접 새로운 엔티티인지 판별을 하고 싶을 경우, Persistable를 상속하고 

```
@Override
public String getId() {
    
}

@Override
public boolean isNew() {
    
}
```
이 두개를 오버라이딩을 하면된다. 강의에선 createdDate를 기반으로 판단을 하는걸 설명하는데 이는 본인이 직접 편한대로 이용을 하면 될듯 하다.
@GeneratedValue 방식을 통해 식별자를 자동생성할 경우, save 시점에 식별자가 없어서 새로운 객체로 판단하기 때문에 정상동작을 한다. 하지만 @Id만 사용해서 값을 직접할당하는 경우, 값이 이미 있기 때문에 merge가 사용되고, 이는 db한번 들러서 조회하고, 없다는 결과를 가져와서 이를 다시 저장하는 것이기 때문에 불필요한 조회가 한번 일어나서 비효율적이다. 따라서 @CreateDate을 기반으로 상속하는 방법을 추천하는 것이니 @GeneratedValue를 사용하지 않을 경우 나중에 한번 이용하면 좋을 듯 하다.

### Query By Example
Example로 엔티티 객체를 생성하면, 해당 객체를 검색조건으로 이용하여 select 쿼리를 만드는 기술이다.
```
Member member = new Member("member1");
ExampleMatcher matcher = ExampleMatcher.matching()
        .withIgnorePaths("age");
Example<Member> example = Example.of(member, matcher);

List<Member> result = memberRepository.findAll(example);
```
이런 식으로 위와 같이 member라는 객체로 example을 만들고 findAll을 하면 조건에 example을 사용하여 맞는 객체를 조회한다. 주의할 점으로, 필드가 객체 타입이면 기본값이 null이기 때문에 무시하는데, primitive 타입일 경우 기본값이 0이나 다른값으로 할당되기 때문에 필드가 무시되지 않고 검색조건으로 사용되어 기대하지 않은 결과를 가져올 수 있다. 따라서 무시하고싶을 경우 ExampleMatcher를 사용하면 된다.

이 방법은 entity를 그대로 사용할 수 있어서 nosql이든 sql이든 상관없이 사용할 수 있다. 다만 조인이 inner join만 되고 left join같은 외부조인이 안되며 매우 단순한 조건만 사용할 수 있기 때문에 실제로 사용하기엔 애매하다.


### Projections
엔티티 대신에 DTO를 편리하게 조회할 때 사용하는 기능이다.

```
public interface UserNameOnly {
    String getUserName();
}
```
이런 식으로 위와 같이 property명과 일치하는 getter를 하나 정의한 인터페이스를 생성하고, 

```
List<UserNameOnly> findProjectionsByUserName(@Param("userName") String userName);
```
반환형을 해당 인터페이스로 하는 메소드를 레포지토리에 정의하면, 해당 필드들을 조회하는 jpql이 생성된다.

```
public interface UserNameOnly {
    String getUserName();
    @Value("#{target.userName + ' ' + target.age}")
    String getAge();
}
```
위와 같은 식으로 spEl을 이용하여 값을 반환할 수 도 있는데, 이렇게 되면 특정 필드를 반환하는 jpql이 생성되는 것이 아니라 전체 조회를 하는 jpql이 실행된 후, 필드의 값을 조합하여 dto로 반환한다.

```
@RequiredArgsConstructor
@Getter
public class UserNameOnlyDto {
    private final String userName;
}
```
위처럼 그냥 dto 정의하고 반환하여 받을 수도 있는데, 이렇게 되면 생성자를 이용하여 값을 필드에 넣어주고 DTO로 반환을 한다.


```
<T> List<T> findProjectionsDtoByUserName(@Param("userName") String userName, Class<T> type);
```
리턴 타입이 고정된게 아니라 dto가 그때 그때 바뀔 경우, 위와 같이 제네릭으로 리턴타입을 정하고, 파라미터로 클래스 타입을 넘기면 넘긴 타입으로 반환한다.

```
public interface NestedClosedProjections {
    String getUserName();
    TeamInfo getTeam();

    interface TeamInfo {
        String getName();
    }
}
```
만약 연관되어있는 엔티티의 필드 정보도 가져와야할 경우 위와 같이 정의를 해주면 된다. 실행되는 쿼리는 Member의 경우, userName만을 가져오는 쿼리를 수행하지만, team정보는 전부 가져오고 그안에서 teamName을 가져와서 필드에 넣는 형태로 실행이 된다. 즉 중첩 엔티티는 최적화가 안된다는 소리이다.

이런 단점이 존재하기 때문에 복잡한 쿼리를 사용해야하는 실무에서 사용하기엔 애매하다.

### native query
```
@Query(value = "select m.member_id as id, m.user_name as userName, t.name as teamName from member m left join team t",
            countQuery = "select count(*) from member",
            nativeQuery = true)
Page<MemberProjection> findByNativeProjection(Pageable pageable);
```
native query를 사용은 할 수 있는데 entity나 dto에 매핑하는게 어렵다. 따라서 이 기능은 projection과 같이 사용한다.

```
public interface MemberProjection {
    Long getId();
    String getUserName();
    String getTeamName();
}
```
이렇게 프로젝션을 정의하고 조회 쿼리를 수행하면 원하는 결과를 얻을 수 있다. 하지만 이 또한 매핑이 애매하기 때문에 사용하기가 어렵다.


참고
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84