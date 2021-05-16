# 1. 비트 필드(bit field)란?

비트들을 멤버로 가지는 클래스이다.

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; //1
    public static final int STYLE_ITALIC = 1 << 1; //2
    public static final int STYLE_UNDERLINE = 1 << 2; //3
    public static final int STYLE_STRIKETHROUGH = 1 << 3; //4

    public void applyStyles(int style) {
       
    }
}
```

# 2. 비트 필드(bit field) 장점

비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

# 3. 비트 필드(bit field) 단점

1. 비트 필드값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때 보다 해석하기기 훨씬 어렵다.
2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.
3. 최대 몆 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입(보통은 int나 long)를 선택해야 한다.

위와 같은 단점들로 인해 비트 필드를 사용하기 보다 java.util 패키지에 있는 EnumSet 클래스를 사용하는 것이 좋다.

# 4. EnumSet이란?

열거형 타입에 사용할 수 있도록 특별 하게 Set 구현한 것이 EnumSet 이다.

EnumSet의 내부는 비트 백터로 구현되어 있으며, 원소가 총 64개 이하라면, EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.

```java
public class Text {
    public enum Style {BOLE, ITALIC, UNDERLINE, STRIKETHROUGH}

    public void applyStyles(Set<Style> styles) {
        
    }
}

public class Application {
    public static void main(String[] args) {
      
    Text text = new Text();
    text.applyStyles(EnumSet.of(Text.Style.BOLE, Text.Style.ITALIC));
    }
}
```

> HashSet은 배열과 해시코드와 LinkedList를 사용하지만, EnumSet은 비트연산을 사용하기 때문에 속도가 훨씬 빠르다.

# 정리

열거할 수 있는 타입을 한데 모아 집합 형태로 사용시 비트 필드가 아닌 EnumSet을 사용하자.

EnumSet은 열거 타입의 장점과 비트 필드 수준의 성능도 제공하기 때문이다.