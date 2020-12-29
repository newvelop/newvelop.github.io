---
date: 2020-12-29 07:00:39
layout: post
title: "Spring Framework-Architecture(1)"
subtitle:
description:
image:
optimized_image:
category:
tags:
- spring
- webdev
- framework
author: newvelop
paginate: false
---
전자정부 프레임워크의 기반이자 Java로 웹 어플리케이션을 개발할시 제일 많이 접하게 되는 프레임워크인 스프링에 대한 설명을 작성해보고자 한다.
우선 스프링의 대표적인 3가지 특징을 먼저 확인한 후에 구조적인 부분을 차례대로 확인한다.

### 제어 역전(Inversion of Control)
스프링 프레임워크는 자바에서 웹개발을 돕기위한 프레임워크로 제어 역전이란 개념을 지니고 있다. 이전의 개발은 개발자가 프로그램의 흐름을 제어하며 객체의 라이프사이클을 관리하였다. 하지만 이 제어 역전 개념에선 개발자가 객체의 라이프사이클을 관리하는 것이 아닌, 프레임워크에서 주도적으로 관리하고 개발자는 이를 그냥 사용하는 방식이다. 스프링 프레임워크에선 IoC 컨테이너가 IoC 개념을 수행하며, 이를 통해 DI(Dependency Injection)을 이용할 수 있다(ex)@Autowired).

참고로 DI를 할 경우 Field Injection은 나쁜 패턴이라 하는데 이는 차후 DI를 돌아보며 정리하도록 한다.

### AOP(Aspect Oriented Programming)
관점 지향이라고 하는 이 개념은 스프링에서 제공해주는 기능으로, 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어서 보고 그 관점을 기준으로 각각 모듈화해주는 관점이다.
흩어진 관점들을 모아서 한곳에서 실행할 수 있게 해주는 개념으로 이 개념도 차후 살펴보며 정리하도록 한다.

### PSA(Portable Service Abstraction)
사용하는 라이브러리의 클래스를 직접 가져다가 쓰는 것이 아니라, POJO(Plain Old Java Object)를 이용하여 객체지향적인 특성을 살려 구현을 하고, 라이브러리의 기능을 이용할 수 있게 추상화를 해주는 개념을
스프링에서 제공해준다. 예를 들어 스프링에선 서블릿 어플리케이션을 만들지만, 개발자가 직접 서블릿을 제작하는 일은 없다. 이는 PSA를 통해 컨트롤러 등을 서블릿으로 만들어주는 작업을 스프링에서 하고있다는
것이다. 이 개념도 차후 정리하도록한다.

### 구조
위와 같이 3가지 스프링의 특성을 확인했다. 이와 같은 제공하는 스프링의 구조는 아래의 그림과 같다.
![screensh](../assets/img/2020-12-28-Spring-Framework---Architecture(1)/spring-overview.png)

기능을 제공하기위해 스프링에선 20여개 정도의 모듈로 구성되어있으며 크게 Core Container, Data Access/Integration, Web, AOP, Instrumentation, Test 부분으로 나뉘어져 있다.

#### Core Container
Core Container는 Core, Beans, Context, Expression Language 모듈들로 이루어져있다.

Core와 Beans는 IoC와 DI 등 프레임워크의 핵심 기능을 제공해주는 모듈들로, 팩토리 패턴으로 구현된 BeanFactory 클래스가 싱글톤 패턴의 필요성을 없애주며 프로그램 로직을 구현할 수 있게 해준다.

Context 모듈은 

Expression Language 모듈은 

#### Data Access/Integration


### 결론
이렇듯 웹 어플리케이션을 구축하는데 있어서 프레임워크는 매우 유용한 장점들을 제공하지만, 약간의 단점을 안고 있기 때문에 사용할 땐 이러한 점들을 고려해야한다.

참고
- https://docs.spring.io/spring-framework/docs/3.0.x/spring-framework-reference/html/overview.html
- https://dailyworker.github.io/spring-triangle/
- https://www.vojtechruzicka.com/field-dependency-injection-considered-harmful/
- https://engkimbs.tistory.com/746
