[ITEM34] 상수 대신 열거 타입을 사용하라
---
-------

<h2>정수 열거 패턴</h2>  

>자바가 열거타입을 지원하기 전 사용하던 기법으로 타입안정석을 보장할수 없으면, 평범한 상수를 나열한 것에 불과해 단점이 많은 기법이다.

```java
public static final int APPLE_FUJI         = 0;
public static final int APPLE_PIPPIN       = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOd  = 2;
```
위와 같은 패턴의 단점   
- 문자열로 출력하기 다소 까다롭다. 
- 디버거로 살펴보면 단지 수자로만 표시 된다. 
- 열거 그룹에 성한 모든 상수를 한바퀴 순회하는 방법도 없다. 

위와 같은 패턴 이외에도 문자열 열거 패턴이 있지만 문자열 그대로를 하드코딩하게 만들기 때문에 더 나쁜 방법 으로 본다. 

<h2>열거타입?</h3>

>**일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입**

열거 타입은 위와 같은 패턴 (정수 열거 패턴, 문자열 열거 패턴)의 단점을 제거 해주는 동시에 여러가지 장점을 안견주는 대안을 제시햇다.

열거타입의 가장 단순한 형태   
```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

자바의 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 **public static final** 필드로 공개한다. 또 한 열거 타입은 밖에서 접근할 수 있느 생성자를 제공하지 않으므로 사실상 **final** 이다.**즉, 열거 타입은 싱글턴을 일반화한 형태라고 볼수 있다.**   


# 열거 타입은 컴파일타임 타입 안정성을 제공한다.
열거 타입을 건네 받는 메소드를 선언했다면, 건네 받는 메소드는 열거타입의 값중 하나임이 확실하다.  

# 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않앗도 된다. 

# 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.
```java
public enum Planet{
    MERCURY(3.302e+23, 2.439e6)
    VENUS  (4.869e+24, 6.052e6)
    ''''''
    URANUS (8.683e+23, 2.556e7)
    NEPTUNE(1.024e+26, 2.477e7)

    private final double mass; 
    private final double radius;
    private final double srfaceGravity;

    priavaet static final double G = 6.67300E-11;

    Planet(double mass, double radius){
        this.mass = mass;
        this.radius = radius;
        surfatceGravity = G * mass / (radius * radius);
    }
    public double mass() {return mass;}
    public double radius() {return radius;}
    public double surfaceGravity() {rturn surfaceGravity;}

    public double surfaceWeight(double mass){
        return mass * surfaceGravity; 
    }

}
```


열거 타입 상수 가가을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.       

아래는 위 열거 타입을 이용한 클라이언트 코드 
```java
public cass WeightTable{
    public static void main(String[] args){
        double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        for(Planet p : Planet.values()){
            System.out.printf("%s에서의 무게느 %f이다.%n", p, p.surfaceWeight(mass));
        }
    }
}
```

# 열거타입은 private으로, 혹은 package-private으로 선언하라. 

일반 클래스와 마찬가지로 그 기능을 긐라이언트에 노출해야 할 합당한 이유가 없다면 private또는 pacaage-private 으로 선언 한다. 

# 상수별로 다르게 동작하는 코르르 구현하는 더나은 수단을 제공한다. 

```java
public enum Operation{
    PLUS("+") 
        {public double apply(double x, double y){return x + y;}},
    MINUS("-")
        {public double apply(double x, double y){return x - y;}},
    TIMES("*")
        {public double apply(double x, double y)
        {return x * y;}},
    DIVIDE("/")
        {public double apply(double x, double y)
        {return x / y;}};
    
    private final String symbol;

    Operation(String symbol){this.symbol = symbol;}

    @Override public String toString {return symbol;}
    public abstract double apply(double x, double y);
}
```


# fromString() 메소드 구현
toString 이 반화하는 무자열을 해다 열거 타입 상수로 변환 하는 fromString 메소드, 다음 코드는 모든 열거 타입에서 사용할 수 있도록 구현한 fromString이다.

```java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String Symbol){
    return Optional.ofNullable(StringToEnum.get(symbol));
}
```

# 전략 열거 타입 패턴
새로운 상수를 추가할 때 전력을 선택하도록 하는것이다. 
```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    
    pirvate final PayType payType;
    PayrollDay(PayType payType){this.payType= payType;}
    
    int pat(int minutesWorked, int payRate){
        return payType.pay(minutesWorked, payRate);
    }

    enum PayType{
        WEEKDAY{
            int overtimePay(int minusWorked, int payRate){
                return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked -MIN_PER_SHIFT) * payRate / 2; 
            }
        }, 
        WEEKEND{
            int overtimePay(int minsWorked, int payRate){
                return minsWorked * payRate / 2; 
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60 

        int pay(int minsWorekd, int payRate){
            int basePat = minsWorked * patRate; 
            return basePay + overtimePay(minsWorkde,payRate)
        }
    }
}
```

# 열거 타입을 과연 언제 쓰란 말인가?!
- 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입ㅇ르 사용하자. 
- 열거타입에 정의된 상수 개수가 영원히 고정 불편일 필요는 없다.
- 