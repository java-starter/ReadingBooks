# 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.

```java
public interface Figure {

}

public class Circle implements Figure {

}

public class Main {
    public static void main(String[] args) {
        //Circle 클래스는 Figure 인터페이스를 구현했기 때문에, Figure 타입으로 참조가 가능
        Figure circle = new Circle();
    }
}
```

## 상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예다.

상수 인터페이스란 메서드 없이 상수를 뜻하는 static final 필드로만 가득찬 인터페이스를 말한다.

상수 인터페이스가 안티패턴인 이유는, 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 클래스 내부 구현에 해당 되기 때문이다.
만약 상수 인터페이스 변경된다면, 해당 상수를 사용하고 있는 모든 클래스에 변경이 발생하기 때문에 안티 패턴이다.

## 상수 인터페이스를 해결하는 방법

### 1. 클래스에 추가

Integer 클래스의 MIN_VALUE, MAX_VALUE 처럼 자신과 연관되어 있는 상수는 자신의 클래스에 정의힌다.

```java
public final class Integer extends Number implements Comparable<Integer> {

    @Native public static final int   MIN_VALUE = 0x80000000;

    @Native public static final int   MAX_VALUE = 0x7fffffff;
```

### 2. 열거 타입

자신의 클래스에서만 사용하는 것이 아닌 많은 클래스에서 사용이 필요한 상수이며, 열거 타입으로 표현이 가능하면 열거 타입을 사용한다.

JPA에서 Enum의 값을 어떻게 처리할 지에 대한 것은 여러 Entity에서 사용가능하며, 열거 타입으로 표현이 가능하기 때문에 아래 처럼 사용되고 있다.

```java
public enum EnumType {
    ORDINAL,
    STRING;

    private EnumType() {
    }
}
```

### 3. 유틸리스 클래스 사용

열거 타입으로 표현하기 힘들다면 인스턴스화 할 수 없는 유틸리티 클래스를 사용하면 된다.

```java
package dev.jihun;

public class MathematicsNormalConstants {

    private MathematicsNormalConstants(){}

    //원주율
    public static final double CIRCUMFERENCE_RATE = 3.14_159_265_358_979_323_846_264_338_327_950_288;

    //네이피어 수
    public static final double NAPIER_COUNT = 2.71828_18284_59045_23536_02874_71352_66249;

    //황금비
    public static final double SIMILARITY_RATIO = 1.61803_39887_49894_84820_45868_34365_63811;
}
```

> Java 7 부터는 숫자 리터럴에 **_** 를 사용할 수 있다. 해당 값은 리터럴에 전혀 영향을 안주므로 가독성이 나쁠 때는 _를 사용해서 가독성을 높여주자.

## 정리

인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.
