#[ITEM74] 메서드가 던지는 모든 예외를 문서화하라. 

## 검사 예외는 따로따로 선언하자
- 공통 상위 클래스 하나로 뭉뚱그려 선언하는 일을 삼가자. 
	- 메서드 사용자에게 각 예외에 대처할 수 있는 힌트를 주지 못함. 
	- 다른 예외들 까지 삼켜버릴수 있음. 
	- 단, main 메서드는 Exception을 던지도록 선언해도 괜찮다. 
## 예외가 발생하는 상황을 자바독의 @throws 태그를 사용하여 정확히 문서화 하자. 
-	잘 정비된 비검사 예외 문서는 사실상 그 메서드를 성공적으로 수행하기 위한 전제 조건이 된다. 
-	발생 가능한 비검사 예외를 문서로 남기는 일은 인터페이스 메서드에서 특히 중요하다. 
-	이 조건이 인터페이스의 일반 규약에 속하게 되어 그 인터페이스를 구현한 모든 구현체가 일관되게 동작하도록 해주기 때문이다. 
-	메서드가 던질 수 있는 예외를 각각 @throws 태그로 문서화하되, 비검사 예외는 메서드 선언의 throws 목록에 넣지 말자.
-	한 클래스에 정으된 많은 메서드가 같은 이유로 같은 예외를 던진다면 그예외를 클래스 설명에 추가 하는 방법도 있다. 