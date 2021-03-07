# 아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

Java 8부터 인터페이스의 정의된 메서드에 구현부를 추가할 수 있는 길이 열렸지만, 모든 기존 구현체들과 매끄럽게 연동되리라는 보장이 없기 때문에 유의해야 한다.

## 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다.

대표적인 예가 org.apache.commons.collections4.collection.SynchronizedCollection이다.
해당 클래스는 Collection을 상속받으며, Java 8에 추가된 default 메서드인 removeIf() 메서드는 오버라이드 하고 있지 않다.

해당 클래스는 클라이언트가 제공한 객체로 Lock을 거는 기능을 제공하는데, Collection의 removeIf() 메서드는 이러한 기능이 없기 때문에 예기치 못한 결과가 발생할 수 있다.

> org.apache.commons.collections4.collection.SynchronizedCollection은 commons-collections4에 존재한다.

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

## 정리

기존 인터페이스에 default 메서드를 추가하는 것은 최대한 지양해야 한다. 왜냐하면 default 추가함에 따라 기존 구현 클래스들이 전혀 다르게 동작할 가능성이 존재하기 때문이다. 대표적인 예가 위에서 설명한 org.apache.commons.collections4.collection.SynchronizedCollection 이다.

하지만 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는데 아주 유용한 수단이다.

> Spring의 WebMvcConfigurer 인터페이스를 상속받을 시 Java 8 이전에는 사용하지 않을 설정도 모두 오버라이드 할 수 밖에 없었는데 Java 8의 default 메서드가 등장함에 따라 WebMvcConfigurer 인터페이스의 모든 메서드가 default로 바뀜에 따라 필요한 것만 오버라이드 하게 되어 코드가 훨씬 깔끔해 졌다.







