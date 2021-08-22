---
layout: post
title: 객체는 인터페이스를 사용해 참조하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



**적합한 인터페이스가 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하자 **

객체의 실체 클래스를 사용해야 할 상황은 오직 생성자로 생성할 때 뿐이다.

예를 들어 다음은 Set 인터페이스를 구현한 LinkedHashSet 변수를 선언하는 올바른 모습이다.

```java
// 좋은 예. 인터페이스를 타입으로 사용했다.
Set<Son> sonSet = new LinkedHashSet<>();

// 나쁜 예. 클래스를 타입으로 사용했다.
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

**인터페이스를 타입으로 사용하는 습관을 길러두면 프로그램이 훨씬 유연해진다.**

나중에 구현 클래스를 교체하고자 한다면 그저 새 클래스의 생성자를 호출해주기만 하면 된다.

```java
// sonSet의 구현체를 교체하고자 한다면
Set<Son> sonSet = new LinkedHashSet<>();

//다음과 같이 변경하면 된다.
Set<Son> sonSet = new HashSet<>();
```



# 주의할 점

- 원래의 클래스가 인터페이스의 일반 규약 이외의 특별한 기능을 제공하며, 주변 코드가 이 기능에 기대어 동작한다면 새로운 클래스도 반드시 같은 기능을 제공해야 한다.
  - 코드가 LinkedHashSet이 따르는 순서 정책을 가정하고 동작하는 상황에서 HashSet으로 바꾸면 문제가 될 수 있다.(HashSet은 순서를 보장하지 않기 때문)

# 구현 타입을 바꾸는 이유

구현 타입을 바꾸는 이유는 원래 것보다 성능이 좋거나 신기능을 제공하기 때문 일 수 있다.

예를 들어 HashMap을 참조하던 변수를 EnumMap으로 바꾸면 속도가 빨라지고 순회 순서도 키의 순서와 같아 진다.

단 EnumMap은 키가 열거 타입일 때만 사용할 수 있다. 

한편 키 타입과 상관없이 사용할 수 있는 LinkedHashMap으로 바꾼다면 성능을 비슷하게 유지하면서 순회 순서를 예측할 수 있다.

선언 타입과 구현 타입을 동시에 바꿀 수 있으니 변수를 구현 타입으로 선언해도 괜찮을 거라 생각할 수도 있다.

하지만 자칫하면 프로그램이 컴파일 되지 않는다.

예컨대 클라이언트에서 기존 타입에서만 제공하는 메서드를 사용했거나, 기존 타입을 사용해야 하는 다른 메서드에 그 인스턴스를 넘겼다고 해보자.

그러면 새로운 코드에서는 컴파일 되지 않는다.

변수를 인터페이스 타입으로 선언하면 이런 일이 발생하지 않는다.

## 적합한 인터페이스가 없는 경우 - 1

**적합한 인터페이스가 없다면 당연히 클래스를 참조해야 한다.**

String과 BigInteger 같은 값 클래스가 그렇다.

값 클래스를 여러 가지로 구현될 수 있다고 생각하고 설게하는 일은 거의 없다. 

따라서 final인 경우가 많고 상응하는 인터페이스가 별도로 존재하는 경우가 드물다.

이런 값 클래스는 매개변수, 변수, 필드, 반환 타입으로 사용해도 무방하다.

## 적합한 인터페이스가 없는 경우 - 2

적합한 인터페이스가 없는 두 번째 부류는 클래스 기반으로 작성된 프레임워크가 제공하는 객체들이다.

이런 경우라도 특정 구현 클래스보다는 보통은 추상 클래스인 기반 클래스를 사용해 참조하는게 좋다. 

OutputStream등 java.io 패키지의 여러 클래스가 이 부류에 속한다.

## 적합한 인터페이스가 없는 경우 - 3

적합한 인터페이스가 없는 마지막 부류는 인터페이스에는 없는 특별한 메서드를 제공하는 클래스들이다.

**예를 들어 PriorityQueue 클래스는 Queue 인터페이스에는 없는 comparator 메서드를 제공한다.**

클래스 타입을 직접 사용하는 경우는 이런 추가 메서드를 꼭 사용해야 하는 경우로 최소화해야 하며,

절대 남발하지 말아야 한다.



**적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 가장 덜 구체적인 클래스를 타입으로 사용하자.**