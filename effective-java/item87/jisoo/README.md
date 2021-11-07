# [Item87] 커스텀 직렬화 형태를 고려해보라.
Serializable을 구현하고 기본 직렬화 형태를 사용한다면 현재의 구현에 영환이 발이 묶이게 된다. 기분 직렬화 형태를 버릴 수 없게 되는 것. 실제로 BigInteger 같은 일부 자바 클래스가 이 문제에 시달리고 있음.    
## 직렬화에대해 고민해보고 괜찮다고 판단 될때만 기본 직렬화 형태를 사용 하라. 
  직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용. 어떤 객체의 기본 직렬화 형태는 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며, 심이어 이 객체들이 연결된 위상(topology) 까지 기술한다. 그러나 아쉽게0도 이상적인 직렬화 형태라면 물리적인 모습과 독리된 논리적인 모습만을 표핸해야 한다.
  ## 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다. 
  
  ```java
  //기본 직렬화 형태에 적합한 후보
  public class Name implements Serializable{
  /**
   * 성. null이 아니어야 함.
   * @serial
   */
   pirvate final String lastName;
   
  /**
   * 이름. null이 아니어야 함.
   * @serial
   */
   private final String firstName
   
   /**
   * 중간이름. 중간이름 없다면 null
   * @serial
   */
   private final String middleName;
  }
  
  ```
  성명은 논리적으로 이름, 성, 중간이름 이라는 3개의 문자열로 구성되며, 앞 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했다.
  
  기본 직렬화 형태가 적합하다고 결정했떠라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.  Name 클래스의 경우에는 readObject메서드가 lastName과 firstName필드가 null이 아님을 보장해야 한다. [[item88]][[item90]]
  ```java
  //기본 직려화 형태에 적합하지 않은 클래스
  public final class StringList implements Serializable{
  	private int size = 0;
	private Entry head = null;
	
	private static class Entry implements Serializable{
		String data;
		Entry next;
		Entry previous;
	}
  }
  ```
  논리적으로는 일련의 문자열을 표현, 물리적으로는 문자열들의 이중 연결 리스트로 연결했다. 
  이 클래스에 기본 직렬화 형태를 사용하면 각 노도의 양방향 연결 정보를 포함해 모든 Entry를 철두철미하게 기록 한다. 
  
  ## 객체의 물리젹 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 크게 네 가지 면에서 문제가 생긴다. 
  1. 공개 API가 현재 내부 표현 방식에 영구히 묶인다. 
	  - 앞의 예에서 private 클래스인 StringList.Entry 가 공개 API가 되어 버린다. 다음 릴리스에서 내부 표현 방식을 바꾸더라도 StringList는 여전히 연결 리스트로 표현된 입력도 처리할 수 있어야 한다. 즉 연결 리스트를 더는 사용하지 않더라도 관련 코드를 절대 제거할 수 없다. 
  2. 너무 많은 공간을 차지할 수 있다. 
	  - 연결 리스트의 모든 에트리와 연결 정보까지 기록 했지만, 엔트리와 연결 정보는 내부 구현에 해당하니 직렬화 형태에 포함할 가치가 없다. 이처럼 직렬화 형태가 너무 커져 버린다
  3. 시간이 너무 많이 걸릴 수 있다.
	  - 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 그래프를 직접 순회해볼 수 밖에없다. 
  4. 스택 오버플로를 일으킬 수 있다. 
      - 기본 직렬화 과정은 객체그래프를 재귀 순회하는데, 이  작업은 중간정도 크기의 객체그래프에서도 자칫 스택 오버 플로를 일으 킬 수 있다. 
  ## 합리적인 직렬화
```java
public static final StringList implements Serializable{
  	private transient int size = 0;
	private transient Entry head = null;
	
	// 이제는 직렬화 하지 않는다
	private static class Entry{
		String data;
		Entry next;
		Entry preivious;
	}
	
	public final void add(String s){ ... }
	
	/**
	 * 이 {@Code StringList} 인스턴스를 직렬화 한다
	 *
	 * @SerialData 이 리스트의 크기(포함된 문자열의 개수) 를 기록한 후 
	 * ({@code int}), 이어서 모든 원소를 (각각은 {@code String})
	 * 순서대로 기록한다
	 */
	private void writeOjbect(ObjectOutputStream s)throws IOException {
	
		s. defaultWirteObject();
		s.wirteInt(size);
		
		for (Entry e = head; e != null; e = e.next)
			s.writeObject(e.data);
	}
	
	private void readObject(ObjectInputStream s)
								throws IOException, ClassNotFoundException{
		s.defaultReadObject();
		int numElements = s.readInt();
		
		for (int i = 0; i < numElements; i++ ){
			add((String)s.readObject());
		}
	}

}
```

transient 한정자는 해당 인스턴스필드가 직려화 되지 않는 다는 의미를 가짐, StringList의 필드가 모두 transient 더라도 wirteObject와 readObject는 각각 가정 먼저 defaultWriteObject와 defaultReadObject를 호출한다. 이렇게 해야 향후 릴리즈에서 transient가 아닌 인스턴스 필드가 추가되더라도 상호 호환되기 때문이다. 
개선 버전의 StringList의 직렬화 형태는 원래 버전의 절반정도의 공간을 차지하며, 두배 정도 빠르게 수행된다. 마지막으로 스택 오버플로가 전혀 발생하지 않아 실질적으로 직렬화 할 수 있는 크기 제한이 없어진다.
## 해쉬테이블의 직렬화?!
해시테이블은 물리적으로 키-값 엔트리들을 담은 해시 버킷을 차례호 나열한 형태다. 어떤 에트리를 어떤 버킷에 담을지는 키에서 구한 해시코드가 결정하는데, 그 계산 방식은 구현에 따라 달라질 수 있다. 
따라서 해시 테이블에 기본직렬화를 사용하면 심각한 버그로 이어질 수 있다. 해시테이블을 직렬화한 후 역 직렬화 하면 불변식이 심각하게 훼손된 객체들이 생겨날 수 있는 것이다. 
기본 직렬화를 수용 하든 하지 않든 defaultWriteObject 메서드를 호출하면 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화 된다. 따라서 trasient로 선언해도 되는 인스턴스 필드에는 모두 transient 한정자를 붙여야 한다. 캐시된 해시 값처럼 다른 필드에서 유도되는 필드도 여기 해당한다. jvm을 실핼할 때마다 값이 달라지는 필드도 마찬가지인데, 네이티브 자료구조를 가리키는 long필드가 여기에 속함 
 > 즉 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient한정자를 생략해야 한다.     
기본 직렬화를 사용한다면 transient 필드들은 역직렬화될 때 기본값으로 초기 화 됨. 
기본 값으로 초기화가 되지 안헥 하라면 readObject에서 defaultReadObject를 호출 한 다음, 해당 필드를 원하는 값으로 복원하자[[item88]]
또는 그 값을 처음사용할 때 초기화 하는 방법도 있다[[item83]]
## 동기화 메커니즘.
기본 직렬화와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.
따라서 예를들어 모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용하라면  writeObject도 다음 코드처럼 synchronized로 선언해야 한다. 
```java
pivate synchronized void writeObject(ObjectOutputStream s) throws IOExcpetion{
	s.defaultWriteObject();
}
```
wirteObject 안에서 도기화 하고 싶다면 클래스의 다른 부부에서 사용하는 락 순서를 똑같이 따라야 한다 그렇지 않으면 자원순서 교착상태에 빠질 수 있다. 
## UID 명시적 부여
어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬버전 UID를 명시적으로 부여하자. 
구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정 하지 말자. 