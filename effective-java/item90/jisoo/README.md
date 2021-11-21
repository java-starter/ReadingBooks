# [[직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라]]
## 직렬화 프록시
### 활용 방안.
Serializable을 구현하기로 하면 언어의 정상 메커니즘인 생성자 이외의 방법으로도 인스턴스를 생성할 수 있게 된다. 버그와 보안 문제가 일어날 가능성이 커진다는 뜻 이 위험을 직렬화 프록시 패턴을 이용하여 크게 줄일 수 있다.
직렬화 프록시 패턴은 먼저, 바끝 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static 으로 선언한다. 이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시다. 
이때,
- 중첩 클래스의 생성자는 단하나여만 한다
- 이 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사 한다. 
그리고 바깥 클래스와 직렬화 프록시 모두 Serializable을 구현한다고 선언해야 한다. 
다음은 아이템50에서 작성 했고 [[item88]]에서 직렬화한 Period 클래스의 직렬화 프록시 클래스  이다. 

```java
private static class SerializationProxy implements Serializable {
	private final Date start;
	private final Date end;
	
	SerializationProxy(Period p){
		this.start = p.start;
		this.end = p.end;
	}
	private static final long serialVersionUID = 12312312312L;(아무값)
}
```

다음으로 바깥 클래스에 writeReplace 메서드를 추가한다.

```java
private Object writeReplace(){
	return new SerializationProxy(this);
}
```

이 메서드는 바깥 클래스의 인스턴스 대신 SerializationProxy를 반환하는 역활을 한다. 즉 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다. 
writeReplace 덕분에 직렬화 시스템은 결코 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다. 

하지만 공격자가 불변식을 훼손하고자 하면  아래의 readObject 메서드를 바깥 클래스에 추가하면 이 공격을 가볍게 막아낼 수 있다.
```java
private void readObject(ObjectInputStream stream)throws InvalidObejctException{
	throw new InvalidOBjectExcption("프록시가 필요합니다");
}
```

마지막으로, 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 SerializationProxy 클래스에 추가한다. 이 메서드는 역직렬화 시에 직렬화 시스템이 프록시를 다시 바깥 클래스의 인스턴스로 반환하게 해준다. 
readResolve 메서드는 공개된 API만을 사용해 바깥 클래스의 인스턴스를 생성한다, 이는 직렬화 생성자를 이용하지 않고도 인스턴스를 생성하는 기능을 제공한다. 이 패턴은 인스턴스를 만들때와 똑같은 생성자, 정적 팩터리, 혹은 다른 메서드를 사용해 역질려화된 인스턴스를 생성하는 것이다. 따라서 그 클래스의 정적 패렅리나 생성자가 불변식을 확인해주고 인스턴스 메서드들이 불변식을 잘 지켜 준다면, 따로 더 해줘야 할 일이 없느 것이다. 
```java
private Object readResolve(){
	return new Period(start, end);
}
```

### 장점
- 가짜 바이트 스트림 공겨과 내부 필드 탈취 공겨을 프록시 수준에서 차단해준다. 
- 필드를 final로 선언해도 되므로 클래스를 지정화 불변으로 만들 수도 있다. 
- 유효성 검사. 공격목표가 무엇이 될지 고민하지 않아도 된다. 
- 역직렬화한 인스턴스와 원래 직렬화된 인스턴스의 클래스가 다라도 정상 작동한다. 
### EnumSet
[[item36]]이 클래스는 public 생성자 없이 정적 팩터리들만 제공한다. 클라이언트 입장에서는 이 팩터리들이  EnumSet 인스턴스를 반환하는 걸로 보이지만, 열거 타입의 크기에 따라 64개 이하면 ReaularEnumSet을 사용하고, 그보다 크면 JumboEnumSet을 사용하는 것이다. 
64개짜리 열거 타입을 가진 EnumSet을 직렬화한 다음 원소 5개를 추가하고 역직렬화 하면 처음 직렬화된 것은 RegualrEnumSet 인스턴스이다. 하지만 역직렬화는 JumboEnumSet 인스턴스로 하면 좋을 것이다. 그리고 EnumSet은 직렬화 프록시 패턴을 사용해서 실제로도 이렇게 동작 한다.
```java
privaet static class SerializationProxy<E extends Enum<<E>> implements Serializable {
	private final Class<E> elementType;
	private final Enum<?>[] elements;
	
	SerializationPorxy(EnumSet<E> set){
		elementType = set.elemenType;
		elements = set.toArray(new Enum<?>[0]);
	}
	
	private Object readResolve(){
		EnumSet<E> result = EnumSet.noneOf(elemetType);
		for (Enum<?> e: elements){
			result.add((E)e);
		}
		return result;
	}
	
	....
}
```

### 한계
- 클라이언트가 멋대로 확장할 수 있는 클래스[[item19]]에는 적용할 수 없다. 
- 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다. 
- 느리다!: