---
title:  
date: [[2021-05-16]] 1:04 PM 
tags: #effectivjava #java
---
# [Item37] ordinal 인덱싱 대신 EnumMap을 사용하라

### ordinal 사용 
배열이나 리스트에서 원소를 꺼낼때 ordinal 메서드[[Item35]]로 인덱스를 얻는 코드
정원에 심은 식물들을 배열 하나로 관리하고, 이들의 생애 주기별로 묶는 코드
```java
class Plant{
	enum LifeCycle {ANNUAL, PERENNIAL, BIENNAIL}
	
	fianl String name;
	fianl LifeCycle lifeCycle;
	
	Plant(String name, LifeCycle lifeCycle){
		this.name = name;
		this.lifeCycle = lifeCycle;
	}
	
	@Override public String toString(){
		return name;
	}
}

```

```java
Set<Plant>[] plantsByLifeCycle =  
        (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];  
  
for (int i =0; i <plantsByLifeCycle.length; i++)  
    plantsByLifeCycle[i] = new HashSet<>();  
  
  
Plant[] garden = new Plant[6];  
garden[0] = new Plant("가", Plant.LifeCycle.ANNUAL);  
garden[1] = new Plant("나", Plant.LifeCycle.BIENNIAL);  
garden[2] = new Plant("다", Plant.LifeCycle.PERENNIAL);  
garden[3] = new Plant("마", Plant.LifeCycle.ANNUAL);  
garden[4] = new Plant("바", Plant.LifeCycle.BIENNIAL);  
garden[5] = new Plant("사", Plant.LifeCycle.PERENNIAL);  

//ordinal의 값이 배열의 인덱스
for (Plant p:  garden)  
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);  
  
for (int i = 0; i < plantsByLifeCycle.length; i++)  
    System.out.printf("%s: %s%n",
	Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
```

#### 문제점!
- 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야한다.     
      --> 깔끔히 컴파일되지 않을 것이다. 
- 배열은 각 인덱스의 의미를 모르니 출력괄과에 직접 레이블을 달아야한다. 
- 정확한 정수값을 사용한다는 것을 개발자가 보증해야한다.    
      --> 잘못된 값이 들어와도 프로그램은 수행됨 
#### 해결책! EnumMap!
열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체가 존재한다. 이것이 바로 EnumMap 이다

```java
Map<Plant.LifeCycle, Set<Plant>>  plantsByLifeCycle =  
        new EnumMap<>(Plant.LifeCycle.class);  
for (Plant.LifeCycle lc : Plant.LifeCycle.values())  
    plantsByLifeCycle.put(lc, new HashSet<>());  
for (Plant p :  garden)  
    plantsByLifeCycle.get(p.lifeCycle).add(p);  
System.out.println(plantsByLifeCycle);
```
- 더 짦고 명료하고 안전하고 성능도 동일.
	--> 내부에서 배열을 사용하기 때문. 
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄
- EnumMap 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다. 
#### 스트림 사용!
```java
System.out.println(Arrays.stream(garden)  
    .collect(groupingBy(p -> p.lifeCycle)));
```
#### 스트림 +EnumMap 사용
```java
System.out.println(Arrays.stream(garden)  
    .collect(groupingBy(p -> p.lifeCycle,  
 () -> new EnumMap<>(Plant.LifeCycle.class), toSet()));

```

EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다. 

#### 다른 예제 (상태전이)
####  EnumMap 사용X
```java
public enum Phase {  
    SOLID, LIQUID, GAS;  
  
 public enum Transition{  
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;  
  
 private static final Transition\[\]\[\] TRANSITIONS \= {  
                {null, MELT, SUBLIME},  
 {FREEZE, null, BOIL},  
 {DEPOSIT, CONDENSE, null}  
        };  
  
 public static Transition from(Phase from, Phase to ){  
            return TRANSITIONS\[from.ordinal()\]\[to.ordinal()\];  
 }  
    }  
}
```
#### EnumMap 사용
```java
public enum Phase {  
    SOLID, LIQUID, GAS;  
  
 	public enum Transition {  
    	MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),  
 		BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),  
 		SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);  
  
  
 		private final Phase from;  
 		private final Phase to;  
  
 		Transition(Phase from, Phase to) {  
         		   this.from \= from;  
 				   this.to \= to;  
 		}  
  
        private static final Map<Phase, Map<Phase,Transition>>  
             m \= Stream.of(values())
			 .collect(groupingBy(t -> t.from, 
			 					() -> new EnumMap<>(Phase.class),  
 								toMap(t -> t.to, t -> t, (x,y) ->  y, () -> new EnumMap<>(Phase.class))));  
  
 		public static Transition  from(Phase from, Phase to){  
           		 return m.get(from).get(to);  
 		}  
    }  
}

```
groupingBy 에서는 전이를 이전 상태를 기준으로 묶고,
toMap 에서는 이후 상태를 전이에 대응시키는 EnumMap을 생성 한다. 
(x,y) ->y 는 선언만하고 실제로 쓰이지는 않는데, 이는 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고 수집기들은 점층적 팩터리를 제공하기 때문이다.

### 결론 
배열의 인덱스를 얻기 위해 Ordinal을 쓰는 것은 일반적으로 좋지않으니, 대순EnumMap을 사용하라 
다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하라.