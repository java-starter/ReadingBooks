# [Item53] 가변인수는 신중히 사용하라. 

## 가변인수? 
  명시한 타입의 인수를 0개 이상 받을 수 있다. 가장 먼저 인수의 개수와 길이가 같은 배열을 만드록 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다. 
  
  ```java
  static int sum(int... args){
  	int sum = 0;
	for (int arg : args){
		sum += arg;
	}
	return sum;
  }
  ```
  
  인수가 1개 이상이어야 할떄
  
  ```java
  static int min(int.. args){
  	if(args.length == 0)
		throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
	int min = args[0];
	for(int i=1; i < args.length; i++){
		if(args[i] < min)
		 	min = args[i];
	}
	return min;
  }
  ```
 -  문제점!
     -> 런타임에 실패한다는점! 
	 -> 코드가 지저분 하다는점!

인수가 1개 일떄 가변인수를 제대로 사용하는 방법. 
```java
static int min(int firstArg, int... remainingArgs){
	int min = firstArgs;
	for(int arg : remainingArgs){
		if(arg < min)
		 	min = arg;
	}
	retrun min;
}
```

### 성능에 민감하다면 가변인수가 걸림돌이 될 수 있다. 
- 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다. 
- 성능 문제를 고려해여 사용하여야 한다. !
