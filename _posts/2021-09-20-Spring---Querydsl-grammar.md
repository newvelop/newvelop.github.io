---
date: 2021-09-20 07:00:39
layout: post
title: "Spring-Querydsl grammar"
subtitle:
description:
image:
optimized_image:
category:
tags:
- Spring
- Querydsl
- grammar
author: newvelop
paginate: false
---
실전 querydsl이라는 강의를 듣고 요약한 내용을 정리한다.

### projection
반환 타입을 의미한다.

기본적으로 select 대상이 하나 이면 해당 대상의 타입으로 반환이 되는 것이고, 여러개를 반환할 경우 Tuple로 반환한다. Tuple로 반환될 경우, tuple.get(필드명)으로 호출해서 필드값 조회를 할 수 있다.

```
List<MemberDto> resultList = em.createQuery("select new spring.jpa.querydsl.dto.MemberDto(m.userName, m.age) from Member m", MemberDto.class)
                .getResultList();
```
만약 jpql을 이용하여 조회한 값을 dto에 저장하려면 클래스의 풀패스를 적은뒤 생성자를 호출해서 dto 객체를 새로 생성하는 방법을 취해야하는데 이는 좀 번거롭다.

querydsl에선 이를 편한 방법으로 지원을 한다.

우선 setter를 이용한 projection 방법을 지원한다.

```
List<MemberDto> fetch = queryFactory
  .select(Projections.bean(MemberDto.class, member.userName, member.age))
  .from(member)
  .fetch();
```
위의 코드처럼 Projections.bean을 호출하여 해당 dto의 setter에 넣을 값을 선언하고, 이를 setter를 이용하여 필드에 값을 넣어준다. 여기서 한가지 유의할 점은 기본생성자가 없을 경우 에러를 띄우는데 setter 주입법이 기본생성자를 이용하여 객체를 생성하고, setter를 이용하여 필드에 값을 주입하기 때문이다. 또 하나 주의할점은 어떤 setter가 어떤 값을 가져갈지 명시를 하지 않았다. 즉 필드가 같았을때 setter를 이용하여 값을 주입하는 것이기 때문에 필드명이 일치하지 않을 경우 값을 넣을 수 없다.

```
List<MemberDto> fetch = queryFactory
                .select(Projections.fields(MemberDto.class, member.userName, member.age))
                .from(member)
                .fetch();
```
필드에 직접 값을 넣어주는 방법도 있는데, 이는 setter를 이용하지 않고 값을 넣어주는 방법이다. 이도 마찬가지로 필드명이 같아야 한다.

```
List<MemberDto> fetch = queryFactory
                .select(Projections.constructor(MemberDto.class, member.userName, member.age))
                .from(member)
                .fetch();
```
생성자를 통해 값을 주입할 수도 있다. 이는 생성자의 변수 순서와 같게 해주면 해당 생성자를 이용하여 값을 넣을 수 있다.

```
List<UserDto> fetch = queryFactory
                .select(Projections.bean(UserDto.class, member.userName.as("name"), member.age))
                .from(member)
                .fetch();
```
만약 필드명이 다르지만 값을 넣어주고 싶을 경우 as를 이용해 넣어줄 필드명으로 바꿔서 동작하게 하면 값이 들어간다.

```
QMember sub = new QMember("sub");
        List<UserDto> fetch = queryFactory
                .select(Projections.bean(UserDto.class, member.userName.as("name")
                        , member.age, ExpressionUtils.as(JPAExpressions
                        .select(sub.age.max()).from(sub), "maxAge")))
                .from(member)
                .fetch();
```
만약에 subquery수행 값 같은 경우는 이름이 정해지지 않은 상태이다. 이를 dto에 넣어주고 싶을 경우 ExpressionUtils.as를 이용해서 subquery의 aliasing을 하면 setter주입으로 넣어줄 수있다.

### @QueryProjection
@QueryProjection을 통해 dto 주입을 하는 방법도 있다.
```
@QueryProjection
public MemberDto(String userName, int age) {
    this.userName = userName;
    this.age = age;
}
```
먼저 MemberDto의 생성자에 @QueryProjection 어노테이션을 넣어주고, compileQuerydsl을 하면 memberdto의 q클래스가 생성이 된다.

```
List<MemberDto> fetch = queryFactory
                .select(new QMemberDto(member.userName, member.age))
                .from(member)
                .fetch();
```
그리고 이렇게 new 연산자를 이용하여 사용하면 된다. 단순 projection에 생성자를 이용하는 것과 다른 점은, 단순 생성자 이유는 올바르지 않은 생성자를 호출한다고 해도 컴파일 시점에선 에러를 잡을 수가 없다. 반면 @QueryProjection을 이용하면 잘못된 생성자를 이용할 경우 컴파일 시점에 잡을 수 있다.


