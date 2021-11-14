# [item89] 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라
## 싱글턴 패턴의 직렬화
[[item3]] 에서 싱글턴 패턴을 보여주 었다, 싱클턴 클래스는 바깥에서 생성자를 호추하지 못하게 막는 방식으로 인스턴슨가 오직 하나만 만들어 짐을 보장 했다. 
싱글턴 클래스의 선어네 implements Serializable을 추가하는 순간 더 이상 싱글턴이 아니게 된다. 기본 직렬화를 쓰지 않더라도[[item88]], 명지적인 readObject를 제공하더라도 소용 없다. 어떤 readObject를 사용하는 이 클래스가 초기화될때 만들어진 이스턴스와는 별개인 인스턴스를 반환하게 된다. 
## readResolve 
readResolve 기능을 이욯하면 readObject가 만들어낸 인스턴스를 다른것으로 대체할 수 있다. 역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의해 뒀다면 역직려화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환한다. 이때 새로 생성된 객체의 참조는 유지되지 않으므로 가비지 컬렉션의 대상이 됨 
```java
private Object readResolve(){
	 return INSTANCE;
}
```
이 메서드는 역직렬화 한 객체는 무시하고 클래스 초기화 때 만들어진 인스턴스를 반환한다. 이 클래스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으니 모든 인스턴스 피드를 transient로 선언해야 한다. 그렇지 않으면 [[item88]]에서 MutalblePeriod와 같은 공격이 될 여지가 남는다. 

## readResolve공격 방법
싱글턴이 trasient가 아닌 필들를 가지고 있따면, 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화 된다.  그 시점에 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다. 
```java 
public Class Elvis implements Serializble {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis(){}
	
	private String[] favoriteSongs = {"Hound dog", " HearBreak Hotel"};
	
	public void printFavorite(){
		Sysyem.out.println(Arrays.toString(favoriteSongs));
	}
	
	private Object readResolve(){
		return INSTANCE;
	}
}
```

```java
public class ElvisStealer impelments Serializable{

	static Elvis impersonator;
	private Elvis payload;
	
	private Object readResolve(){
		impersonator = payload; //resolve 되기 전의 elvis 인스턴스의 참조 저장
		return new String[] {"A FOOL such as I"};
		
	}
	private static final long serialVersionUID = 0;
}
```
먼저 readResolve  메서드와 인스턴스 필드 하나를 포함한 도둑 클래스를 작성한다. 이 인스턴스 필드는 도둑이 숨길 직렬화된 싱글턴을 참조하는 역할을 한다. 
직렬화된 스트림에서 싱글턴의 비휘발성 필드를 이 도둑의 인스턴스로 교체한다. 
이제 싱글턴은 도둑을 참조하고 도둑은 싱글턴을 참조하는 순환고리가 만들어졋다. 
싱글턴이 도둑을 포함함으로 역직렬화될 때 도둑의 readResolve메서드가 먼저 호출 된다. 그 결과 도둑의 readResolve메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬호 도중인 싱글턴의 참조가 담겨 있게 된다. 
도둑의 readResolve 메서드는 이 인스턴스 필드가 참조한 값을 정적필드로 복사하여 readResolve가 끝난 후에도 참조할 수 있도록 한다. 그런 다음 이 메서드는 도둑이 숨긴 trasent가 아닌 필드의 원래 타입에 맞는 값을 반환한다. 

```java
public class ElvisImpresonator{
	private staic final byte[] serializedForm = {
			....
	};
	
	public static void main (String[] args){
		Elvis elvis = (Elvis) deserialize(serializedForm);
		Elvis impersonator = ElvisStealer.impersonator;
		
		elvis.priftFavorites();  
		impersonator.priftFavorites();  
	}
}
```

	출력결과 
	[Houng Dob, Heartbreak Hotel]
	[A FOll Such as I]
이 프로글매을 실행하면 다음 결과를 출력한다. 이것으로 서로 다른 2개의 elvis 인스턴스를 생성할 수 있음을 증명 했다. 

## 열거타입 싱글턴 패턴
favoritSongs 필드를 trasient로 선언하여 이 문제를 고칠 수 있지만 evlis를 원소 하나짜리 열거 타입으로 바꾸는 편이 더 나은 선택이다.
```java
public enum Elvis{
	INSTANCE;
	private String[] favoriteSongs = {"Hound Dog", "Hearbreak Hotel"};
	public void printFavorites(){
		System.out.println(Arrays.toString(favoriteSongs));
	}
}
```
인스턴스 통제를 위해 readResolve는 컴파일타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황에 쓸때 적절하다.

## 접근성
final 클래스에서라면 readResolve 메서드는 private이어야 한다. final이 아닌 클래스에서는 다음 몇가지를 주의 해야 한다. 
- prvate으로 선언하면 하위클래스에서 사용할 수 없다.
- package-private으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용 할 수 있다. 
- protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다. 