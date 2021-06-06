# 아이템 43. 람다보다는 메서드 참조를 사용하라

메서드 참조를 사용하면 람다식을 더 간결하게 사용할 수 있다.

```java
public class Application {
    public static void main(String[] args) {

        Map<String, Integer> frequencyTable = new TreeMap<>();
        String[] key = {"a","b","c"};

        for (String s : key)
            frequencyTable.merge(s, 1, (count, incr) -> count + incr); // Lambda
        System.out.println(frequencyTable);

        frequencyTable.clear();
        for (String s : key)
            frequencyTable.merge(s, 1, Integer::sum); // Method reference
        System.out.println(frequencyTable);
    }
}
```

일반적으로 메서드 참조를 사용하면 람다식을 더 간결하게 사용할 수 있으나, 간혹 메서드 참조가 더 복잡하거나 의미가 불분명 할 때가 있다.
Function.identity()를 람다식으로 변경하였으며, 람다식이 훨씬 간결하고 의미가 명확하다.
```java
        return bookInfoFileRead.stream()
                        .map(bookConvert::convert)
                        .collect(Collectors.toMap(Book::getTitle, Function.identity()))

        return bookInfoFileRead.stream()
                        .map(bookConvert::convert)
                        .collect(Collectors.toMap(Book::getTitle, x -> x))
```

## 메서드 참조 유형

### 1. 정적 메서드 참조
정적 메서드를 참조한다.
```java
//method reference
Interger::parseInt

//lamdba
str -> Integer.parseInt(str)
```

### 2. 한정적 인스턴스 메서드 참조
수신 객체(receiving object;  참조 대상 인스턴스)를 특정하는 참조이다.
정적 메서드 참조 처럼 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 동일하다.
코드를 보면 수신 객체는 `Instant then`로 특정되어 있으며, 함수 객체가 받는 인수인 time과 참조되는 메서드인 isAfter의 인수가 동일하다.
```java
//method reference
Instant.now::isAfter

//lambda
Instant then = Inatant.now();
time -> then.isAfter(time)
```

### 3. 비한정적 인스턴스 메서드 참조
수신 객체(receiving object; 참조 대상 인스턴스)를 특정하지 않는 참조이다.
비한정적 참조에서는 함수 객체를 적용하는 시점에 수신객체를 알려준다.
```java
//method reference
String::toLowerCase

//lambda
str -> str.toLowerCase()
```

수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가되며, 그 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 뒤따른다. 비한정적 참조는 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰인다.
예를들어 비한정적 인스턴스 메서드 참조를 풀면 아래와 같은 구조가 된다.
참조되는 메서드 선언에 정의된 매개변수가 없기 때문에, 수신 객체 전달용 매개변수만 추가 되었다.
```java
//수신 객체 전달용 매개변수가 추가된다.
toLowerCase(String str)
```

### 4. 클래스 생성자
클래스 생성자를 가리킨다.
```java
//method reference
TreeMap<K,V>::new

//lambda
() -> new TreeMap<K,V>()
```

### 5. 배열 생성자
배열 생성자를 가리킨다.
```java
//method reference
int[]::new

//lambda
length -> new int[length]
```

## 정리
메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.

## 궁금한거
GoshThisCIassNamelsHumongous 예제코드