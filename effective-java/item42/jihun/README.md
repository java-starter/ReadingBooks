# 아이템 42. 익명 클래스보다는 람다는 사용하라

## 익명 클래스보다 람다를 사용해야 하는 이유
자질 구레한 코드들이 사라지고 어떤 동작을 하는지가 명확하게 드러난다.

```java
//익명 클래스
Collections.sort(words, new Comparator<String>() {
	public int compare(String1, String2) { 
		return Integer.compare(sl.1ength(), s2.1ength());
	}
}) ;

//람
Collections.sort(words,
		(s1, s2) -> Integer.compare(s1.length(), s2.length))
```

아이템 34에서 상수별 메서드 구현은 상수 마다 추상 메서드를 오버라이드 해서 재정의 해줘야 했다.
하지만 람다를 사용하면 다음과 같이 코드가 훨씬 깔끔해 진다.

```java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    public String getSymbol() {
        return symbol;
    }

    protected final String symbol;
    private final DoubleBinaryOperator op;

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}

public abstract class Application {

    public static void main(String[] args) {
        Operation plus = Operation.PLUS;
        System.out.println(plus.apply(2,3));
    }
}

```

## 람다와 익명 클래스 차이
문법의 차이도 있지만 this의 scope가 다르다. 람다에서 this는 바깥 인스턴스를 가리키지만, 익명 클래스에서 this는 익명 클래스의 인스턴스 자신을 가리킨다.  그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야한다.

anonymousTest 메서드는 익명 클래스 안에 있는 age 변수인 20을 return하고 lamdbaTest 메서드는 바깥 인스턴스인 Application에 있는 age 변수인 30을 return 한다.
```java
public class Application {

    public static void main(String[] args) {
        Application application = new Application();
        System.out.println(application.anonymousTest());
        System.out.println(application.lamdbaTest());
    }

    private static final int age = 30;

    public int anonymousTest() {
        Foo foo = new Foo() {
            int age = 20;

            @Override
            public int getAge() {
                return this.age;
            }
        };

        return foo.getAge();
    }

    public int lamdbaTest() {
        Foo foo = () -> {
            return this.age;
        };

        return foo.getAge();
    }
}

interface Foo {
    int getAge();
}

```


## 람다를 사용하지 말아야 할 때는 언제일까?
- 람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
- 직렬화 해야하는 경우 사용하지 말아야 한다. 람다도 익명 클래스 처럼 직렬화 형태가 구현별(VM)로 다를 수 있가 때문이다.


## 타입 추론에 관하여
람다 사용시 타입을 주로 생략하는데, 생략해도 문제가 없는 것은 컴파일러가 타입을 추론하는 데 필요한 타입 정보를 대부분 제네릭에서 얻기 때문이다.

실제 Collections의 sort 메서드를 보면 제네릭이 사용되고 있기 때문에 타입 생략이 가능한 것이다.
```java
public class Collections {
	public static <T> void sort(List<T> list, Comparator<? super T> c) {
    list.sort(c);
	}
}
```

## 정리
람다를 사용하면 익명클래스를 사용할 때 보다 코드가 줄어들고 의미도 훨씬 명확해 진다.
다만 람다식이 길어질 시 오히려 가독성을 해치므로 람다식이 길어진다면 익명클래스를 사용하는게 낫다.