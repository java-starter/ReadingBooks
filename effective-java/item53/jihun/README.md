# 아이템 53. 가변인수는 신중히 사용하라

## 가변인자(varargs)란?

JDK 1.5에 도입된 것으로 메서드 매개변수의 개수를 동적으로 지정할 수 있는 기능으로 `타입... 변수명` 으로 선언한다.

## 가변인자(varargs) 동작방식

가변인자를 가진 메서드를 호출하면, 인수의 개수와 길이가 같은 배열을 만들고 인수들은 이 배열에 저장하여 가변인수 메서드에 건네준다.

## 가변인자(varargs) 사용시 유의사항

### 인수가 1개 이상이어야 하는 가변인자 메서드

min 메서드는 최소 1개 이상의 인수가 필요한데 가변인자(varargs)는 인수를 0개도 가능하기 때문에 런타임 예외가 발생할 수 있다.

```java
public class Application {
    public static void main(String[] args) {
        min();
    }

    static int min(int... args) {
        if (args.length == 0)
            throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
        int min = args[0];
        for (int i = 1; i < args.length; i++)
            if (args[i] < min)
                min = args[i];
        return min;
    }
}
```

위의 문제를 해결하기 위해서는 매개변수를 2개 받도록 하면된다. 첫 번째로는 평범한 매개변수를 받고, 두 번째로는 가변인수를 받으면 된다.

```java
public class Application {
    public static void main(String[] args) {
        min(1);
    }

    static int min(int firstArg, int... remainingArgs) {
        int min = firstArg;
        for (int arg : remainingArgs)
            if (arg < min)
                min = arg;
        return min;
    }
}
```

> 가변인자 외에도 매개변수가 더 있다면, 가변인자를 매개변수 중에서 제일 마지막에 선언해야 한다. 그렇지 않으면 컴파일 에러가 발생한다. 가변인자인지 아닌지를 구별할 방법이 없기 때문에 허용하지 않는 것이다. 
자바의 정석 3판 p.287

## 가변인수를 사용할 때 성능 최적화 하기

가변인수가 사용된 메서드를 호출할 때 마다 가변인수를 저장하기 위한 배열이 생성되고 초기화되기 때문에 성능이슈가 발생할 수 있다.

이러한 성능 이슈를 해결하면서 가변인수를 사용할 수 있는 패턴이 있다. 자주 사용되는 메서드를 다중정의하면서 마지막 다중정의 메서드에 가변인수를 사용하는 것이다.

실제 메서드 호출시 가변인수 메서드의 호출수는 줄어드므로 성능상 이점을 가질 수 있다.

```java
public int min(int arg1) {}
public int min(int arg1, int arg2) {}
public int min(int arg1, int arg2, int arg3) {}
public int min(int arg1, int arg2, int arg3, int... remainingArgs) {}
```

JDK 9부터 List 인터페이스의 of 메서드가 위와 같이 구현되어 있다.

## 정리

가변인수를 사용하는 메서드 정의시 필수 매개변수는 가변인수 앞에 두고, 성능상 이유가 발생할 수 있으므로 유의하자