# 람다보다는 메서드 참조를 사용하라
람다가 익명클래스 보다 나은점은 코드의 간결함이다. 그런데 메서드참조는 더 간단 하다.
#### 람다 표현식

```java
map.merge(key, 1,(count, incr)-> count +incr);
```

- 다음과 같은 merge 함수는 키, 값, 함수를 메서드로 받으며 키 가없다면 값을 저장하고, 키가 있으면 함수를 현재 값과 주어진 값에 적용한 다음, 그  결과로 현재 값을 덮어쓴다. 
- 깔끔해 보이지만 매개변수 count와 incr은 크개 하는 일이없이 공간을 꽤 차지한다. 
- Java8 이며 되면서 Integer 클래스는 이 림다와 기능이 같은 정적 메서드 sum을 제공

#### 메서드 참조

```java
map.merge(key,1, Integer::sum)
```
- 위와 같이 수정 가능
- 메서드 참조를 사용하는 편이 보통은 더 짧고 간결하므로, 람다로 구현했을때 너무 길거나 복잡하다면 메서드 참조가 좋은 대안이 되어준다. 
- 즉 람다로 작성할코드르 새로운 메서드에 담은 다음, 람다 대신 그 메서드 첨조를 사용하는 식이다. 
##### 메서드 참조의 유형
###### 1. 정적 멧서드를 가라키는 메서드 참조. 
```java
Integer::parseInt
```
```java
str -> Integer.parseInt(str)
```
###### 2. 수신 객체를 특정하는 한정적인스턴스 메서드 참조. 
```java
Instance.now()::isAfter  
```
```java
Instant then = Instatn.now();
t -> then.isAfter(t)
```
- 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다. 
###### 3. 수신  객체를 특정하지 않는 비한적정 인스턴스 메서드 참조. 
```java
String::toLowerCase   
```
```java
str -> str.toLowerCase();
```
- 함수 객체를 적용하는 시점에 수신 객체를 알려준다. 
- 이를 위해 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가 되며,  그 뒤로는 참조되는 메서드 선어에 정의된 매개변수들이 뒤를 따른다. 
###### 4. 클래스 생성자
```java
TreeMap<K,V>::new 
```
```java
() -> new TreeMap<K,V>()
```
###### 5. 배열 생성자
```java
int[]::new
```
```java
len -> new int[len]
```
#### 반대의 경우?
```java
service.execute(GoshThisClassNameIsHumongous::action);
```

```java
service.execute(() -> action());
```

- 메서드 참조 쪽은 더 짧지도, 더 명확하지도 않다. 따라서 람다 쪽이 낫다. 

