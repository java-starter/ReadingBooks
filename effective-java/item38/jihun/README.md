# 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## 1. 타입 안전 열거 패턴(typesafe enum pattern)이란?
타입 안전 열거 패턴이란 상수 열거 패턴의 문제를 보완하기 위한 패턴으로 Enum이 나오기 전까지 사용되었으며, 한정된 수의 값만 사용할 수 있는 pattern 이다.
아래 예제에서는 클래스 변수인 `terminal`, `process`, `decision`만 외부에서 참조 가능하다.
```java
public class DsymbolType {
    private final String type;

    private DsymbolType(String type) {
        this.type = type;
    }

    @Override
    public String toString() {
        return "DsymbolType{" +
                "type='" + type + '\'' +
                '}';
    }
    
    public static final DsymbolType terminal = new DsymbolType("Terminal");
    public static final DsymbolType process = new DsymbolType("Process");
    public static final DsymbolType decision = new DsymbolType("Decision");
}
```

> 참고 : [Danguria's stables :: Type safe enumeration pattern을 적용하고 있습니다.](https://danguria.tistory.com/57)

## 2. 열거 타입 확장이 불가능한 이유
1. 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대가 성립이 되지 않는다.
2. 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법도 마땅치 않다.
3. 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 더 복잡해 진다.

하지만 확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있다. 바로 **연산 코드(operation code 혹은 opcode)** 다. 연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻하며, 이따금 API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 열어줘야 할 떄가 있다.

## 3. 인터페이스를 사용한 열거 타입 확장
열거 타입은 인터페이스를 구현할 수 있기 때문에, 인터페이스에 상수별 메서드를 정의 해놓고 열거 타입에서 해당 인터페이스를 구현하면 열거 타입이 해당 인터페이스의 표준 구현체 역할을 한다.

아이템 34. int 상수 대신 열거 타입을 사용하라에서 상수별 메서드 구현의 예제는 다음과 같다.
```java
public enum Operation { 

PLUS{
    public double apply(double x, double y){return x+y;}
}, 
MINUS{
    public double apply(double x, double y){return x-y;}
}, 
TIMES{
    public double apply(double x, double y){return x*y;}
}, 
DIVIDE{
    public double apply(double x, double y){return x/y;}
}; 

public abstract double apply(double x, double y); }
```

상수별 메서드 구현에서 추상 메서드는 Operation 인터페이스에 정의되어 있으며, 열거 타입에서는 Operation 인터페이스를 구현함으로써 Operation 인터페이스의 구현체 역할 한다.
```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return "BasicOperation{" +
                "symbol='" + symbol + '\'' +
                '}';
    }
}
```

만약 연산 코드가 추가된다면 Operation 인터페이스를 구현해 주기만 하면 된다.
```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        @Override
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        @Override
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return "BasicOperation{" +
                "symbol='" + symbol + '\'' +
                '}';
    }
}
```

## 4. 확장된 열거 타입 원소 모두 사용하는 방법
### 4-1. Class 객체 사용
test 메서드의 첫 번째 파라미터로 class 리터럴을 받아서 `getEnumConstants()` 메서드를 사용하면 해당 열거타입의 원소를 모두 가져올 수 있다.

`<T extends Enum<T> & Operation>` 는 T가 Enum의 자손이면서, Operation 인터페이스를 구현한 타입이여야 한다는 뜻이다.
```java
public class Application {
    public static void main(String[] args) {
        test(BasicOperation.class, 3, 4);
        test(ExtendedOperation.class, 3, 4);
    }

    private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
        for (Operation op : opEnumType.getEnumConstants())
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

> 인터페이스 타입도 extends를 사용한다! 위처럼 여러개를 사용할 때 &를 사용하는 것이다!

### 4-2. 한정적 와일드카드 타입인 Collection<? extends Operation> 사용
위의 `test` 메서드는 특정 타입만 사용가능 하지만 아래 `test` 메서드는 Operation 인터페이스만 구현하면 되기 때문에 위의 `test` 메서드에 비해 훨씬 유연해 졌다.
```java
public class Application {
    public static void main(String[] args) {
        test(Arrays.asList(ExtendedOperation.values()), 3, 4);
    }

    private static void test(Collection<? extends Operation> opSet, double x, double y) {
        for (Operation op : opSet)
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

## 5. 인터페이스를 사용한 열거 타입 확장 문제
- 열거 타입끼리 구현은 상속이 불가하기 때문에 중복코드가 발생한다.
    - 아무 상태에도 의존하지 않는 경우 default method를 사용해서 해결할 수 있다.
    - 상태에 의존하는 경우 별도의 도우미 클래스나 정적 도우미 메서드로 분리해서 코드 중복을 회피할 수 있다.


## 정리
열거타입 자체는 확장이 불가하지만 인터페이스를 구현함으로써 열거타입 확장과 같은 효과를 낼 수 있다.
