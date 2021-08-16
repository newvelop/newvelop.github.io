---
date: 2021-08-16 07:00:39
layout: post
title: "Spring Data JPA-Multi Data Source Setting and Operation principle"
subtitle:
description:
image:
optimized_image:
category:
tags:
- Spring
- JPA
author: newvelop
paginate: false
---
DB에서 NOSQL과 SQL의 차이점, 그리고 인덱스에 대해서 알아본다

### 설정
기본적으로 1개의 데이터 소스를 사용할 경우에는 DataSource가 하나만 존재하기 때문에 application 설정파일에 해당하는 DB 정보를 적으면 알아서 TranscationManager와 EntityFactoryBean이 생성되면서 JPA를 사용할때 EntityManager를 만든다. 하지만 여러개의 데이터 소스를 사용해야 할 때는 직접 설정해서 어떤 Repository가 어떤 데이터 소스를 바라볼지 등을 설정해야한다.  이런 측면에서 설정 단계를 진행한다.

```
@Configuration
@EnableJpaRepositories(
			entityManagerFactoryRef = "엔티티팩토리매니저명",
			transactionManagerRef = "트랜잭션 매니저명",
			basePackages =  {"스캔할 레포지토리 패키지명"}
)
public class DatasourceConfig {
}
```

위와 같은 코드로 설정파일을 작성한다.

```

@Bean
public DataSource dataSource() {
    DriverManagerDataSource datasource = new DriverManagerDataSource();
    datasource.setUrl("데이터소스 url");
    datasource.setUsername("데이터소스 id");
    datasource.setPassword("데이터소스 pw");
    return datasource;
}
```
그리고 이런 식으로 DataSource 클래스의 빈을 등록해준다.

```
@Bean
public LocalContainerEntityManagerFactoryBean entityFactoryBean (@Qualifier("데이터 소스명")Datasource dataSource) {
    LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
    em.setDataSource(dataSource);
    em.setPackagesToScan(new String[] {"스캔할 패키지명"});
    em.setPersistenceUnitName("사용할 영속성유닛명");
    
    HibernateJpaVendorAdapter vendorAdapter= new HibernateJpaVendorAdapter();
    em.setJpaVendorAdapter(vendorAdapter);
    HashMap<String, Object> properties = new HashMap<>();
    //여러가지 hibernate및 jpa 설정값 삽입
    em.setJpaPropertyMap(properties);
    return em;
}
```
그리고 위와같이 데이터 소스를 LocalContainerEntityManagerFactoryBean 객체를 새로 생성한 후에 세팅을 해준다. 그리고 이 bean이 사용할 유닛명을 세팅해주고 여러 jpa 관련 설정값을 map으로 만들어준 이후 설정해서 객체를 bean으로 등록 한다.

```
@Bean
public PlatformTransactionManager transactionManager(@Qualifier("팩토리 빈 명") LocalContainerEntityManagerFactoryBean factoryBean) {
    JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setEntityManagerFactory(factoryBean);
    return transactionManager;
}
```
그리고 데이터소스별로 트랜잭션을 관리하기 위해 트랜잭션매니저를 새로 생성하면서, factoryBean을 등록해서 bean으로 등록한다.

```
    @Bean
    public DataSourceInitializer dataSourceInitializerComponent(@Qualifier("데이터소스명") DataSource dataSource){
        ResourceDatabasePopulator resourceDatabasePopulator = new ResourceDatabasePopulator();
        boolean flag = Optional.ofNullable(Boolean.parseBoolean(env.getProperty("component.jpa.init"))).orElse(false);
        if (flag) {
            resourceDatabasePopulator.addScript(new ClassPathResource("initData-component.sql"));
        }

        DataSourceInitializer dataSourceInitializer = new DataSourceInitializer();
        dataSourceInitializer.setDataSource(dataSource);
        dataSourceInitializer.setDatabasePopulator(resourceDatabasePopulator);
        return dataSourceInitializer;
    }
}
```
위 부분은 추가적인 부분인데, 매번 어플리케이션을 로드할때 db를 초기화 하고싶을 경우 초기화 옵션을 위해 빈으로 등록하는 부분이다. 

```
@PersistenceContext(unitName = "팩토리빈명")
private EntityManager entityManager;

@Bean
public JPAQueryFactory jpaQueryFactory() {
    return new JPAQueryFactory(entityManager);
}
```
그리고 QueryDsl을 사용하고자 할 경우 마찬가지로 데이터소스별로 jpaQueryFactory를 등록해야 하는데 그럴 경우 위와 같이 @PersistenceContext어노테이션에 위에서 설정했던 유닛명을 등록해서 EntityManager를 만든다. 그 이후 jpaQueryFactory에 entityManager를 넣어서 생성한 후 bean으로 등록한다.

@PersistenceContext가 주입해주는 방법은 우선, unitName에 설정한 이름을 토대로 EntityManager를 가져오는 시도를 한다. 만약 생성된 객체가 없어서 가져오기를 실패할 경우, unitName의 EntityFactoryBean에서 새로 생성해서 받아오는 동작을 한다. EntityManagerResolver의 resolveEntityManager를 기반으로 동작한다.


여기서 헷갈리는 부분이 EntityManager를 JPAQueryFactory에 설정해서 bean으로 등록할 경우, 트랜잭션별로 생성될 EntityManager를 공유해서 사용하는 것이며, 이럴경우 thread safe하지 않지않냐라는 의문이 들수 있는데, 여기서 스프링이 주입해주는 엔티티매니저(em)는 실제 동작 시점에 진짜 엔티티 매니저를 찾아주는 프록시용 가짜 엔티티 매니저이고, 이 가자 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해주기 때문에 문제가 없는 부분이다. 이러면 QueryDSL과 JPA의 동작을 위한 설정이 완료가 된다.

### 동작 원리
Spring Data JPA의 경우는 JpaRepository 인터페이스를 상속한 Repository를 생성하면, finAll(), findOne() 등의 메소드들을 그대로 사용할 수 있다. 이 JpaRepository는 PagingAndSortingRepository인터페이스를 상속한 것으로 여기에 몇가지 기능을 더 추가한 인터페이스이다. 이 인터페이스의 구현체는 SimpleJpaRepository 클래스이다.

이 JpaRepository 인터페이스를 상속받은 레포지토리를 생성한 후에, findBy컬럼명 같은 Spring Data JPA 컨벤션을 지켜서 메소드를 몇가지 생성하면, 알아서 메소드명을 컨벤션을 따라 해석하면서 JPQL을 생성하여 실행 하는 구조이다.

이렇게 메소드를 정의하여 생성되는 JPQL을 사용할 수도 있지만 Entity에 정의한 @NamedQuery의 JPQL을 직접 사용할 수도 있다. Spring Data JPA의 경우는 이 도메인 + '.' + 메서드명의 컨벤션을 지킨 @NamedQuery를 먼저 탐색해서 있으면 이 jpql을 실행하고 없으면 jpql을 생성하여 실행한다. 

참고
- https://netframework.tistory.com/entry/Spring%EC%97%90%EC%84%9C-PersistenceContext%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC
- https://catsbi.oopy.io/932715f8-7216-4278-9aeb-02d40afa9e74
- https://joont92.github.io/jpa/Spring-Data-JPA/
- https://www.netsurfingzone.com/jpa/spring-data-jpa-interview-questions-and-answers/#Difference_between_findById_and_getOne_in_Spring_Data_JPA