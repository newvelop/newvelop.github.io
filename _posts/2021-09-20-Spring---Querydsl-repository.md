---
date: 2021-09-20 08:00:39
layout: post
title: "Spring-Querydsl repository"
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

### repository
spring data jpa로 repository 변경하고 여기에 querydsl을 적용해본다.

기본적으로 spring data jpa에선 JpaRepository 인터페이스를 상속하는 인터페이스를 정의하면 레포지토리를 사용할 수 있다. 여기에 spring data jpa로 처리하기 번거로운 쿼리들을 querydsl로 구현한 메소드들을 사용할 수 있게 한다.

```
public interface MemberRepositoryCustom {
    List<MemberTeamDto> search(MemberSearchCondition condition);
}
```

그러기 위해선 위처럼 커스텀 쿼리를 작성할 메소드를 인터페이스로 정의한다.

```
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom
```
그리고 JpaRepository와 함께 custom 인터페이스를 상속한다.

```
public class MemberRepositoryImpl implements MemberRepositoryCustom
```
마지막으로 커스텀 인터페이스의 메소드를 구현한 구현체 클래스를 만들면 된다. 주의할 점은 레포지토리명을 spring data jpa 레포지토리 + impl로 지정을 해줘야 인식을 한다는 점이다.

### 페이징
```
QueryResults<MemberTeamDto> memberTeamDtoQueryResults = queryFactory
                ...
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetchResults();
List<MemberTeamDto> content = memberTeamDtoQueryResults.getResults();
long total = memberTeamDtoQueryResults.getTotal();
return new PageImpl<>(content, pageable, total);
```
querydsl을 통해서 페이징을 할 수 있는 방법은 위와 같다. 일반 쿼리는 동일하게 작성을 하고, offset과 limit에 파라미터를 넘기고 fetchResults를 하면 조회한 정보와 동시에 전체 카운트 수를 가져온다. 이를 PageImpl을 통해서 페이지 객체로 반환하는 것이 첫번째이다.

```
List<MemberTeamDto> memberTeamDtoQueryResults = queryFactory
    ...
.fetch();
```

두번째로는 카운트 쿼리를 querydsl에 생성을 맡기지 않고 최적화하는 방법이다. 위와 같이 fetchResults가 아니라 fetch로 content만 가져온 뒤, 카운트쿼리를 생성해서 fetchCount를 호출해서 데이터 수를 조회하는 쿼리를 실행하면 된다.

```
JPAQuery<Member> countQuery = queryFactory
    ...
```
좀더 최적화하는 방법으로는 위와 같이 최적화하는 쿼리 자체를 생성하고, 

```
return PageableExecutionUtils.getPage(memberTeamDtoQueryResults, pageable, () -> countQuery.fetchCount());

```
위와 같이 PageableExecutionUtils의 getPage에 content와 countquery.fetchCount를 넘겨주면 페이징 쿼리를 실행을 해야할 경우 실행을 하고, 실행할 필요 없을 경우 실행을 하지 않는 최적화를 수행한다.

### spring data에서 제공하는 기능
QuerydslPredicateExecutor를 repository에 상속시키면, find 계열 메소드에 querydsl predicate를 사용할 수 있다. 즉

```
memberRepository.findAll(QMember.member.userName.eq("m1"))
```
이런식으로 사용이 가능하다는 것이다. 다만 문제는 위의 기능은 join이 불가능하다는 것이 한계이다. 또한 querydsl에 종속되는 객체를 만들어서 넘겨야하기 때문에 실무에 사용하기가 어렵다.

또한 QueryDslRepositorySupport라는 추상 클래스가 있는데 이를 상속하면 몇가지 편리한 기능을 사용할 수 있다.

```
QueryResults<MemberTeamDto> memberTeamDtoQueryResults = getQuerydsl().applyPagination(pageable, select).fetchResults();

```
상속해서 사용하면 entitymanager를 가져올 수 있고 위와 같이 pageable을 넘겨주면 페이징을 쉽게 할 수 있다. 하지만 sort가 되지 않는다.


참고
- Querydsl 실전 강의