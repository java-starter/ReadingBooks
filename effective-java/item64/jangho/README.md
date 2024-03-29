# [이펙티브 자바] Item64 - 객체는 인터페이스를 사용해 참조하라

적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하자. 실제 클래스를 사용해야하는 상황은 생성할 때뿐이다.

인터페이스를 타입으로 선언하는 습관은 프로그램을 훤씬 유연하게 만들어 준다.

# 인터페이스를 타입으로 선언하자.

```java
// Good
Set<Son> sonSet = new LinkedHashSet<>();

// Bad
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

위 예시에서 구현 클래스를 교체하고자 한다면 아래와 같이 변경할 수 있다.

```java
Set<Son> sonSet = new HashSet<>();
```

인터페이스를 타입으로 사용한 선언은 간단하게 새로운 구현 클래스로 교체할 수 있다.

## 주의사항

만약 인터페이스의 일반 규약 이외의 특별한 기능에 의존하고 있다면, 새로운 클래스도 반드시 이 기능을 제공해야한다. 

예를 들어, LinkedHashSet 선언 주변 코드가 순서가 보장됨을 가정하고 동작하는 상황에서 HashSet으로 변경하면 문제가 생길 수 있다. (HashSet은 순서를 보장하지 않기 때문에)

# 클래스 참조를 해야하는 경우..

적합한 인터페이스가 없다면 당연히 클래스를 참조해야한다.

## 참조해도 괜찮은 녀석들?

### 1. 값 클래스

String, BigInteger와 같은 값 클래스는 여러가지로 구현될 수 있다고 생각하고 설계하는 일은 거의 없다. 따라서 final인 경우가 많고 인터페이스가 별도로 존재하는 경우도 드물다. 이런 **값 클래스는 매개변수, 변수, 필드, 반환 타입으로 사용해도 무방하다.**

### 2. 클래스 기반으로 작성된 프레임워크 객체

OutputStream 등 java.io 패키지의 여러 클래스가 이 부류에 속한다. 프레임워크가 제공하는 클래스기반 객체라도 특정 구현 클래스보다는 기반(추상) 클래스를 참조하자.

### 3. 인터페이스에는 없는 특별한 기능을 제공하는 클래스(권장 x)

특별한 기능이 꼭 필요한 경우 최소한의 범위내에서 클래스 참조를 사용해도 된다. 예를 들어, PriorityQueue 클래스는 Queue 인터페이스에는 없는 comparator 메서드를 제공한다. 이런 추가 메서드가 필요한 경우에만 사용해야 한다.

마지막 예시는 인터페이스 대신 클래스 타입을 사용해도 되는 예도 있음을 보여주기 위한 것일 뿐이므로 권장하지 않는다.

# 핵심정리

- 클래스보다는 인터페이스나 추상 클래스를 참조하자.
- 적합한 인터페이스가 없다면 필요한 기능을 만족하는 **가장 덜 구체적인 클래스**를 타입으로 사용하자.