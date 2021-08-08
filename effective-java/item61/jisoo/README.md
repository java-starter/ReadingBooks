# [Item 61] 박싱된 기본 타입보다는 기본 타입을 사용하라
## 자바의 기본타입 
- int, double, boolean 기본타입
- String, List 같은 참조 타입 

## 박싱된 기본 타입
- 각각의 기본 타입에 대등하는 참조타입이 하나씩 존재 
- Integer, Double, Boolean

## 기본타입과 박싱된 기본 타입의 차이
- 오토박싱과 오토언박싱 덕분에 두 타입을 크게 구분하지 않고 사용할 수는 있지만, 차이가 사라지는 것은 아님
### 기본 타입은 값만 가지고 있으나, 박싱된 기본타입은 값에 더해 식별성이란 속성을 갖는다. 즉, 박싱된 기본타입은 값이 같아도 두 인스턴스는 서로 다르다고 볼 수 있다. 
```java
Comparatoer<Integer> naturalOrder = 
	(i,j) -> (i<j) ? -1 : (i==j ? 0:1);
```
naturalOrder.compare(new Integer(42), new Integer(42)); 의 값을 출력 해보자
두 인스턴스의 값이 같으므로 0을  예상된다.
실제로는 1을 출력한다. 즉, 첫 Integer가 두 번째보다 크다고 주장한다.     
- 첫 번째 (i < j) 에서는 오토박싱된 Integer 인스턴스는 기본 타입 값으로 변환된다. 그런 다음 첫 번쨰 정숫값이 두 번째 값보다 작은지를 평가한다. 
- 두 번째 검사에서는 두 "객체 참조"의 실벽성을 검사하게 된다. i와 j가 서로 다른 인스턴스라면 비교 결과가 false 가 되고 비교자는 1읠 반환한다. 
- 박신된 기본타입에 == 연산자를 사용 하면 오류가 일어난다. 
- 다음과 같이 해결한다. 
```java
Comparatoer<Integer> naturalOrder = (iBoxde, jBoxed)->{
	int i = iBoexed, j = jBoexed;
	(i,j) -> (i<j) ? -1 : (i==j ? 0:1);
}
```

### 기본 타입의 값은 언제나 유효하나 박싱된 기본타입은 유효 하지 않은 값을 가질 수 있다. 
```java
public class Unbelievable{
	static Integer i;
	public static void main(String[] args){
		if( i == 42){
			System.out.println("민들 수 없군");
		}
	}
}
```
- i == 42 를 검사할때 NullPointerException을 던지게 된다.
- 원인은 Integer이며 i의 초기값이 null이라는 것이다
- 기본타입과 박싱된 기본타입을 혼용한 연산에서는 박싱된 기분타입의 박싱이 자동으로 풀린다. 
- null 참조를 언박싱하면 NullPointerException이 발생한다. 
- 해결법은 i 를 int로 선언해주면 된다.

### 기본 타입이 시간과 메모리 사용면에서 더 효율적이다. 
 ```java
public static void main(String[] args){
	Long sum = 0L;
	for(long i = 0; i <= Integer.MAX_VALUE; i++){
		sum+=1;
	}
	System.out.println(sum);
}
```
- 이 프로그램은 실수로 지역변수 Sum을 박싱된 기본타입으로 선언하여 느려 졌다. 
- 박싱과 언박싱이 반복해서 일어나 체감될 정도로 성능이 느려진다. 

## 박싱된 기본 타입은 언제 쓸까 
- 컬렉션의 원소, 키 값으로 쓴다.
- 매겨변수화 타입이나 매개변수화 메서드의 타입 매개변수로는 박싱된 기본 타입을 써야한다.
- 리플렉션을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용해야 한다. 

## 결론
- 이번 아이템에서 다룬 세 프로그램의 모두 문제의 원인은 하나다. 프로그래머가 기본타입과 박싱된 기본타입의 차이를 무시한 대가를 치른 것이다. 처음 두개는 실패 했고 마지막은 심각한 성능 문제가 발상 했다. 