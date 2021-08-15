---
date: 2021-08-15 07:00:39
layout: post
title: "JAVA-feature(1)"
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
자바의 특성중에 알아야될 몇 가지를 확인해본다.

### 자료구조
자바에서 제공하는 자료구조를 쉽게 사용하기 위한 구현체 클래스들 중에서 몇가지 대표적인 클래스들을 살펴 본다.
![screensh](../assets/img/2021-08-15-JAVA---feature(1)/datastructure.png)

#### ArrayList
대표적으로 많이 사용하는 클래스이다. 배열의 원소를 추가, 삭제 할 수 있는 메소드를 제공하는데 이는 동적으로 늘어나는 구조는 아니다. ArrayList안에는 초기 capacity와 배열이 있어서 이 capacity만큼의 사이즈 배열을 생성해서 사용한다. 그러다가 배열의 사이즈보다 더 많은 원소를 가지고 있어야 할 경우, 배열을 재할당하고 원소들을 복사해서 사용한다. 이때 재할당하는 크기는 기존 크기 * 1.5 사이즈로 할당해서 사용한다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/arraylist.png)

그리고 배열을 기반으로 사용하는 것이기 때문에 중간 삽입이나, 삭제에 비효율적이다. 탐색을 자주해야될 경우 사용이 권장된다.

#### LinkedList
이 또한 데이터를 다루는 클래스로써 리스트의 구현체이다. 이 클래스의 경우는 양방향 연결 리스트로 구성되어있어서 처음부터 끝까지, 아니면 역순으로 순회할 수 있는 구조이다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/linkedlist.png)

추가나 삭제의 경우, head 부터 해당 위치까지 직접 찾아가야 되는 탐색의 문제는 있으나, 앞뒤의 연결 포인트를 제거/변경만 하기 때문에 arraylist보다는 낫다고 볼수 있다. 다만 탐색은 O(n)이 걸린다.

즉 결론적으로 중간에서의 삽입, 추가가 빈번하게 일어나거나, 데이터의 개수가 많이 변경될 경우는 LinkedList를, 단순 탐색이나 순차적 삽입/삭제가 일어날 경우는 ArrayList를 권장한다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/list-timecomplexity.png)

#### HashMap
자료구조중에 key와 value 매핑을 통해 값을 저장해야될 때 사용하는 Map의 구현체 클래스이다. key의 정보를 hash 알고리즘을 통해 해시값을 추출한 후, 이 hash값으로 key value 매핑을 하는 것이다. 문제는 해시 충돌이 날 가능성이 충분히 있기 때문에 이 충돌을 어떻게 해결할 수 있는 지를 확인해본다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/hashmap.png)

JAVA 8 이전의 경우에는 Separate chaining이라고 하여, key의 해시가 겹칠 경우 그대로 linked list로 key와 value를 저장하는 구조로 해결을 했다고 한다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/separate-chaining.png)

JAVA 8 부터는 좀 다르게 처리하는데, 데이터가 8개가 모이면 linked list를 red black tree로 변경하고, 6개로 줄어들 경우 linked list로 변경을 한다고 한다.

##### RED BLACK TREE
이진 탐색 트리를 개선했다고 보면 되는데, 일반 이진트리 같은 경우는 루트 노드보다 크거나 작은 값들만 들어오다보면 한쪽으로 쏠려서 삽입되는 경우가 존재한다. 이런 점들을 개선하기 위해 나온 개념이 RED BLACK TREE이다.

이 트리를 만들기 위한 조건으로는 4가지가 있는데
1. 루트 노드의 색은 검정이다.
2. 리프 노드의 색은 검정이다
3. 빨강 노드의 자식은 검정이다 -> 빨강 노드가 연속으로 나올 수 없다.
4. 모든 리프노드에서 루트노드까지 가는데 검정 노드의 개수는 동일하다.
가 존재한다.

또한 입력되는 노드는 무조건 빨강인데 이 조건을 맞추기 위한 동작을 하나씩 살펴본다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/RBT1.png)

위의 그림은 빨강 노드가 연속으로 두개 나와서 조건 위배되는 케이스인데, 이를 해결하기 위해선 두가지 방법이 있다. Restructuring과 Recoloring이 있다고 하는데 이 두가지를 살펴본다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/RBT2.png)
![screensh](../assets/img/2021-08-15-JAVA---feature(1)/RBT3.png)

Z라고 표시된 노드가 새로 삽입된 노드라고 하면 Z의 부모는 V이고, 삼촌은 W이다. 이때 W의 색에 따라서 어떤 해결책을 사용할지 정해지는데 삼촌이 검정이면 Restructuring, 빨강이면 Recoloring을 한다고 한다.

