# [Item62] 다른 타입이 적절하다면 문자열 사용을 피하라 
## 문자열은 다른 값 타입을 대신하기에 적합하지 않다. 
- 데이터가 수치형이라면  int, float, BigInteger 등 적당한 수치 타입으로 변환해야 한다. 
- ''예/아니오" 의 질문의 답이라면 Boolean으로 변환해야 한다. 
## 문자열은 열거 타입을 대신하기에 적합하지 않다. 
- 상수를 열거할 때는 문자열보다는 열거 타입이 월등히 낫다. 
## 문자열은 혼합 타입을 대신하기에 적합하지 않다. 
```java
String compoundKey = classNAme + "#" +i.next();
```
- 위의 방식은 단점이 많은 방식이다. 두 요소를 구분해주는 # 이 두 요소중 하나에서 쓰엿다면 혼란 스러운 결과를 초래한다. 
- eqauls, toString, compareTo 메서드를 제공할 수 없으며 String이 제공하는 기능에만 의존해야한다. 
- 차라리 전용 클래스를 새로 만드는 편이 낫다. 이런 클래스는 보통 private 정적 멤버 클래스로 선언한다 [[item24]]
## 문자열은 권한을 표현하기에 적합하지 않다. 
- 예를 들어 스레드 지역변수 기능을 설계한다고 해보자. 그 이름처럼 각 스레드가 자신만의 변수를 갖게 해주는 기능이다. 이 기능의 설계는 클라이언트가 제공한 문자열 키로 스레드별 지역변수를 식별한 것이다. 
```java
public class ThreadLocal{
	private ThreadLocal(){}
	public static void set(String key, Object value);
	public static Object get(String key,);
}
```

- 이 방식의 문제는 스레드 구분용 문자열 키가 전역 이름공간에서 공유된다는 점이다. 이 방식이 의도대로 동작하려면 각 클라이언트가 고유한 키를 제공해야 한다. 만약 두 클라이언트에서 같은 키를 쓰기로 결정한다면, 의도치 않게 같은 변수를 공유하게 된다. 
- 이 API는 문자열 대신 위조할 수 없는 키를 사용하면 해결된다. 이 키를 권한이라고도 한다.
```java
public calss ThreadLocal{
	private ThreadLocal(){}
	public static class Key{
		key(){}
		
	}
	public static Key getKey(){
		returns new Key();
	}
	public static void set(Key key, Object value );
	public static Object get(Key key);
}
```
- 위 방법은 앞서의 문자열 기반 API의 문제 두 가지를 모두 해결해주지만, 개선 할 여지가 있다.
- set과 get 읜 정적 메서드일 이유가 없다. 
- 이렇게 하면 Key는 더 이상 스레드 지역변수를 구분하기 위한 키가아니라, 그 자체가 스레드 지역 변수가 된다. 

```java
public final class ThreadLocal{
	public ThreadLocal();
	public void set(Object value);
	public Object get();
}
```

- Object를 실제타입으로 형변해 써야 해서 타입안전하지 않다. 

```java
public final class ThreadLocal<T>{
	public ThreadLocal();
	public void set(T value);
	public T get();
}
```