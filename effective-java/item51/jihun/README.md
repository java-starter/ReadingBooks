# 아이템 51. 메서드 시그니처를 신중히 설계하라

## 메서드 시그니처란?

메서드 선언의 일부로 **메서드명과 파라미터의 순서, 타입, 개수**에 따라 시그니처가 달라진다.

## API 설계시 고려사항

### 1. 메서드 이름을 신중히 짓자

이해할 수 있고, 같은 패키지에 속한 다른 이름들과 일관되게 짓는 게 최우선 목표다.

이름짓기가 힘들다면 자바 및 라이브러리의 API 가이드를 참조하자.

### 2. 편의 메서드를 너무 많이 만들지 말자

메서드가 너무 많다면 유지보수 및 테스트가 어려우므로 너무 많이 만들지 말자.

클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야 한다.

> Convince Method란 자주 하는 작업을 편하게 하기 위한 것으로 라이브러리 또는 프레임워크에서 꼭 필요하지 않는 서브루틴이다.

참고 : [Convenience function](https://en.wikipedia.org/wiki/Convenience_function)

### 3. 매개변수 목록은 짧게 유지하자

매개변수가 4개 이상이라면 기억하기 쉽지 않기 때문에 4개이하를 유지하도록 하자.

특히 같은 타입의 매개변수가 연달아 나오는 경우 순서가 잘못되어도 컴파일 되고 실행될 수 있으므로 유의해야 한다.

## 매개변수를 줄이는 방법

### 1. 여러 메서드로 쪼갠다

메서드를 잘 쪼개면 직교성(orthogonality)을 높여 오히려 메서드 수를 줄여주는 효과가 있다.

> 소프트웨어 설계 영역에서 "직교성이 높다"라고 하면 "공통점이 없다는 기능들이 잘 분리되어 있다" 혹은 "기능을 원자적으로 쪼개 제공한다" 정도로 해서학 수 있다.
기능을 원자적으로 쪼개다 보면, 자연스럽게 중복이 줄고 결합성이 낮아져 코드를 수정하기 수월해 진다.

대표적으로 List에서 지정된 범위의 부분리스트에서 인덱스를 찾는다고 할 때 "부분리스트의 시작, 부분리스트의 끝, 찾을 원소"까지 총 3개의 매개변수를 가진 메서드를 만들 수 있다.

```java
public class ArrayList {
    
    public List<T> subListIndexOf(int fromIndex, int toIndex, Object o) {
        
    }
}
```

하지만 java.util.List를 보면 부분리스트를 반환하는 subList 메서드와 주어진 원스의 인덱스를 알려주는 indexOf 메서드를 별개로 제공한다.

```java
public List<E> subList(int fromIndex, int toIndex) {
      subListRangeCheck(fromIndex, toIndex, size);
      return new SubList<>(this, fromIndex, toIndex);
}

public int indexOf(Object o) {
        return indexOfRange(o, 0, size);
}
```

이렇게 잘 쪼개진 메서드를 조합하면 원하는 목적을 이룰 수 있으며, 불필요한 메서드도 줄어들고 유연하게 사용할 수 있다.

**즉, 매개변수가 많다면 기능을 원자적으로 쪼개서 직교성을 높여야 한다.**

### 2. 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다.

일반적으로 이런 도우미 클래스는 정적 멤버 클래스로 둔다.

숫자(rank)와 무늬(suit)는 항상 같이 다니므로, 정적 멤버 클래스로 만들어 매개변수로 주고 받으면 깔끔해 진다.

```java
public class Blackjack {
   
    public void dealing(String gamerName, String rank, String suit) {
        
    }
    
}

public class Blackjack {
    
    static class Card {
        private String rank;
        private String suit;
    }
    
    public void dealing(String gamerName, Card card) {
        
    }
}
```

### 3. 객체 생성에 사용한 빌더 패턴을 메서드 호출에 사용한다.

매개변수가 많고 그중 일부는 생략해도 괜찮을 때 메서드 매개변수에 빌더패턴을 사용해서 객체를 생성하여 넘겨줄 수 있다.

## 매개변수의 타입으로는 클래스보다 인터페이스가 더 낫다

매개변수의 타입으로 클래스를 사용하면 클라이언트에게 특정 구현체만 사용하도록 제한하는 꼴이며, 혹시라도 입력 데이터가 다른 형태로 존재한다면 특정 명시한 특정 구현체의 객체로 옮겨 담느라 비싼 복사 비용을 치러야 한다.

예를들어서 Collections에 sort 메서드의 매개변수 타입이 ArrayList면, LinkedList를 정렬하고 싶을 시 ArrayList로 변경 후 해당 메서드를 사용할 수 있다는 문제점이 존재한다.

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    list.sort(null);
}
```

## boolean보다는 원소 2개짜리 열거 타입이 낫다

다음 코드의 정적 팩토리 메서드는 boolean 값에 따라 화씨온도 또는 섭씨온도의 객체를 return 한다.

```java
public class Temperature {

    public static class Fahrenheit extends Temperature {

    }

    public static class Celsius extends Temperature {
        
    }

    public static Temperature of(boolean fahrenheitWhether) {
        if (fahrenheitWhether) {
            return new Fahrenheit();
        }
        return new Celsius();
    }
}
```

위의 boolean 매개변수를 열거타입으로 변경함에 따라코드의 가독성이 높아지고, 확장도 쉬워졌다.

```java
public class Temperature {

    public static class Fahrenheit extends Temperature {

    }

    public static class Celsius extends Temperature {

    }

    public static Temperature of(TemperatureScale temperatureScale) {
        switch (temperatureScale) {
            case FAHRENHEIT:
                return new Fahrenheit();
            case CELSIUS:
                return new Celsius();
        }
        return null;
    }

    public enum TemperatureScale {
        FAHRENHEIT,
        CELSIUS
    }
}
```