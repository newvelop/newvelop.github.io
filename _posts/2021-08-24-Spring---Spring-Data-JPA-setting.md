---
date: 2021-08-24 07:00:39
layout: post
title: "Spring-Spring Data JPA setting"
subtitle:
description:
image:
optimized_image:
category:
tags:
- Spring
- Spring Data JPA
- setting
- 실전! 스프링 데이터 JPA
author: newvelop
paginate: false
---
실전 스프링 데이터 JPA라는 강의를 들으면서 요약한 내용을  정리한다.

### Project Setting
[Spring initializer](https://start.spring.io/)에 들어가서 Spring과 Spring Data JPA 의존성이 담긴 프로젝트를 생성한다. 이 강의에선 maven이 아니라 gradle을 기반으로 동작하며, 스프링의 버전은 시간이 지나면 지날수록 계속 올라가기 때문에 생성하는 시점의 lts 버전을 기반으로 생성한다.

추가하는 의존성으로는 Spring Web, Spring Data JPA, lombok, h2가 있으며 개인적으로 필요한 부분이 있으면 추가해서 생성한다. 이렇게 생성하면 intellij 기반으로 동작을 할건데, intellij를 이용하여 다운로드 받은 프로젝트를 gradle project로 import한다

* 주의할점 : 처음 프로젝트를 import할 경우 gradle이 필요한 의존성을 import하는 동작을 intellij에서 수행할 것이다. 이때 gradle import가 실패하는 경우가 종종있는데 한두번 reimport 해보다가 안되면, gradle의 버전에 문제가 있을 수 도 있으니 이를 수정해서 reimport한다. 방법으로는 gradle-wrapper.properties 파일에서 distributionUrl에 적혀있는 gradle 버전을 낮춰서 import하면 동작 수행이 가능하다.

### H2 Setting
자바 기반의 RDBMS인 H2를 설치해서 간단하게 데이터의 CRUD를 수행할 수 있게 한다. 물론 실제로 운영상에서는 사용하지 않지만 간단히 테스트 할 때 유용하다.

http://www.h2database.com/html/main.html 에 들어가서 현재 스프링 프로젝트의 H2 의존성에 맞는 버전의 H2를 다운로드한다. 다운로드하고 h2를 실행하면 웹 화면에서 아래와 같은 실행 탭이 뜰 것이다.

![screensh](../assets/img/2021-08-24-Spring---Spring-Data-JPA-setting/h2.png)

이제 위의 화면에서 설정을 h2 server로 바꾼뒤, JDBC URL을 jdbc:h2:~/datajpa 이런 식으로 변경하면, H2 DB가 파일로 관리되면서 생성이 된다. 이렇게 db를 처음에 생성하고 난 이후로는 url을 다시 tcp로 바꿔서 연결을 진행한다. (ex) jdbc:h2:tcp://localhost/~/datajpa)



참고
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84