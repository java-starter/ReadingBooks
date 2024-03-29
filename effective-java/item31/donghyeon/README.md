---
layout: post
title: 한정적 와일드카드를 사용해 API 유연성을 높이자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



앞에서 이야기 했듯이 매개변수화 타입 (ex: List<String\> 등 )은 불공변이다. 즉 `List<String>` 과 `List<Object>` 의 하위타입이 아니라는 뜻이다.

`List<Object>` 에는 어떤 객체든지 넣을 수 있지만 `List<String>` 에는 문자열만 넣을 수 있다. 



## 불공변 방식의 문제점

다음과 같이 Stack 클래스의 public API가 있다.

```java
public class Stack<E> {
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
```

여기에 일련의 원소를 스택에 넣는 메서드를 추가한다고 가정해보자.

```java
public void pushAll(Iterable<E> src) {
  for (E e : src)
    push(e)
}
```

**이 메서드는 src의 원소타입이 Stack의 타입과 일치할 때는 정상적으로 작동한다.**

하지만, Stack의 하위타입을 넣고자 할 때는 오류가 발생한다.

다음과 같이 `Stack<Number>` 스택을 하나 선언하고, **Number의 하위 타입인 Integer 값**을 넣어보자.

**`Stack<Number>` 에 Integer 타입 추가**

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);
```

위와 같이 코드를 작성하면 **호환되지 않는 타입(incompatible types)** 이라고 컴파일 오류가 나온다.



## 왜 그럴까?

앞서 말한 것처럼 매개변수화 타입인 `List<String>` ,`Collection<Integer>`  등등은 하위타입이 없고 자기자신 그대로만 받기 때문이다. 

아무리 Number의 하위타입이 Integer라고 해도 매개변수화 타입에서 사용되면, 각각 다른 클래스로 보는 것이다.



## 해결 해보기

`Stack<Number>` 의 클래스가 Number의 하위타입인 Integer도 받고 싶으면 매개변수에 한정적 와일드카드 타입을 선언하면 된다.

`Iterable<? extends E> src` 라고 선언하면 된다. 이 코드를 해석하면, **Iterable을 받는데, E타입과 E를 확장한 클래스를 타입으로 받겠다.** 라고 해석하면 된다.

다시 Number의 예제로 돌아가서, `Stack<Number>` 로 Stack을 만들게 되면, `Iterable<? extends Number> src` 이 되기 때문에 정상적으로 컴파일이 된다.

```java
public void pushAll(Iterable<? extends E> src) {
  for (E e : src)
    push(e)
}
```

> parameterized type의 제너릭 소거
>
> Parameterized Type의 경우에는 Type erasure에 의하여 컴파일시에 `Raw Type`으로 변경된다.
>
> `List<String>`, `List<Integer>`, `List<List<String>>`의 타입정보는 컴파일 시에 타입 안정성 검증 용도로 사용될 뿐 컴파일이 완료되면 Raw Type인 List로 치환된다.



## 불공변 방식의 문제점2

이번에는 Stack 안의 모든 원소를 주어진 컬렉션으로 옮겨 담는다고 가정해보자.

**popAll 메서드**

```java
public void popAll(Collection<E> dst) {
	while (!isEmpty()) 
    dst.add(pop())
}
```

이 메서드의 문제점도 위와 비슷하다. 파라미터 타입은 불공변이기 때문에 **Stack이 Number 타입이라면 Collection도 Number 타입이여만 한다.**

**Collection<Object\> 에는 넣지 못하는 코드**

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects); // 오류가 발생한다.
```

이 오류를 해결하기 위해서는 Number의 상위타입을 받아들인다고 제너릭에게 말을 해야 한다.

**매개변수에 와일드카드 타입 적용**

```java
public void popAll(Collection<? super E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
```



## PECS 공식

**PESC 공식은 producer-extends, consumer-super 의 줄임말이다.**

위와 같이 어느 경우에는 extends를 , 다른 경우에는 super를 사용하는데, 다음 공식을 외워두면 어떤 어떤걸 사용할지 기억하는 데 도움이 된다.