### 동적 쿼리
where절에 사용할 조건 파라미터에 따라서 쿼리를 제작을 해야할 필요가 있다. 이 방식엔 두가지가 있는데 먼저 BooleanBuilder를 이용한 방식이다.

```
BooleanBuilder builder = new BooleanBuilder();
if (userNameParam != null) {
    builder.and(member.userName.eq(userNameParam));
}

if (ageParam != null) {
    builder.and(member.age.eq(ageParam));
}

return queryFactory
        .selectFrom(member)
        .where(builder)
        .fetch();
```
위와 같이 null일 경우 조건에 넣지않고, null이 아닐 경우 조건으로 만들어주는 방식을 통해 동적 쿼리를 제작할 수 있다.


```
private List<Member> searchMember2(String userNameParam, Integer ageParam) {
    return queryFactory
            .selectFrom(member)
            .where(userNameEq(userNameParam), ageEq(ageParam))
            .fetch();
}

private BooleanExpression userNameEq(String userNameParam) {
    if (userNameParam == null) {
        return null;
    } else {
        return member.userName.eq(userNameParam);
    }
}

private BooleanExpression ageEq(Integer ageParam) {
    if (ageParam == null) {
        return null;
    } else {
        return member.age.eq(ageParam);
    }
}
```
where절에 동적쿼리를 직접 제작할 수 있는데, 위와 같은식으로 BooleanExpression를 넘겨주는 메소드를 제작하고 where절에 ,를 기준으로 메소드를 여러개 호출하면 된다. BooleanExpression가 null일 경우 where에서 자동으로 무시하며, BooleanExpression가 들어오면 조건을 적용하여 query를 생성한다.

### 수정, 삭제 벌크 연산
```
long count = queryFactory
                .update(member)
                .set(member.userName, "비회원")
                .where(member.age.lt(28))
                .execute();
```
위와 같이 update 를 실행하면 조건에 맞는 행들의 데이터를 한번에 바꿔준다. 다만 이렇게 하면 문제점이 영속성컨텍스트를 거치지 않고 db에 바로 쿼리를 날리기 때문에, 영속성 컨텍스트와 db의 불일치가 발생하게된다. 그리고 이 트랜잭션 안에서 select를 한번 더하면 db에서 조회한 값과 영속성컨텍스트의 값이 다르게 되는데, 이때 영속성컨텍스트가 우선권을 가지고 와서 db의 값을 무시하게 된다. 이러한 현상을 해결하기 위해선 EntityManager의 flush와 clear를 수행해서 bulk 연산 이후에 영속성을 날려버리는게 좋다.


### SQL function 호출하기
```
List<String> fetch = queryFactory
  .select(Expressions.stringTemplate("function('replace', {0}, {1}, {2})"
          , member.userName, "m", "Member"))
  .from(member)
  .fetch();
```
이런 식으로 db의 함수를 호출 할 수 있다. 위의 코드 기대값은 userName의 m을 Member로 교체해서 보여주는 것이다. function이 있는지 확인하고 싶을 경우, 현재 사용하는 DB의 DIALECT를 찾아보면 된다.

### querydsl repository
JPA로 구현했던 REPOSITORY를 QUERYDSL로 만들 수 있다.

```
public List<Member> findAll_Querydsl() {
    return queryFactory
            .selectFrom(member)
            .fetch();
}

public List<Member> findByUserName_querydsl(String userName) {
    return queryFactory.selectFrom(member)
            .where(
                    member.userName.eq(userName)
            )
            .fetch();
}
```

위와 같이 find 메소드를 querydsl로 구현할 수 있다.

```
BooleanBuilder builder = new BooleanBuilder();
if (hasText(condition.getUserName())) {
    builder.and(member.userName.eq(condition.getUserName()));
}

...

return queryFactory
        .select(new QMemberTeamDto(
                member.id.as("memberId"),
                member.userName,
                member.age,
                team.id.as("teamId"),
                team.name.as("teamName")
        ))
        .from(member)
        .where(builder)
        .leftJoin(member.team, team)
        .fetch();
```
위와 같은 식으로 조건이 여러개가 될 수 있을 경우, BooleanBuilder를 통해 동적쿼리를 생성할 수 있다.

```
.where(
      userNameEq(condition.getUserName()),
      teamNameEq(condition.getTeamName()),
      ageGoe(condition.getAgeGoe()),
      ageLoe(condition.getAgeLoe())
)
```
where절에 직접 booleanexpression을 반환하는 메소드를 여러개 호출해서 동작하게 할 수 도 있다.


참고
- Querydsl 실전 강의