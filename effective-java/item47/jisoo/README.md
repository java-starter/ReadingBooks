# [Item47] 반환 타입으로는 스트림보다 컬렉션이 낫다. 
- 원소 시퀀스, 즉 일련의 원소를 반환하는 메서드는 수없이 많다.
- 자바 7까지는 Collection, Set, List 같은 컬렉션 인터페이스, 혹은 Iterable이나 배열을 썻다. 
 	--> 기본은 컬렉션 인터페이스, 반환 된 원소 시퀸스가 일부  Collection 메서드를 구현할 수 없을때는 Iterable 인터페이스 사용 , 반환 원소들이 기본 타입이거나 성능에 민감한 상황이라면 배열을 사용 
- 스트림은 반복을 지원하지 않는다. 
- 스트림 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드르 ㄹ전부 포함할 뿐만아니라, Iteralbe 인터페이스가 정의한 방식대로 동작한다. 그럼에도 스트림을 반복할 수 없는 까닭은 stream 이 Iterable을 확장하지 않아서다.

```java
for (ProcessHandle ph : ProcessHandle.allPorcesses()::itertor){
}
```
- 위코드는 오류를 낸다
```java
for(proecessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator)
```
- 작동은 하지만 너무 난잡하고 직관성이 떨어진다. 

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream){
	return stream::iterator;
}

for (ProcessHandle p: iterableOf(PorcessHandle.allProcessese())){
}
```
- 어댑터 메서드를 사용 하면 상황이 나아진다.
- 자바는 이런 메서드를 제공하지 않지만 위 코드와 같이 만들 수 있다. 
- 이 경우 에는 자바의 타입추론이 문맥을 잘파악하여 어댑터 메서드 안에서 따로 형변환 하지 않아도 된다. 

```java
public static <E> STream<E> streamOf(Iterable<E> iterable){
	return StreamSupport.stream(iterable.spliterator(), false);
}
```
- 반대의 경우의 어댑터 클래스 Iterable -> Stream
- 객체 시퀀스를 반화하는 메서드를 작성하는데, 이 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 마음 놓고 스트림을 반환하게 해주자.  하지만 공개 API를 작성할 떄는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 모두를 배려해야 한다. 

##### Collection Interface
- Collection 인터페이스는 Iterable의 하위 타입이고 stram 메서드도 제공하니 반복과 스트림을 동시  지원한다.
- 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection 이나 그 하위 타입을 쓰는 게 일반적으로 최선이다. 
- 반환 하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet 같은 표준 컬렉션 구현체를 반환하는게 최선일 수 있다. 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다. 
- 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토해보자. 
- AbstractList를 이용하면 훌륭한 전용 컬렉션을 손쉽게 구현할 수 있다.  

```java
public class PowerSet {  
    public static final <E\> Collection<Set<E\>> of(Set<E\> s){  
        List<E\> src = new ArrayList<>(s);  
 if(src.size() > 30)  
            throw new IllegalArgumentException(  
                    "집합에 원소가 너무 많습니다(최대 30개). : " \+ s);  
 return new AbstractList<Set<E\>>() {  
            @Override  
 public Set<E\> get(int index) {  
                Set<E\> result = new HashSet<>();  
 for (int i = 0; index != 0; i++, index >>=1)  
                    if((index & 1) == 1)  
                        result.add(src.get(i));  
 return result;  
 }  
  
            @Override  
 public int size() {  
                return 1 << src.size();  
 }  
  
            @Override  
 public boolean contains(Object o) {  
                return o instanceof Set && src.containsAll((Set) o);  
 }  
        };  
 }  
}
```

- AbstractCollection을 활용해서 Collection 구현체를 작성할 때는 Itrable용 메서드 외에 2개만 더 구현하면 된다. 바로 Constains와 size이다. 
- Constains와 size를 구현하는게 불가능  할 때는 컬렉션보다는 스트림이나 Iterable로 반환하는 편이 낫다. 


```java
public class SubLists {  
    public static <E\> Stream<List<E\>> of(List<E\> list){  
        return Stream.concat(Stream.of(Collections.emptyList()),  
 prefixes(list).flatMap(SubLists::suffixes));  
 }  
  
    private static <E\> Stream<List<E\>> prefixes(List<E\> list) {  
        return IntStream.rangeClosed(1, list.size())  
                .mapToObj(end -> list.subList(0, end));  
 }  
      
    private static <E\> Stream<List<E\>> suffixes(List<E\> list){  
        return IntStream.range(0, list.size())  
                .mapToObj(start -> list.subList(start, list.size()));  
 }  
}
```
- Stream.concat 메서드는 반환되는 스트림에 빈 리스트를 추가
- flatMap메서드는 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만든다. 
- IntSTraem.range와 IntStraem.rangeClosed 가 반환하는 연속되는 정수값들을 매핑 

이상으로 스트림을 반환하느 두 가지 구현을 알아봤는데, 모두 쓸만은 하다. 
- 하지만 반복을 사용하는 게 더 자연스러운 상황에도 사용자는 그냥 스트림을 쓰거나 Stream을 Iterable 로 변환해주는 어댑터를 이용해야 한다. 
- 어댑터는 클라이언트 코드를 어수선하게 만들고 느리다
- 직접 구현한 전용 Collection을 사용하니 코드는 훨씬 지저분햇졋지만 스트림을 활요한 구현보다 빨랏다. 