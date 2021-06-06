# 익명 클래스보다는 람다를 사용하라

#### Function Object 
- 예전 자바에서 함수 타입을 표현할 때 추성 메서드를 하나만 담은 인터페이스를 사용 했다. 
- 특정 함수나 동작을 나타내는 데 사용
- JDK 1.1 등장이후 함수 객체를 만드는 주요 수단은 익명 클래스[[item24]]가 되었다. 

```java
Collections.sort(words, new Comparator<String>(){
	public int compare(String s1, String s2){
		return Integer.compare(s1.length(), s2.length());
	}
}
```


#### 함수형 인터페이스
- 자바8 에서 추성 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었다. 
- 함수형 인터페이스들의 인스턴스를 람다식을 사용해 만들수 있게 되었다. 
- 람다는 함수나 익병 클래스와 개념은 비슷하지만 코드는 훨씬 간결한다. 

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length));
```
-여기서 람다, 매개변수(s1,s2), 반환값의 타입은 각각 (Comparator\<String>),  String, int 지만 코드에서는 언급이 없다. 우리 대신 컴파일러가 문맥을 살펴 타입을 추론해준 것이다.
- 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.   
```java
public enum Operation {  
    PLUS("+", (x,y) -> x + y),  
 MINUS("-", (x,y) -> x - y),  
 TIMES("\*", (x,y) -> x \* y),  
 DEVICE("/", (x,y) -> x / y);  
  
 private final String symbol;  
 private final DoubleBinaryOperator op;  
  
 Operation(String symbol, DoubleBinaryOperator op) {  
        this.symbol \= symbol;  
 this.op \= op;  
 }  
    @Override public String toString(){return symbol;}  
  
    public double apply(double x, double y){  
        return op.applyAsDouble(x,y);  
 }  
}
```
람다 기반 Operation 열거 타입을 보면 상수별 클래스느 몸체는 더 이상 사용할 이유가 없다고 느낄지 모르지만, 꼭 그렇지 않다. 
람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다. 
람다는 한줄일 때 가장 좋고 길어야 세줄 안에 끝내는게 좋다. 

#### 람다로 잉명 클래스로 대체할 수 없는 곳 
- 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야한다. 
- 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 잉명 클래스를 쓸 수있다. 
- 람다는 자신을 참조할 수없다. 람다의 this는 바깥 인스턴스를 카리킨다. 그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야한다. 
- 람다도 잉명 클래스처럼 직렬화 형태가 구현벼로 다를 수 있따. 따라서 람다를 직렬화하는 일을 극히 삼가야 한다.(익명 클래스의 인스턴스도 마찬가지). 직렬화해야만 하는 함수 객체가 있따면 private 정적 중첩클래스[[item12]]의 인스턴스를 사용하자. 
- 
