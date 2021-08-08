# [Item60] 정확한 답이 필요하다면 float와 double은 피하라
## float 와 double
- float와 double 은 공학 계산용으로 설계되었다.넓은 범위의 수를 빠르게 정밀한 '근사치' 로 계산하도록 세심하게 설계 되었다. 
- 즉, 정확한 결과가 필요할 때는 사용하면 안된다. 특히 금융관련 계산과는 맞지 않는다.
- 금융 계산과 관련된 예제 
```java
public static void main(String[] args){  
        double funds = 1.00;  
	 int itemsBought = 0;  
	 for(double price = 0.10; funds >= price; price += 0.10){  
				funds -= price;  
	 itemsBought++;  

	 }  
	 System.out.println(itemsBought +"개 구입");  
 	System.out.println("잔돈(달러):" + funds);  
 }  
}
```
- 프로그램을 실행해보면 사탕 3개를 구입한 후 잔돈은 0.399999999999 달러가 남았음을 알게된다.

## BIgDecimal
- 이 문제를 올바르게 해결하기 위해서는 BigDecimal, int 혹은 long을 사용해야 한다. 
- 위의 문제를 BigDecimal을 사용해 해결 한 코드 
```java
public static void main(String[] args){  
    final BigDecimal TEN_CENTS = new BigDecimal(".10");  
  
	 int itemsBought = 0;  
	 BigDecimal funds = new BigDecimal("1.00");  
	 for(BigDecimal price = TEN_CENTS;  
			 funds.compareTo(price) >= 0;  
			 price = price.add(TEN_CENTS)){  
		funds = funds.subtract(price);  
	 	itemsBought++;  
	 }  
  
    System.out.println(itemsBought +"개 구입");  
 	System.out.println("잔돈(달러) : "+ funds);  
}
```
- 문제는 해결 되었지만. BigDecimal은 기본 타입보다 쓰기가 불편하고, 훨씬 느리다는 문제가 있다. 

## long 
- BigDecimal의 대안으로 int 혹은 long타입을 쓸 수도 있다. 그럴 경우 값의 크기가 제한되고, 소수점을 직접 관리해야 한다. 
```java
public static void main(String[] args){
	int itemsBought = 0;
	int funds = 100;
	for( int price = 10; funds >= price; price += 10){
		funds -= price;
		itemsBought++;
	}
	System.out.println(itemsBought +"개 구입");  
 	System.out.println("잔돈(센트) : "+ funds);  
}
```
- 모든 개산을 달러 대신 센트로 수행하면 모든 일이 해결 된다. 

## 정리 
- 정확한 답이 필요한 계산에는 float나 double을 피하라. 
- 9자리 십진수로 표현 할 수 있다면 int 사용
- 8자리 십진수를 표현 할 수 있다면 long 사용
- 18자리를 넘어가면 BigDecimal을 사용
- Big Decimal 이 제공하는 여덟 가지 반올리 모드를 이용하여 반올림을 완벽히 제어 할 수 있다. 
- 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지않다면  int나 long을 사용하라. 
