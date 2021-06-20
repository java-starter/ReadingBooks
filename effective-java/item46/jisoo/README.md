# [Item46] 스트림에서는 부작용 없느 ㄴ함수를 사용하라. 

### 스트림의 패러다임 
- 스트림의 핵심은 계산을 일련의 변환으로 재구성하는 부분. 
- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다. 
- 순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다. ( 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.)
- 이렇게 하려면 스트림 연산에 거네는 함수 객체는 모두 부작용이 없어야 한다. 

```java
Map<String, Long> freq \= new HashMap<>();  
try(Stream<String> words \= new Scanner(file).tokens()){  
    words.forEach(word -> {  
        freq.merge(word.toLowerCase(), 1L, Long::sum)  
    });  
}
```
- 문제점 ? 
	--> 스트림, 람다, 메서드 참조를 사용했고, 결과도 올바르다. 
	--> 절대 스트림 코드라 할 수 없다.  **스트림 코드를 가장한 반복적 코드다.!**
	
```java
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()){
	freq = words
		.collect(groupingBy(String::toLowerCase, counting()));
}
```

 - forEach 종단 연산은 기능이 가장 적고 '덜' 스트림답다. 
 - 대놓고 반복적이라서 병렬화할 수도 없다. 
 - forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자. 
 
 #### Collector
  - Collector를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다. 
  - 수집기는 총 세가지로, toList(), toSet(), toCollection(collectionFactory)가  그 주인공이다.  이들은 차례로 리스트, 집합, 프로그래머가 지정한 컬렉션 타입을 반환한다. 
  
##### toList
```java
List<String> topTen = freq.keySet().stream()
	.sorted(comparing(freq::get).reversed())
	.limit(10)
	.collect(toList());
```
 - toList는 Collectors의 메서드이다.  Collectors의 멤버를 정적 임포트하여 쓴것. 
 - comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드[[item14]], 그리고 한정적 메서드 참조이자, 여기서 키 추출 함수로 쓰인 freq::get은 입력받은 단어를 빈도표에서 찾아 그빈도로 반환한다. 

##### toMap
```java
private static final Map<String, Operation> stringToEnum = 
	Stream.of(values()).collect(
		toMap(Object::toString, e -> e));
```
- 위의 간단한 toMap 형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을때 적합하다. 
- 스트림 원소 다수가 같은 키를 사용한다면 파이프라인이 IllegalStateException을 던지며 종료
- 더 복잡한 형태의 toMap이나 groupingBy는 이런 충돌을 다루는 다양한 전략을 제공한다. 
```java
Map<Artistm, Album> topHits = albums.collect(
	toMap(Alubm::artist, a->a, maxBy(comparing(Albums::sales))));
```
- 인수 3개를 받는 toMap은 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다. 


```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```
- 인수가 개인 toMap은 충돌이 나면 마지막 값을 취하는 수집기를 만들 때도 유용하다. 
- 많은 스트림의 결과가 비결적적이다. 하지만 매핑 함수가 키 하나에 연결해준 값들이 모두 같을때, 혹은 값이 다르더라도 모두 허용되는 값일때 이렇게 동작하는 수집기가 필요하다. 

- 마지막 toMap은 네 번째 인수로 맵 팩터리를 받는다. 이 인수로는 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다. 

##### groupingBy
입력으로 분류 함수를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 답은 수집기를 반환한다. 
```java
words.collect(groupingBy(word -> alphabetize(word)))
```
- groupingBy가 반환하는 숮집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림 수집기도 명시해야한다. 다운스트림 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다. 
- 이 매개변수를 가숑하는 가장 간단한 방법은 toSet()을 넘기는 것이다. 
```java
Map<String, Long> freq = words 
	.collect(groupingBy(String::toLowerCase, counting()));
```
- 다운스트림 수집기로 counting()을 건네는 방법도 있다. 이렇게 하면 각 카테고리를 해당 카테고리에 속하는 원소의 개수와 매핑한 맵을 얻는다. 
-
##### joining 
- 이 메서드는 CharSequence 인스턴스의 스트림에만 적용할 수 있다. 이 중 매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기를 반환한다. 
