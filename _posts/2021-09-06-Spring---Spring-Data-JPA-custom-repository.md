---
date: 2021-09-06 07:00:39
layout: post
title: "Spring-Spring Data JPA custom repository"
subtitle:
description:
image:
optimized_image:
category:
tags:
- Spring
- Spring Data JPA
- 실전! 스프링 데이터 JPA
- custom repository
author: newvelop
paginate: false
---
실전 스프링 데이터 JPA라는 강의를 들으면서 요약한 내용을  정리한다.

### 사용자 정의 리포지토리
Spring Data JPA에서 제공해주는, 컨벤션에 맞춘 메소드명을 지닌 메소드 정의를 통해 자동으로 구현되는 메소드를 사용하는 것 말고, 커스텀 메소드를 사용하고 싶을 수 있다. 하지만 커스텀 메소드를 사용하면서 클래스를 전부 처음부터 구현하려면 제공하는 편리한 기능들을 버리고 밑바닥부터 사람이 새로 구현해야하며 이는 불편하다.

따라서 커스텀의 기능도 넣고, 제공해주는 기능도 이용할 수 있는 방법을 알아본다.

```
public interface MemberRepositoryCustom {
    List<Member> findMemberCustom();
}
```

먼저 위와 같이 인터페이스를 정의하고,

```
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m")
                .getResultList();
    }
}
```

위와 같이 구현한 클래스를 정의한다.

```
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
...
}
```
그리고 위와 같이 인터페이스를 상속하면 사용자가 구현한 커스텀 메소드와 다른 메소드를 혼용해서 사용할 수 있다. 주의할 점은, 본인이 상속시키고자 하는 MemberRepository인터페이스 뒤에 impl을 붙여야한다는 것이다. 만약 이게 싫다면 @EnableJpaRepositories의 repositoryImplementationPosfix에서 바꿔주면 된다. 하지만 어지간하면 이런식으로 사용하는게 좋다.

이런 식으로 클래스 구현후 상속해서 이어붙여도 되지만, 클래스가 너무 복잡해질 경우, Repository클래스를 따로 구현하고, 상속시키지 않고 @Repository 어노테이션을 달아서 빈등록하고 직접 주입해서 사용해도 된다.

### Auditing
엔티티를 생성, 변경할때 변경한 사람과 시간을 추적하고 싶을 경우에 사용하는 기능이다.

```
@MappedSuperclass
public class JpaBaseEntity {
    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;

    @PrePersist
    public void prePersist() {
        createdDate = LocalDateTime.now();
        updatedDate = LocalDateTime.now();
    }

    @PreUpdate
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
}
```
위와 같이 클래스를 정의해서 persist 전에 시간 업데이트하고, update 전에 시간업데이트하는 메소드를 정의하면 해당 동작시에 메소드가 수행이 되어서 컬럼에 값을 업데이트한다. 그리고 @MappedSuperclass을 사용을 하면 이 클래스를 상속하는 엔티티가 실제 상속관계가 아니라 컬럼을 그대로 테이블에 가져다 쓰는 형태로 ddl을 제작한다. 따라서 Member 엔티티가 해당 클래스를 상속하면 createdDate와 updatedDate 컬럼이 생성된다.

위의 방식은 Spring Data JPA의 기능을 이용하지 않은 방식이고 기능을 이용하여 구현하는 방법이 있다.

우선 Spring Boot의 메인 클래스에 @EnableJpaAuditing 어노테이션을 작성한다.

```
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Data
public class BaseEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}
```
그리고 위와 같이 작성해서 Entity에 상속시키면 된다. AuditingEntityListener가 entity audit 이벤트를 캐리하는 역할을 하고 @CreateDate와 @LastModifiedDate가 Persist와 Update이벤트에 대한 처리를 해준다.

만약 생성자와 수정자도 알고싶으면,
```
@CreatedBy
@Column(updatable = false)
private String createdBy;

@LastModifiedBy
private String lastModifiedBy;
```
해당 필드를 엔티티에 추가하고, 

```
@Bean
public AuditorAware<String> auditorProvider() {
    return () -> ...);
}
```
AuditorAware을 빈으로 등록하고, session정보를 읽어와서 사용자 정보를 반환하면 된다.

Audit 이벤트를 캐치하려면 @EntityListeners(AuditingEntityListener.class)를 매번 등록해야 하는데 이게 귀찮을 경우, META-INF/orm.xml에 이벤트 리스너를 등록하면 된다고 한다.

사실 컬럼에 그대로 넣어서 정의해도 되니 이건 상황에 맞게 사용하는 걸로 알고 있으면 될듯 하다.

### 도메인 클래스 컨버터 & 페이징
```
@GetMapping("/members2/{id}")
public String findMember2(@PathVariable("id") Member member) {
    return member.getUserName();
}
```
위와 같이 컨트롤러 코드를 작성하면, 해당 id를 지닌 member 엔티티 값을 조회한다. 다만 해당 member의 값을 수정해도 바뀌지 않는데 이는 트랜잭션 범위 밖에서 값을 조회했기 때문에 영속성에 들어있지 않기 때문이다.

또한 Spring Data JPA를 이용할때, 컨트롤러에서 RequestBody를 쉽게 만들수 있다.

```
@GetMapping("/members")
public Page<Member> list(@PageableDefault(size= 5) Pageable pageable) {
    return memberRepository.findAll(pageable);
}
```
이런 식으로 Pageable 타입을 파라미터로 받는 컨트롤러 메소드를 정의하면, Spring Data JPA에서 해당 인터페이스의 구현체인 PageRequest타입의 객체로 RequestBody를 제작하여 사용한다. 또한 @PageableDefault어노테이션을 사용하면, null일 경우의 default 값을 필드에 설정해서 사용할 수 있다. 만약 paging을 해야하는 주체가 두개라면, url에 넘겨줄 때는 '접두사명_프로퍼티'(ex) member_size)로 넘겨주고, controller에서는 @Qualifier("접두사") 로 구분을 한다.

```
@GetMapping("/members/dto")
public Page<MemberDto> listDto(@PageableDefault(size= 5) Pageable pageable) {
    Page<Member> memberPage = memberRepository.findAll(pageable);
    Page<MemberDto> memberDtoPage = memberPage.map(m -> new MemberDto(m.getId(), m.getUserName(), m.getTeam().getName()));
    return memberDtoPage;
}
```
만약 dto로 변환을 해서 반환하고 싶을 경우, 위와 같이 변환해서 반환하면 되며, dto에 entity를 파라미터로 하는 생성자를 새로 정의해서 map을 해도 된다.



참고
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84