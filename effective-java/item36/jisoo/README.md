---
title:  
date: [[TIL/2021-05-12]] 1:04 PM 
tags: #effectivjava #java
---
# [Item36]  비트 필드 대신 EnumSet을 사용하라 
## 비트필드?
비트별 OR를 사용해 여러 상수를 모아 하나의 집하읍으로 만든 것.
비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합연선을 효율적으로 수행할 수 있다. 하지만 비트필드는 열거 상수의 단점을 그대로 지닌다. 그외의 단점으로는 비트 필드의 값이 그대로 출력되면 해석하기가 어렵다. 모든 원소를 순회하기도 까다롭다. API 작성시 미리 예측하여 적절한 타입을 선택해야 한다. 
## EnumSet?
비트 필디의 대안. Set 인터페이스를 완벽히 구현하며, 타입이 안전자고, 다른 어떤 Set구현체와도 함께 사용할 수 있다. 하지만 EnumSet의 내부는 비트 벡터로 구현 되었다. 원소가 총 64개 이하라면, EnumSet 전체를 long변소 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다. 

## 비트필드 To EnumSet 
#### 비트필드 
```java
public class Text{
	public static final int STYLE_BOLD = 1 << 0;
	public static final int STYLE_ITALIC = 1 << 1;
	public static final int STYLE_UNDERLINE = 1 << 2;
	public static final int STYLE_STRIKETHROUGH = 1 << 3;
	
	public void applySytles(int styles){...}
}
//[Client CODE]
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```
#### EnumSet
```java
public class Text{
	 public enum Style { BOLD , ITALIC, UNDERLINE, STRIKETHROUGH }
	 
	 public void applyStyles(Set<Style> styles){...}
}
//[Client CODE]
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

applyStyle 에서 EnumSet\<Style>이 아닌 set\<Style>을 받은 이유는 이왕이면 인터페이스로 받는 게 일번적으로 좋은 습관 [[item64]] 이렇게 하면 좀 특이한 클라이언트가 다른 Set 구현체를 넘기더라도 처리할 수 있다.  