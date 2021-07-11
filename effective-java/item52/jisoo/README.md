# [Item52] 다중정의는 신중히 사용하라. 

#### 다중정의 오류 케이스

#### Overloading
```java
public class CollectionClassifier {  
    public static String classify(Set<?> s){  
        return "집합";  
 }  
    public static String classify(List<?> lst){  
        return "리스트";  
 }  
    public static String classify(Collection<?> c){  
        return "그 외";  
 }  
    public static void main(String[] args){  
        Collection<?>[] collections = {  
                new HashSet<String>(),  
 new ArrayList<BigInteger>(),  
 new HashMap<String, String>().values()  
        };  
 for (Collection<?> c: collections){  
            System.out.println(classify(c));  
 }  
    }  
}
```

"집합", "리스트", "그 외"를 차례 대로 출력 할 것 같지만, 실제로 수행해보면 "그외" 만 출력 하게 됨.
이유는 ?  다중정의된 세 classify 중 어느 메서드를 호출할지가 컴파일 타임에 정해지기 때문이다. 컴파일타임에는 for 문 안의 c는 항상 Collection\<?> 타입이다. 런타임에는 타입이 매번 달라지지만, 호출할 메서드를 선택하는 데는 영향을 주지 못한다. 

```java
public static String classify(Collection<?> c){
	return c instance of Set ? "집합": 
	       c instance of List ? "리스트" :" 그외";
}
```
명시적으로 검사하면 해결 되는 부분이다.
 
> 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문이다. 

#### Overriding
```java
public class Wine {  
    String name(){ return "포도주";}  
}  
  
class SparklingWine extends Wine{  
    @Override  
 String name() {  
        return "발포성 포도주";  
 }  
}  
class Champagne extends SparklingWine{  
    @Override  
 String name() {  
        return "샴페인";  
 }  
}  
public class Overriding {  
    public static void main(String[] args){  
        List<Wine> wineList = List.of(  
                new Wine(), new SparklingWine(), new Champagne());  
 for (Wine wine : wineList){  
            System.out.println(wine.name());  
 }  
    }  
}
```

재정의 한 메소드는 wine과 무관하게 '가장 하위에서 정의한 ' 재정의한 메서드가 실행된다. 


#### 다중정의가 혼동을 일으키느 상황을 피하자. 
- 다중정의는 프로그래머가 기대한 대로 동작하지 않는다. 즉 프로그래머를 헷갈릴 수 있도록 하는 코드는 작성 하지 않는 게 좋다. 
- 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자. 
가변 인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다. 
- 다중정의하는 대신 메서드 이름을 다르게 지어주는 길도 항상 열려 있다 ! 

 
 ##### 오토박싱의 도입으로 인한 문제!
 ```java
 public class SetList {  
    public static void main(String[] args){  
        Set<Integer> set = new TreeSet<>();  
 List<Integer> list = new ArrayList<>();  
  
 	for (int i = -3; i < 3; i++){  
            set.add(i);  
 			list.add(i);  
 	}  
  
    for (int i =0; i < 3; i++){  
            set.remove(i);  
			list.remove(i);  
 	}  
        System.out.println(set+"  "+ list);  
  }  
  
}
 ```
- 예상  "[-3,-2,-1] [-3,-2,-1]"  결과 [-3,-2,-1][-2,0,-2]
- 문제가 무엇일까?
  -> set.remove(i) 시그니처는 remove(Object) 이다. 
  -> list.remove(i) 는 다중정의된 remove(int index)를 선택한다. 
- 이 문제는 list.remove의 인수를 Integer로 형변화 하여 올바른 다중정의 메서드를 선택 하게 하면됨. 
- 제네릭과 오토박싱이 등장하면서 두 메서드의 매개변수 타입이 더는 근본적으로 다르지 않게 되었다. 

#### 결론
 - 매개 변수 수가 같을때 다중정의는 피하는게 좋다.!
 