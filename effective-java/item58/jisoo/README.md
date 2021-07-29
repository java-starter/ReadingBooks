# 전통적인 for 문보다는  for-each 문을 사용하라

## 전통적인 for 문의 단점
```java

for(Iterator<Element> i = c.iterator(); i.hasNext();){
	Element e = i.next();
	... do something
}

for(int i = 0; i < a.length; i++){
	... do somthing
}
```
- while 문 보다는 낫지만 가장 좋은 방법은 아님.[[Item57]]
- 우리에게 필요한건 원소 뿐. 반복자와 인덱스 변수는 코드를 지저 분하게 한다. 
- 요소의 종류가 늘어나면 오류가 생길 가능성이 높아진다.
- 컬렉션이냐 배열이냐에 따라 코드 형태가 상당히 달라지게 된다.

## for-each (향상된 for 문)
```java
for(Element e : elements){
	...
}
```
- 반복자와 인덱스 변수를 사용하지 않음.
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있다.
- 반복 대상이 컬렉션이든 배열이든, for-each 문을 사용해도 속도는 그대로다. 

### 반복자에서의 버그
```java
enum Suit{ CLUB, DIAMOND, HEART, SPADE}  
enum Rank{ ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,  
 NINE, TEN, JACK, QUEEN, KING }  
  
...  
  
static Collection<Suit> suits = Arrays.asList(Suit.values());  
static Collection<Rank> ranks = Arrays.asList(Rank.values());  
  
List<Card> deck = new ArrayList<>();  
for(Iterator<Suit> i = suits.iterator(); i.hasNext();){  
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext()){  
        deck.add(new Card(i.next(), j.next()));  
 }  
}
```
- 해당 코드는 마지막줄의  i.next()에서 NoSuchElementException 을 던지게 된다.

### 반복자에서의 버그 2
```java
enum Face {ONE, TWO, THREE, FOUR, FIVE, SIX}  
  
Collection<Face> faces = EnumSet.allOf(Face.class);  
  
for(Iterator<Face> i = faces.iterator(); i.hasNext();){  
    for(Iterator<Face> j = faces.iterator(); j.hasNext();){  
        System.out.println(i.next() + " "+ j.next());  
 }  
}
```
- 예외를 던지진 않는다. 
- 36개의 조합이 나와야 하지만 6개의 조합 밖에 나오지 않음. 
```java
for(Iterator<Suit> i = suits.iterator(). i.hasNext();){
	Suit suit = i.next();
	for(Iterator<Rank> j = ranks.iterator();j.hasNext();){
		deck.add(new Card(suit, j.next()));
	}
}
```
- 위와 같이 바깥 반복문에 바깥 원소를 저장하는 변수를 하나 추가해 해결하였다. 
### for-each로 해결
```java
for(Suit suit : suits){
	for(Rank rank : ranks){
		deck.add(new Card(suit, rank));
	}
}
```
- for-each 문으로 간단히 해결!

## for-each 사용하지 못하는 상황
### 1. 파괴적인 필터링(destructive filtering) 
 - 컬렉션을 순회하면서 선택된 원소를 제거 해야 한다면 반복자의 remove메서드를 호출 해야 한다. 
 - 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다. 
### 2. 변형(transforming)
- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체 할때 
### 3. 병렬 반복 (parallel iteration)
 - 여러 컬렉션을 병렬로 순회해야 한다면 각가의 반복자ㅏ와 잍덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다. 
  
## for-each 사용을 위한 Iterable
- for-each 문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든지 순회 할 수 있다. 
- Iterable을 처음 부터 직접 구현하기느 까다롭지만, 원소들의 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현하는 쪽으로 고민해보자. (Collection 인터페이스는 구현하지 않았더라도...)
- Iterable을 구현하면 for-each문을 사용가능 하기 떄문에!
****
  