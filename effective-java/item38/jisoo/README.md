# 확장할 수 있는 열거타입이 필요하면 인터페이스를 사용하라
### 확장 할 수 있는 열거타입?!
사실 대부분의 상황에서 열거타입을 확장하는건 좋지 않은 생각이다. 하지만  쓰임이 최소한 하나는 있다. 바로 연산코드 이다.
### 연산코드(operation code, opcode)
연산코드의 각원소는 특정 긱꼐가 수행하는 연산을 뜻한다.[[Item34]]의 Operation타입도 그중 하나로 간단한 계산기의 연상 기능을 의미 했다.
API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가 할 수 있도록 열어줘야 할 때가 있다. 
### 열거타입으로 opcode 확장하기
연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이슬 구현하게 하면된다. 
```java
public interface Operation {  
    double apply(double x, double y);  
}
```
```java
public enum BasicOperation implements Operation{  
    PLUS("+"){  
        public double apply(double x, double y){return x + y; }  
    },  
 MINUS("-"){  
        public double apply(double x, double y){return x - y; }  
    },  
 TIMES("*"){  
        public double apply(double x, double y){return x * y; }  
    },  
 DIVIDE("/"){  
        public double apply(double x, double y){return x / y; }  
    };  
 private final String symbol;  
 BasicOperation(String symbol){  
        this.symbol = symbol;  
 }  
      
    @Override public String toString(){  
        return symbol;  
 }  
}
```

입인 BaiscOperation 은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연샅 타입으로 사용하면 된다. 이렇게 하면 Operation을 구현한 또다른 열거 타입을 정의해 기분 타입인 BasicOperation을 대체 할 수 있다. 

```java
public enum ExtendedOperation implements Operation{  
    EXP("^"){  
        @Override  
 public double apply(double x, double y) { return Math.pow(x, y); }  
    },  
 REMINDER("%"){  
        @Override  
 public double apply(double x, double y) { return x % y; }  
    };  
  
 private final String symbol;  
  
 ExtendedOperation(String symbol) {  
        this.symbol = symbol;  
 }  
  
    @Override  
 public String toString() {  
        return symbol;  
 }  
}
```

Apply 가 Operation 에 선언 되어 있으니 열거 타입에 따로 추상 메서드로 선언하지도 않아도 되며, 새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다. 
```java
public static void main(String[] args) {  
    double x = Double.parseDouble(args[0]);  
 double y = Double.parseDouble(args[1]);  
 test(ExtendedOperation.class, x,y);  
}  
private static <T extends Enum<T> & Operation> void test(  
        Class<T> opEnumType, double x, double y){  
    for (Operation op : opEnumType.getEnumConstants())  
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));  
}
```

test메서드에 ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다. 여기서 class 리터럴은 한정적 타입 토큰[[item33]]역활을 한다. 
(<T extends Enum\<T\> \& Operation>) 은 Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻이다. 

```java
private static void test2(Collection<? extends Operation> opSet,double x, double y){  
    for (Operation op : opSet){  
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));  
 }  
}
```
Class 객체 대신 한정적 와일드 카드 타입인 Collection\<? extends Operation\>을 넘기는 방법이다. 
이코드는 그나마 덜 복잡하고 test코드가 살짝 더유연해졋다. 반면 EnumSet과 EnumMap을 사용하지 못한다. 

### 단점
열거 타입끼리 구현을 상속할 수 없다는 점이다. 아무 상태에도 의존하지 않는 경우 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다. 중복코드가 많이 생기게 되는데 공유하는 기능이 많다면 그부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨수 있을 것이다.
