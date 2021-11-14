# [Item88] readObject 메서드는 방어적으로 작성하라.
## 방어적 복사를 사용하는 불변 클래스
```java
public final class Period {
	private final Date start;
	private final Date end;
	
	pulbic Period(Date start, Date end){
		this.start = new Date(start.gameTime());
		this.end = new Date(end.getTiem());
		if (this.start.compareTo(this.end) > 0)
			throw new IllegalArgummentException(start + " after " + end);
	}
	
	public Date start() { retrun new Date(start.getTime());}
	public Date end() { return new Date(end.getTime());}
	public String toString() { return start + " - " + end };
}
```
[[item50]] 에서 불변인 날짜 범위 클래스를 만드는 데 가변인 Date 필드를 이용하여 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 Date를 방어적으로 복사 하였다. 
이 클래스를 직렬화 하기로 결정 했을 떄 , 물리적 표현이 논리적 표현과 부합함으로 기본 직렬화 형태[[item87]]를 사용해도 나쁘지 않다.  하지만 이렇게 하면 이 클래스의 주요한 불변식을 더는 보장하지 못한다.  문제는 readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다. 따라서 다른 생성자와 똑같은 수준으로 주의를 기울여야 한다. 보통의 생성자처럼 readObject 메서드에서도 인수가 유효한지 검사해야 하고[[item49]] 필요하다면 매개변수를 방어적으로 복사해야 한다[[item50]]
 readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다. 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다. 하지만 불변식을 깨트릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 생긴다. 이는 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해낼 수 있기 때문이다. 
 ## 불변식을 깨트리는 직렬화
 ```java
 public class BogusPeriod{
 
 	private static final byte[] serializedForm = {
		... 바이트...
	};
	
	public static void main(String[] args){
	
		static Object deserialize(byte[] sf){
			try{
				return new ObjectInputStream(
					new ByteArrayInputStream(sf)).readObject();
			}catch (IOException | CalssNotFoundException e ){
				throw new IllegalArgumentException(e);
			}
		}
	}
 }
 ```
 보시다시피 직렬화 Period를 직렬화할 수 있도록 선언한 것만으로 클래스의 불변식을 꺠트리는 객체를 만들 수 있게 된 것이다. 
 
 ## 유효성 검사하는 readObject
 ```java
 private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundExeption{
 	s.defaultReadObjefct();
	
	if(strart.compareTO(end) > 0 ){
		throw new InvalidObjectException(start + " after " + end);
	}
 }
 ```
 위 작업으로 공격자가 허용되지 않는 Period 인스턴스를 생성하는 일을 막을 수 있지만, 아직도 미묘한 문제가 하나 숨어 있따.  ***정상  Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변  Period 인스턴스를 만들어 낼 수 있다.*** 이제 이 참조로 얻은 Date 인스턴스들을 수정 할 수 있으니, Period는 더이상 불변이 아니게 된다. 
 
 가변 공격의 예
 ```java
 public Class MutablePeriod{
 	public final Period period;
	
	public final Date start;
	
	public final Date end;
	
	public MutablePeriod(){
		try{
			ByteArrayOutPutStream bos = new ByteArrayOutPutStream();
			ObjectOutputStream out = new ObjectOutputStream();
			
			out.writeObject(new Period(new Date(), new Date()));
			
			byte[] ref = {0x71, 0, 0x72, 0, 5}; //참조 #5
			bos.write(ref); // 시작(start) 필드
			ref[4] = 4; // 참조 #4
			bos.write(ref); // 종료(end) 필드
			
			ObjectInputStream in = nmew ObjectInputStream(
				new ByteArrayInputStream(bos.toByteArray()));
			period = (Period) in.readObject();
			start = (Date) in.readObject();
			end = (Date) in.readObject();
		}catch (IOException | ClassNotFoundException e){
			throw new AssertionError(e);
		}
	}
 }
 ```
```java
public static void main(String arge){
	MutablePeriod mp = new MutablePeriod();
	Period p = mep.period;
	Date pEnd = mp.end;
	
	pEnd.setYear(78);
	system.out.println(p);
	
}
```
이 예에서 Period 인스턴스느 불변식을 유지한 채 생성 됐지만, 의도적으로 내부의 값을 수정할 수 있었다. 이것이 너무 극단적인 예가 아닌것이, 실제로도 보안 문제를 String이 불변이라는 사실에 기댄 클래스들이 존재하기 때문이다. 

## 방어적 복사
문제의 근원은 Period의 readObject 메서드가 방어적 복사를 충분히 하지 않은데 있다. 
객체를 역질려화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드들을 모두 반드시 방어저긍로 복사해야 한다. 
 ```java
 private void readObject(ObjectInputStream s) throws IOExcption, ClassNotFoundExcpetion{
 	s.defaultReadObject();
	
	start = new Date(start.getTime());
	end new Date(end.getTime());
	
	if(start.compreTo(end) > 0){
			throw new IllegalArgummentException(start + " after " + end);
	}
 
 }
 ```
 방어적 복사를 유혀성 검사보다 앞서 수행하며, Date의 Clone메서드를 사용하지 않느것은 곡역으로부터 보호하는데 필요하다[[item50]]
 final 필드는 방어적 복사가 불가능하니, 이 readObject를 사용하려면 start와 end필드에 final 한정자를 제거 해야한다. 
 
 ## readObject를 써도 좋을지 판단 하는법
 transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가하면 안되면 커스텀 readObject를 만들어 모든 유효성 검사와 방어적 복사를 수행 해야 한다. 
 또는 직렬화 프록시패턴[[item90]]을 사용하는 방법도 있다. 이 패턴은 적극 권장된다. 