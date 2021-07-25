# [Itme57] 지역변수의 범위를 최소화 하라. 
## 가장 처음 쓰일떄 선언하기 !
- 미리 선언부터 해두면 코드가 어수선해져 가독성이 떨어진다. 
- 실제 사용시점엔 타입과 초깃값이 기억이 나지 않을 수도 있다 . 
- 지역변수의 범위는 서언된 지점부터 그 지점을 포함한 블록이 끝날 때까지이다. 

## 거의 모든 지역변수는 선언과 동시에 초기화 하자
- 초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선은을 미뤄야  한다. 
- try-catch 무은 이규칙에서 예외이다. 변수를 초기화 하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록 안에서 초기화해야 한다. (그렇지 않으면 예외가 블록을 넘어 메서드에까지 전파 된다. )
- try블록 바깥에서도 사용해야 한다면 try블록 앞에서 선언해해야한다. 

## 반복문에서의 변수 
- 반복변수의 값을 반복문이 종료된 뒤에더 써야 하는 상황이 아니라면 while 문보다 for문을 쓰는 편이 낫다.

```java
for(Element e : c){
	...
}
```

```java
for(Iterator<Element> i = c.iterator(); i.hasNext();){
	Element e = i.next();
}
```
 - 위 코드는 컬렉션을 순회할 때 권장 하는 관용구다. 
 
 ```java
 Iterator<Element> i = c.iterator();
 while(i.hasNext()){
 	doSomething(i.next());
 }
 
 Iterator<Element> i2 = c2.iterator();
 while(i.hasNext()){
 	doSomething(i2.hasNext());
 }
 ```
 
 ```java
 for(Iterator<Element> i = c.iterator(); i.hasNext();){
 	...
 }
 
 for(Iterator<Element> i2 = c2.iterator(); i.hasNext();){
 	...
 }
 
```
- while 문은 i의 범위 가 계속되어 두번쨰 while문도 작동하게 되어 프로그램이 정상 작동 한다.  하지만 for문의 i는 범위 가 첫번쨰 for문 종료와 함께 끝나기 때문에 컴파일 단계에서 오류를 잡을 수 있다. 