Restructuring의 경우는
1. 삽입 노드 - 삽입 노드의 부모 - 삽입 노드의 할아버지를 오름차순으로 정렬
2. 가운데 값을 부모로 만들고 나머지 둘을 자식으로 만듦
3. 올라간 가운데 있는 값을 검정으로 만들고 나머지 두 자식을 빨강으로 만듦
이라고 한다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/RBT4.png)

위의 그림과 같이 본인 - 부모 - 할아버지 3개의 노드를 선택해서 오름차순으로 정렬한다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/RBT5.png)

그리고 3개의 노드중 가운데 값을 부모로 하여 트리를 만든다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/RBT6.png)

그 다음, 가운데를 검정으로, 자식을 빨강으로 칠한다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/RBT7.png)

마지막으로 기존에 있던 자식을 붙이면 완성이다.
이 Restructuring의 경우는 다른 서브트리에 영향을 끼치지 않기 때문에 한번의 동작으로 끝나며 시간 복잡도는 O(logn)이라고 한다.

Recoloring의 경우는
1. 현재 삽입된 노드의 부모와 그 형제를 검정으로 하고 할아버지를 빨강으로 한다.
2. 할아버지가 루트노드가 아니였을 경우 Double Red가 가능하기 때문에 그 경우 한번 더 작업을 해준다.
의 순으로 동작한다.

#### HashTable
해시맵과 마찬가지로 key value 구조를 지니는데, hashmap과 다른 점은 thread safe하게 동작한다는 것인다. 왜냐하면 모든 API 메소드에 synchronized가 걸려있어서 하나의 thread가 진입하면 lock이 걸려서 다른 thread들은 대기를 해야하기 때문이다. 하지만 이 클래스는 ConcurrentHashMap이 나온 이후로는 잘 안쓰인다고 한다.

#### ConcurrentHashMap
HashTable과 비슷하게 HashMap의 thread unsafe한 부분을 해결하기 위해 나온 클래스이다. HashTable과 다른점은 읽기에는 synchronized가 걸려있지 않으며, 쓰기 작업이라고 해서 무조건 lock 이 걸리지는 않는다는 점이다. 이 클래스는 map의 버킷 단위로 lock을 걸고있기 때문에 같은 버킷의 요청이 아닌경우는 여러 쓰레드가 동시 작업이 가능하다.

또 Map에서 그냥 HashMap, LinkedHashMap, TreeMap이 있는데 차이는 HashMap은 순서 보장이 전혀 안됨, LikedHashMap은 삽입 순서로 인한 순서 보장은 됨, TreeMap은 키로 정렬도 가능 이라고 보면 된다.

또한 synchronized로 동작하는게 아니라 compare and swap으로 동작을 한다고 하는데 JAVA 5에서 나온 개념이라고 한다.

```
    public static class MyLock {
    private AtomicBoolean locked = new AtomicBoolean(false);

    public boolean lock() {
        return locked.compareAndSet(false, true);
    }

}
```

이런식으로 lock 메소드에 synchronized를 거는게 아니라, AtomicBoolean을 사용해서 false일 경우 true로 바꾸고 작업을 하게 한다고 한다.

#### Set
셋은 HashMap과 동작은 비슷하나 차이점으로는 key value 구조가 아니라 key만 존재한다는 점이다. 나머지 구조는 map과 비슷하기 때문에 자세한 설명은 생략하겠다.

#### Collection 선택
여러가지의 Collection들을 위에서 확인했는데, 이 Collection들중 어떤 것을 선택해야 될지 고민될 때가 있다. 그땐 하단의 그림을 참고하면 좋을 것이다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/choose-collection.png)

### 제네릭
ArrayList나 LinkedList처럼 안에 들어가는 데이터의 타입에 의존하지 않고 하나의 타입의 데이터를 이용하여 동작하는 클래스들이 있다. 이런 동작들을 할 수 있게 해주는 제네릭이다. 기본적인 형태로 <> 안에 타입을 넣어서 사용하면 된다. 

제네릭을 이용하면
1. 제네릭을 사용하면 잘못된 타입이 들어올 수 있는 것을 컴파일 단계에서 방지할 수 있다.

2. 클래스 외부에서 타입을 지정해주기 때문에 따로 타입을 체크하고 변환해줄 필요가 없다. 즉, 관리하기가 편하다.

3. 비슷한 기능을 지원하는 경우 코드의 재사용성이 높아진다.

등의 장점이 있다고 한다.

기본적으로 제네릭을 사용할때 T 가 타입, E가 엘리먼트, K가 키, V 가 값, N 이 넘버로 사용된다고 하는데 이름 짓는건 나름이지만 일반적인 컨벤션을 지켜주면 좋다.

