---
date: 2021-08-15 07:00:39
layout: post
title: "JAVA-feature(2)"
subtitle:
description:
image:
optimized_image:
category:
tags:
- JAVA
author: newvelop
paginate: false
---
저번 포스팅에 이어서 자바의 특성중에 알아야될 몇 가지를 확인해본다.

### lambda
JAVA 8부터 나온 개념으로 메소드 명같은 식별자 없이 실행 가능한 함수를 의미한다. 즉 함수를 클래스 내에서 따로 선언하지 않고 작성하여 호출하는 방식이라고 할 수 있다.

```
(매개변수, ...) -> {실행문}
```
위와 같은 구조가 람다식의 기본 형태이다.

```
@FunctionalInterface
interface Say{
    int someting(int a,int b);
}
class Person{
    public void hi(Say line) {
	int number = line.someting(3,4);
	System.out.println("Number is "+number);
    }
}
```

위의 코드는 출처에서 가져온 코드로, @FunctionalInterface란 어노테이션은 함수가 단 하나만 존재하는 인터페이스를 의미한다. Person이라는 클래스의 객체에서 hi라는 메소드를 호출하기 위해선 Say라는 인터페이스를 상속하는 구현체를 넘겨줘야 something이 실행되면서 메소드가 수행된다.

람다를 없이 작성할 경우 아래와 같이 구현할 수 있다.

```
Person rin = new Person();
rin.hi(new Say() {
    public int someting(int a, int b) {
	System.out.println("parameter number is "+a+","+b);
	return a+b;
    }
});
```

이렇게 1회용으로 작성하는 코드에도 이름을 달아줘야하는데 람다를 사용할 경우, 이름을 작성할 필요없이 1회용 메소드를 작성할 수 있다.

```
Person rin = new Person();
rin.hi((a, b)) -> {
	System.out.println("parameter number is "+a+","+b);
	return a+b;
});
```

람다를 사용할 경우 코드를 간결하게 만들 수 있으며 멀티 스레드를 활용하여 병력처리를 사용할 수 있다. 다만 디버깅하기가 매우 어렵고, 람다를 사용하여 작성한 함수는 재사용이 불가능하며, 다른 사람이 해석하기 어려울 수 있다.


### stream
마찬가지로 JAVA 8 에서 나온 개념으로 Collection의 요소를 하나씩 참조하여 람다식으로 처리할 수 있게 해주는 개념이다.
스트림은 중간연산과 단말연산이 있다. 중간 연산이라 함은 연속해서 사용할 수 있는 개념으로 map 등이 있으며 이 중간연산이 끝나고 나면 다른 형태의 stream이 반환된다.

![screensh](../assets/img/2021-08-15-JAVA---feature(2)/stream_1.png)

단말연산은 더 이상 stream을 반환하는게 아니라 stream을 닫으며, 최종 산출물을 반환하며 연산을 종료한다. collect 등이 있다.

![screensh](../assets/img/2021-08-15-JAVA---feature(2)/stream_2.png)

스트림의 경우 lazy 연산을 하기 때문에 단말 연산이 없으면 중간연산이 있어도 실행하지 않는다.

### interface method
JAVA 8 이전의 interface들은 단순히 선언만 가능했을 뿐 구현은 불가능 했었다. 하지만 8부터는 좀 다르다. interface 내에 static 메소드와 함께, 구현을 가진 default 메소드가 가능하다.

#### default 메소드 상속 문제
자바는 클래스 다중상속이 되지 않기 때문에 diamond 상속문제가 없었다. 하지만 interface는 여러개 상속이 가능하기 때문에 여기서 default 메소드는 어떻게 동작을 할지 확인을 해본다.

먼저
```
public static void main(String[] args) {
	qqqq c = new qqqq();
	System.out.println(c.a());
}

static class qqqq implements B {
}

interface B extends A {
	@Override
	default int a() {
		return 2;
	}
}

interface A {
	default int a() {
		return 1;
	}
}
```

이렇게 위와 같은 코드를 구현했을 경우, 2라는 값이 출력된다. 즉 B의 override가 잘 동작을 한것이다.

