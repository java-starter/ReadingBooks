[ITEM75] 예외의 상세 메시지에 실패 관련 정보를 담으라
## 사후분석을 위해 실패 순간의 상황을 정확히 포착해 예외의 상세 미시지에 담아야 한다.
- 실패 순간을 포착하려면 발생환 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다. 
- 관련된 데이터를 모두 담아야 하지만 장황할 필요는 없다. 소스코드에서 얻을 수 있는 정보는 기렉 늘어놔바야 군더더기가 될 뿐인다. 
## 예외의 상세 메시지와 최종사용자에게 보여줄 오류 메시지를 혼동해서는 안된다. 
-  최종 사용자에게는 친절한 메시지, 예외 메시지는 가독성보다는 담긴 내용이 훨씬 중요하다. 
## 실패를 적절히 포착하려면 정보를 예외 생성자에서 모두 받아서 상세 메시지까지 미리 생성해놓는 방법도 괜찮다. 
```java
public IndexOutOfBoundsException(int lowBound, int upperBound, int index){
	//실패를 포착하는 상세 메시지를 생성
	super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, 		                       index));
	this.lowerBound = lowerBound;
	this.upperBound = upperBound;
	this.index = index;
}
```