제네릭을 하나만 사용할 수 있는 건 아니고 &lt;K, V&gt;와 같이 여러개를 사용할 수 있다. 다만 타입으로는 primitive 타입은 사용할 수 없고 Reference Type만 사용할 수 있다. 즉 클래스만 사용할 수 있다는 소리이다.

```
class ClassName<E> {
	
	private E element;	// 제네릭 타입 변수
	
	void set(E element) {	// 제네릭 파라미터 메소드
		this.element = element;
	}
	
	E get() {	// 제네릭 타입 반환 메소드
		return element;
	}
	
}
```
위와 같이 사용하는게 제네릭의 일반적인 사용법이다. 다만 클래스 내에서 통일 해서 사용할 수 도 있지만, 메소드에만 적용해서 사용하고 싶을 수도 있다. 그럴 경우는 메소드에 직접 사용해서 동작할 수 있다.

```
class ClassName<E> {
	
	private E element;	// 제네릭 타입 변수
	
    public <E> E genericMethod(E o) {	// 제네릭 메소드
		...
    }

	void set(E element) {	// 제네릭 파라미터 메소드
		this.element = element;
	}
	
	E get() {	// 제네릭 타입 반환 메소드
		return element;
	}
	
}
```

위와 같이 사용할 경우, genericMethod의 E는 ClassName의 E와 다를 수 있다. 이 메소드에 직접 적용하는 방법은 static 메소드나 함수 등에 사용하기 위해 필요하다. Class의 객체에 제네릭을 사용하는 경우, 프로그램이 동작하면서 타입을 정의해줄 수 있기 때문에(ex) ArrayList&lt;Integer&gt;) 상관없지만 static 같은 경우는 이미 실행하는 시점에 메모리에 올라가기 때문에 타입이 따로 동작할 필요가 있다.

#### 제네릭 제한
이 제네릭을 단순하게 문자만 사용하면 어떤 클래스든 들어올 수 있다. 하지만 이 제네릭을 이용하여 구현한 클래스가 특정 데이터 타입에 국한되어 동작될수가 있다. 예를 들어 Person이라는 클래스를 사용하여 동작하는 PersonCount라는 클래스를 구현했다고 해보자. 그럴 경우 사람의 부류인 Student나 Teacher의 클래스는 와도 되나, Car 같이 사람이 아닌 것들은 동작할 수 없기 때문에 올수있는 타입의 제약을 둘 필요가 있다.

K super T나 ? super T를 사용할 경우, T타입의 조상클래스를 타입으로 사용할 수 있다는 것이며, K extends T나 ? extends T는 T의 자식 클래스를 타입으로 사용할 수 있다는 것이다. ?와 K의 차이점은 K를 사용할 경우, T에 관련된 클래스 단 하나만을 내부에서 사용할 수 있으며, ?의 경우는 T에 관련된 모든 클래스 객체가 내부에서 사용이 가능하다. 특정 타입을 명시하고 그 안에서 사용할 경우 K를 사용하면 된다.


### 어노테이션
JAVA 1.5에서 나왔던 개념이다. 클래스나 속성등에 어떤 동작을 할 지, 이 것은 어떤 것을 의미하는 지등의 정보를 삽입해주는 개념을 어노테이션이라고 한다.

예를 들어서 @Getter와 @Setter가 lombok에서 지원이 되는데, 이 어노테이션은 해당 클래스에 모든 필드의 getter와 setter를 만들어줘 라는 의미의 정보를 나타낸다고 볼 수 있다. 이 개념은 interface를 정의한 후, 상속해서 사용해도 동일한 동작을 할 수 있지만 이렇게 되면 필요한 곳마다 상속해서 사용해야하기 때문에 어노테이션을 이용해서 처리 하면 매우 간편하다.

```
@Target(ElementType.?)
@Retention(RetentionPolicy.?)
public @interfacet 이름{

}
```

위와 같이 정의 할 수 있으며, 어노테이션의 범위인 ElementType과 용도인 RetentionPolicy는 아래와 같다.

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/elementType.png)

![screensh](../assets/img/2021-08-15-JAVA---feature(1)/retentionPolicy.png)

이렇게 정의하고, 클래스나 필드에 적용하면 컴파일 타임에 해당 어노테이션을 해석시킬 수 있고, 또한 런타임에 리플렉션을 이용하여 동작할 수 있게 할수 있다.


참고
- https://devlog-wjdrbs96.tistory.com/64
- https://stackoverflow.com/questions/21974361/which-java-collection-should-i-use
- https://medium.com/tanay-toshniwal/count-distinct-elements-in-input-sequence-using-java-hashmaps-373a58697dd2
- https://zeddios.tistory.com/237
- https://st-lab.tistory.com/153
- https://www.nextree.co.kr/p5864/
- https://blog.naver.com/kang594/39704853