```
static class qqqq implements B, C {
}


interface C extends A {
	@Override
	default int a() {
		return 2;
	}
}

interface B extends A {
	@Override
	default int a() {
		return 2;
	}
}

interface A {
	default int a() {
		return 1;
	}
}
```
만약 이렇게 다이아몬드 형태로 A를 B와 C가 상속하고, qqqq가 A와 B를 구현한다면, 컴파일 타임에 에러가 일어난다. int a()라는 default 메소드가 겹치기 때문이다.

```
static class qqqq implements A, B {
}

interface B {
	default int a() {
		return 2;
	}
}

interface A {
	default int a() {
		return 1;
	}
}
```
다이아몬드 형태가 아니더라도, 메소드의 형태가 겹칠경우 컴파일 시점에서 에러를 출력한다. 즉 인터페이스간에 default 메소드가 있을 경우 겹치지 않게 정의를 하거나, 아니면 구현하는 클래스에서 명확하게 오버라이딩을 해야할 필요가 있다.

또 하나 주의할 점은 Object 클래스의 toString, equals, hashCode와 같은 시그니처로 default method를 정의할 수 없다.

#### static 메소드
원래는 interface 에서 static 메소드도 정의가 되지 않았다. 하지만 8로 넘어오면서 정의가 가능해졌다.

```
static class qqqq implements A {
	public int test() {
		return a();
	}
}


interface A {
	static int a() {
		return 1;
	}
}
```
다만 위와 같이 코드를 구현했을 경우 컴파일 에러가 난다. A에서 정의한 메소드 이기 때문에 A.a()로 사용해야하며, 상속한다고 해도 호출 할 수 없다.

#### private 메소드
JAVA 9에서는 또한 private 메소드를 정의할 수 있다. 이 메소드는 body를 무조건 가지고 있어야 하며, static이나 non static 둘다 가능하다. 구현 클래스와 인터페이스에선 메소드 상속이 불가능하다. 인터페이스에서는 다른 메소드를 호출할 수 있다.

### Call by Value, Reference, Address
call by value 같은 경우는 값을 전달하고 동작하는 방식을 의미한다.

```
public static void swap(int a, int b) {
	int temp = a;
	a = b;
	b = temp;
}

public static void main(String[] args) {
	int a = 10;
	int b = 20;
	swap(a, b);
	System.out.println(a);
	System.out.println(b);
}
```
위와 같은 예시를 들면 swap에 던진 a, b와 main에서의 a 와 b는 다른 변수이기때문에, 즉 값을 복사해서 던진 개념이기 때문에 swap에서 바꿨다고 그래도 main에서 영향을 주지 않는다.

Call by Reference 같은 경우는 Call by Value와는 다르게 참조하는 주소를 복사해서 던져서 호출했던 곳에서도 영향을 주는 방법이다. Java의 클래스 객체를 넘길경우가 이해 해당된다. 예를 들어 String a = "abc"; 라는 문장을 선언하면 String 타입의 참조 변수 a가 스택에 저장되고, "abc"는 힙에 저장된다. a는 "abc"의 주소를 가지고 있기 때문에 이를 메소드에 전달할 경우 주소를 전달해서 "abc"의 영역에 직접 영향을 미친다. 단 Integer 같은 primitive 타입의 wrapper 클래스는 immutable 속성을 지녀서, 넘긴다 그래도 pass by value로 넘겨진다고 하니 원하는 대로 동작하지 않는다.

여기서 call by address와 call by reference의 차이가 어떤건지를 좀 알아보면, call by address 같은 경우는 파라미터로 넘기는 실제 파라미터와, 메소드에서 사용하는 파라미터를 위해 각각 메모리 할당을 한다. 반면에 call by reference의 경우는 실제 파라미터와 메소드에서 사용하는 파라미터가 메모리를 공유한다는 차이점이 있다.


참고
- https://coding-factory.tistory.com/265
- https://sehun-kim.github.io/sehun/java-lambda-stream/
- http://kbs0327.github.io/blog/technology/java8-default-interface/
- https://flyburi.com/605
- https://askinglot.com/open-detail/328887