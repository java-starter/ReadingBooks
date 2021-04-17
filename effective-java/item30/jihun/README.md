# 아이템 30. 이왕이면 제네릭 메서드로 만들라

## 1. 제네릭 메서드란?

메서드 선언부에 지네릭 타입이 선언된 메서드를 지네릭 메서드라 한다.

(타입 매개변수들을 선언하는) 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.

아래 코드에서 타입 매개변수 목록은 <T>이며, 반환 타입은 Set<T> 이다.

```java
public static <T> Set<T> union(Set<T> s1, Set<T> s2) {
    Set result = new HashSet();
    result.addAll(s2);
    return result;
}
```

위의 union 메서드는 입력과 반환의 타입이 모두 T로 동일하다. 이 때 한정적 와일드 카드를 사용하면 더 유연하게 사용할 수 있다.

입력 2개에 대해서는 한정적 와일드 카드를 사용함에 따라 타입 매개변수인 T의 자손은 모두 들어올 수 있게 되었다.

```java
public static <T> Set<T> union(Set<? extends T> s1, Set<? extends T> s2) {
        Set result = new HashSet();
        result.addAll(s2);
        return result;
}
```

## 2. 지네릭 클래스와 지네릭 메서드

지네릭 클래스에 정의된 타입 매개변수와 지네릭 메서드에 정의된 타입 매개변수는 전혀 별개의 것으므로 유의해야 한다.

```java
public class FruitBox<T> {
    
    public static <T> void sort(List<T> list, Comparable<? super T> comparable) {
        
    }
}
```

또한 지네릭 메서드는 지네릭 클래스가 아닌 클래스에서도 사용될 수 있다.

```java
public class Main {

    public static void main(String[] args) {

    }

    public static <T> void sort(List<T> list, Comparable<? super T> comparable) {

    }

}
```

## 3. 한정적 와일드 카드를 사용한 지네릭 메서드

아래 union 메서드는 입력과 반환의 타입이 모두 T로 동일하다. 이 때 한정적 와일드 카드를 사용하면 더 유연하게 사용할 수 있다.

```java
public static <T> Set<T> union(Set<T> s1, Set<T> s2) {
    Set result = new HashSet();
    result.addAll(s2);
    return result;
}
```

입력 2개에 대해서는 한정적 와일드 카드를 사용함에 따라 타입 매개변수인 T의 자손은 모두 들어올 수 있게 되었다.

```java
public static <T> Set<T> union(Set<? extends T> s1, Set<? extends T> s2) {
        Set result = new HashSet();
        result.addAll(s2);
        return result;
}
```

## 4. 제네릭 싱글턴 팩터리

제네릭 싱글턴 팩터리란 요청한 타입 매개변수에 맞게 그 객체의 타입을 바꿔주는 정적 팩터리 메서드 이다.

제네릭 싱글턴 팩터리가 가능한 이유는 런타임시 제네릭 타임 정보가 소거되기 때문이다. 만약 런타임시에도 제네릭이 모두 실체화 된다면 타입별로 만들어 줘야 했을 것이다.

아래 코드는 Collections에 있는 코드이다. 제네릭 싱글턴 팩터리인 emptySet 메서드가 요청한 타입 매개변수에 맞게 객체의 타입을 바꿔준다.

```java
@SuppressWarnings("rawtypes")
public static final Set EMPTY_SET = new EmptySet<>();

@SuppressWarnings("unchecked")
public static final <T> Set<T> emptySet() {
    return (Set<T>) EMPTY_SET;
}
```

emptySet 메서드가 요청한 타입 매개변수에 맞게 객체의 타입을 바꿔주기 때문에 요청한 타입 매개변수 인 String, Integer등을 사용할 수 있는 것이다.

```java
public class Main {

    public static void main(String[] args) {
        Set<String> stringSet = Collections.emptySet();
        Set<Integer> integerSet = Collections.emptySet();
    }
}
```

## 5. 재귀적 타입 한정(recursive type bound)

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정하는 것으로, 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 사용된다.

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
        list.sort(null);
}
```

T는 Comprable을 구현한 클래스이어야 하며(T extends Comparable), T 또는 그 조상의 타입을 비교하는 Comparable이어야 한다는 것(Comparable<? super T>)을 의미한다. 만일 T가 Student이고, Person의 자손이라면, <? super T>는 Student, Person, Object가 모두 가능하다.

## 정리

클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드가 있다면 모두 제네릭 메서드로 변경하자.