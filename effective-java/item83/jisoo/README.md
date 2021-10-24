# [Item83] 지연 초기화는 신중히 사용하라
## 지연 초기화 
- 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다. 
- 값이 쓰이지 않으면 초기화도 결코 일어나지 않는다. 
- 주로 최적화 용도로 쓰이지만, 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결화는 효과도 있다. 
- 다른 최적화와 마찬가지로 지연 초기화 또한 필요할 때까지는 하지 말아야 한다. 

## 지연 초기화는 양날의 검!
- 클래스 또는 인스턴스 생성 시의 초기화 비용은 줄지만 그 대신 지연 초기화하는 필드에 접근하는 비용은 커진다. 
- 초기화의 비용에 따라, 초기화된 각 필드를 얼마나 빈번히 호출하느냐에 따라 지연 초기화가 실제로는 성능을 느려지게 할 수도 있다. 
- 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화 하는 비용이 크다면 지연 초기화가 제 역활을 해줄 것이다.
- 멀티스레드 환경에서는 지연 초기화를 하기가 까다롭다. 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다. 
## 대부분의 상황에서 일반적인 초기화가 지연초기화보다 낫다. 
- 일반적인 초기화 
	```java
	private final FieldType fiedld = computeFiedlValue();
	```
- 지연 초기화 
	```java
	private FieldType field;
	
	private synchronized FiedlType getFiedl(){
		if(field == null)
			field = computeFieldValue();
		return field;
		
	}
	```
	- synchronized 접근자를 사용한 지연 초기화는 정적 필드에도 똑같이 정용된다. 물론 필드와 접근자 메서드 선언에 static 한정자를 추가해야 한다. 
	
- 정적 필드용 지연 초기화 (홀더 클래스 관용구)
	```java
	private static class FeildHolder{
		static final Field = computeFieldValue();
	}
	private static FieldType getField(){
		return FieldHolder.field;
	}
	```
	- 성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관융구를 사용하자. 클래스는 클래스가 처음 쓰일 때 비로소 초기화된다느 특성을 이용한 관용구다. 
	- getField가 처음 호출되는순가 FieldHolder.field가 처음 읽히면서, 비로소 FieldHolder 클래스 초기화를 촉발한다. 
	- 이 관융구의 장점은 getField메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 거리가 전혀 없다는것이다. 
	- 일반적인 vm은 오직 클래스를 초기화할 때만 필드 접근을 동기화할 것이다. 클래스 초기화가 끝난 후에는 VM이 동기화 코드를 제거하여, 그 다음부터는 아무런 검사나 동기화 없이 필드에 접근하게 된다. 

## 이중검사 (double-check)
- 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라. 이 관융구는 초기화된 필드에 접근할 때의 동기화 비용을 업애준다. 
- 필드의 값을 두 번 검사하는 방식으로, 한 번은 동기화 없이 검사하고, 두번째는 동기화 하여 검사한다. 
- 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile로 선언해야 한다. 

```java
private volatile FieldType field;

private FeildType getField(){
	FiedlType result = field;
	if( result != null ){
		return result;
	}
	
	synchronized(this){
		if( field == null ){
			field = computeFieldValue();
		}
		return field;
	}
}
```
- result라는 지역 변수가 필요한 이유는 
	- 필드가 이미 초기화된 상황에서는 그 필드를 딱 한번만 읽도록 보장하는 역활을 한다. 
	- 반드시 필요하지 않지만 성능을 높여주고, 저수준 동시성 프로그래밍에 표준적으로 적용되는 더 우아한 방버이다. 
- 이중검사를 정적 필드에도 적용할 수 있지만 굳이 그럴 이유는 없다. 이보다는 지연 초기화 홀더 클래스 방식이 더 낫다. 

## 이중검사 변종
- 반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화해야 할 때가 있는데, 이런 경우라면 두번째 검사를 생략할 수 있다. 이 변종의 이름은 단일검사 관용구가 된다. 
```java
private volatile FieldType field;

private FeildType getField(){
	FiedlType result = field;
	if( result != null ){
		field = result = computeFieldValue();
	}
	return result;
}
```
- 짜릿한 단일검사 (racy single-check)
	- 모든 스레드가 필드의 값을 다시 계싼해도 상관없고 필드의 타입이 long과 double를 제외한 다른 기본 타입이라면, 닽일검사의 필드 선언에서 volatile 한정자를 없애도 된다. 
	- 이 관용구는 어떤 환경에서는 필드 접근 속도를 높여주지만, 초기화가 스래드당 한 번 더 이뤄질 수 있다. 아 주 이례적인 기법으로 보통은 거의 쓰지 않는다. 