첫 번째 예제에서는 `Iterable<? extends E> src` 로 extends를 사용했다. 여기서 extends가 사용된 이유는 **src은 stack에 원소를 추가하는 생산자 역할을 하기 때문이다.** 그러므로 생산자 역할을 한다면 extends를 사용한다.

두 번째 예제에서 `Collection<? super E> dst` 는 super를 사용했다. **super가 사용된 이유는 Stack으로부터 E 인스턴스를 소비하므로 super가 사용 되었다.**

**만약 생산자와 소비자의 역할을 동시에 한다면, 타입을 정확히 지정해야 하는 상황이므로 제너릭 타입을 그대로 쓰면 된다.**

## 불공변 방식의 문제점3

이번에는 좀 더 복잡한 문제를 해결해보자.

```java
public static <E extends Comparable<E>> E max(List<E> list)
```

이 메서드의 정의를 해석해보자면, **E타입의 List를 인자로 받고, 그 List에서 가장 큰 값을 리턴한다. 하지만 리턴되는 타입은 Comparable을 구현해야 한다.**

그럼 이 메서드의 인자로 `List<ScheduledFutuer<?>> scheduledFutures = ...;` 를 전달해보자.

### ScheduledFutuer 클래스

**ScheduledFuture 클래스의 부모 클래스는 Delayed 클래스이고, Delayed 클래스는 Comparable을 구현했다.**

```java
public interface Comparable<E>
public interface Delayed extends Comparable<Delayed>
public interface ScheduledFuture<V> extends Delayed, Future<V>
```

그러면 결국 ScheduledFuture 클래스도 Delayed 클래스를 상속 받았으니, Comprable을 구현한 것이나 마찬가지다.

그러나 `List<ScheduledFutuer<?>> scheduledFutures` 을 인자로 넘기게 되면,  매개변수 타입의 불공변 때문에, **ScheduledFutuer 클래스는 Comparable을 구현하지 못한 클래스가 되버린다.**

## 해결법

해결하기 위해서는 Delayed 클래스가 Comparable을 구현했으니, 이걸 사용할 수 있도록 만들어 줘야 한다.

```java
public static <E extends Comparable<? super E>> E max(List<E> list)
```

이렇게 하면 정상적으로 동작한다.

하지만 list은 publisher 역할을 하기 때문에 **PECS 공식**을 사용하여 다음과 같이 바꿔주면 좋다.

```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```



## 와일드카드와 타입 매개변수

타입 배개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다.

예를 들어, 주어진 리스트에서 명시한 두 인덱스의 아이템들을 교환하는 정적 메서드를 두 방식으로 모두 정의 해보자.

1. **비한정적입 매개변수 swap()  2. 비한정적 와일드카드 swap()**

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

만약 **public API라면** 두 번째가 낫다. 어떤 리스트든 이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해줄 것이고, 신경 써야 할 타입 매개변수도 없다.



## 와일드카드와 타입 매개변수을 정하는 규칙

**메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하자.**  이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

하지만 위에 나온 2번째 swap 메서드는 문제가 하나 있다.

```java
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

이 코드를 컴파일하면 다음의 오류메세지가 나온다.

![](./images/image1.png))

방금 꺼낸 원소를 리스트에 다시 넣을 수 없게 된다. 그 이유는 리스트이 타입이 `List<?>` 인데 , `List<?>` 에는 null 외에는 어떤 값도 넣을 수 없다는 데 있다. 

**이 오류를 해결하기 위해서는 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용하는 방법이다.** 실제 타입을 알아내려면 이 도우미 메서드는 제너릭 메서드여야 한다.

```java
public static void swap(List<?> list, int i, int j) {
  swapHelper(list,i,j);
}

//와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

**swapHelper 메서드는 리스트가 List<E\>임을 알고 있다.** 즉 이 리스트에서 꺼낸 값의 타입은 항상 E 이고 E 타입의 값이라면 이 리스트에 넣어도 안전함을 알 고 있다. 

swap 메서드 내부에서는 더 복잡한 메서드를 이용했지만, 덕분에 외부에서는 와일드카드 기반의 멋진 선언을 유지할 수 있었다. **즉, swap 메서드를 호출하는 클라이언트는 복잡한 swapHelper의 존재를 모른 채 그 혜택을 누릴 수 있다.**