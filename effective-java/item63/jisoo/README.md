# [Item63] 문자열 연결은 느리니 주의하라
## 문자열 연결 연산자(+)
- 문자열 연결 연산자는 여러 문자열을 하나로 합쳐주느 편리한 수단이다. 
- 그런데 한 줄짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현을 만들 때라면 괸찮지만, 본격적으로 사용하기 시작하면 성능 저하를 감내하기 어렵다. 
- <b>문자열 연결 연산자로 문자열 n개를 잇는 시간은 n∧2 에 비례한다. </b>
- 문자열은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야하므로 성능 저하는 피할 수 없는 결과다. 
```java
public String statement(){
	String result= "":
	for(int i = 0; i < numItems(); i++){
		result += lineForItem(i);
	}
	retrun result;
}
```
## 성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자.

```java
public String statement2(){
	StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
	for(int i = 0; i < numItems(); i++){
		b.append(lineForItem(i));
	}
	return b.toString();
}
```
- java 6 이후 문자열 연결 성능을 다방면으로 개선했지만, 이 두 메서드의 성능차이는 여전히 크다. 