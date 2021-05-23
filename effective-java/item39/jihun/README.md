# 아이템 39. 명명 패턴보다 애너테이션을 사용하라

## 1. 명명패턴이란?
도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에 딱 구분되는 이름을 붙이는 것이다. 예를들어 Junit 3까지는 테스트 메서드의 이름이 test로 시작해야 테스트 메서드가 실행되는 명명 패턴을 사용하였다.

### 1-1. 명명패턴의 문제점
1. 오타가 나면 안된다.
    - Junit3에서 test 명명 패턴에 오타가 나면 테스트 메서드가 실행되지 않는다.
2.  올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.
    - 예를들어 명명 패턴이 메서드에 적용되어야 하나 클래스에 사용시 테스트 메서드가 실행되지 않는다.
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

애너테이션을 사용하면 위와 같은 명명 패턴의 문제를 모두 해결할 수 있다.

## 2. `@Test` 애너테이션을 만들어보자
아래 애너테이션은 테스트 메서드임을 선언하는 애너테이션으로 매개변수 없는 정적 메서드 전용이다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

- `@Test` 애너테이션 타입 선언 자체에도 애너테이션이 달려있는데, 이러한 애너테이션을 메타 애너테이션(meta-annotation)이라 한다.
- `@Test` 처럼 아무 매개변수 없이 단순히 대상에 마킹(markin)을 하는 애노테이션을 마커(marker) 애노테이션이라 한다.
    - 여기서 매개변수란 해당 어노테이션의 매개변수를 의미한다.

다음 코드는 위에서 생성한 마커 애너테이션을 사용한 코드이다.
```java
public class Sample {

    //통과
    @Test
    public static void m1() {};

    //실행 안됨
    public static void m2() {};

    //실패
    @Test
    public static void m3() {
        throw new RuntimeException("실패");
    }

    //실행 안됨
    public static void m4() {}

    //@Test 잘못사용
    //정적 메서드가 아님
    @Test
    public void m5() {}

    //실행 안됨
    public static void m6() {}

    //실패
    @Test
    public static void m7() {
        throw new RuntimeException("실패");
    }

    //실행 안됨
    public static void m8() {
    }
}
```

위의 `Sample` 클래스에서 사용된 `@Test` 마커 애너테이션을 처리하는 테스트 러너이다.
```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("Sample");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                }catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                }catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

> 테스트 메서드가 예외를 던지면 리플렉션 매커니즘이 InvocationTargetException으로 감싸서 다시 던진다. 하지만 InvocationTargetException 이외의 예외가 발생한다면 @Test를 잘못사용한 것이다.

## 3. `@ExceptionTest` 애너테이션을 만들어보자
아래 애너테이션은 명시한 예외를 던져야 성공하는 테스트 메서드용 애너테이션이다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

```

다음 코드는 위에서 생성한  `@ExceptionTest`  애너테이션을 사용한 코드이다.
```java
public class Sample2 {
    //성공
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {
        int i = 0;
        i = i / i;
    }

    //실패(다른 예외 발생)
    public static void m2() {
        int[] a = new int[0];
        int i = a[1];
    }

    //실패(예외 발생 안함)
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }
}
```

위의 `Sample2` 클래스에서 사용된 `@ExceptionTest`  애너테이션을 처리하는 테스트 러너이다.
```java
public class RunTests2 {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("Sample2");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                }catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    //애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인
                    Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
                    }
                }catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

> 해당 예외의 클래스 파일이 컴파일타임에는 존재했으나 런타임에 존재하지 않을 시 테스트 러너가 TypeNotPresentException을 던진다.

## 4. 여러 개의 값을 받는 애너테이션
배열 매개변수를 사용해서 여러 개의 값을 받을 수도 있다.

위의 `@ExceptionTest` 에서 여러 예외를 받고 싶다면 다음 코드처럼 배열 매개변수를 사용하면 된다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}
```

어노테이션의 매개변수가 배열도 변경되었기 때문에  `@ExceptionTest`  애너테이션을 처리하는 테스트 러너도 약간 변경되어야 한다.
```java
public class RunTests2 {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("Sample2");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                }catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    Class<? extends Throwable> excTypes[] = m.getAnnotation(ExceptionTest.class).value();
                    for (Class<? extends Throwable> excType : excTypes) {
                        if (excType.isInstance(exc)) {
                            passed++;
                            break;
                        }
                        if (passed == oldPassed) {
                            System.out.printf("테스트 %s 실패: %s %n", m, exc);
                        }
                    }
                }catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

## 5. 반복 가능 애너테이션 사용하기
위의 예제처럼 애노테이션 매개변수에 배열을 사용할 수도 있지만 Java 8부터는 `@Repeatable`을 사용해서 하나의 프로그램 요소에 애노테이션을 여러번 다는 방법으로도 처리할 수 있다.

### 5-1. `@Repeatable` 사용시 주의할 점
1. @Repeatable을 단 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더 정의한다.컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 하며, 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```
2. @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

반복 가능 애너테이션을 다음과 같이 적용할 수 있다.
```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    @ExceptionTest(IndexOutOfBoundsException.class)
    public static void m1() {
    }
```

반복 가능 애너테이션 처리시 여러 개 달았을 때와 하나만 달았을 때를 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다.
isAnnotationPresent 메서드는 둘을 명확히 구분하기 때문에 따로 확인해야하며, getAnnotationByType 메서드는 둘을 구분하지 않기 때문에 따로 확인하지 않아도 된다.
```java
public class RunTests2 {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("Sample2");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                }catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                        if (passed == oldPassed) {
                            System.out.printf("테스트 %s 실패: %s %n", m, exc);
                        }
                    }
                }catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}


```

## 정리
- 애너테이션으로 할 수 있는 일을 명명패턴으로 처리할 이유는 없다.
