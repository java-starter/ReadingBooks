# [ITEM39] 명명 패턴보다 애너티에션을 사용하라. 

전톡적의로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다. 
#### 명명 패턴
junit은 버전 3까지 테스트 메서드 이름을 test로 시작 하게끔 했다. 
다음과 같은 담점이 존재
- 오타가 나면안된다,
- 올바른 프로그램 요소에만 사용되라라 보증할 방법이 없다. 
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.
  --> 특정 예외를 던져야만 성공하는 테스트가 있다고 가정, 예외이름을 테스트 메서드 이름에 덧 붙히는 방법도 있지만, 보기도 나쁘고 깨지기도 쉽다[[Item62]], 컴파일러는 메서드 이름에 덧붙인 문자열이 예외를 가리키는지 알 도리가 없다. 
  #### 애너테이션
  애너테이션은 이 모든 문제를 해결해주는 멋진 개념. 
  
  #### 애너테이션 구현
  Test라는 이름의 애너테이션을 정의, 자동으로 수행되는 간단한 테스트용 애터테이션으로, 예외가 발생한다면 해당 테스트를 실패로 처리한다. 
  
```java
/**
*테스트 메서드임을 선언하는 애너테이션이다.
*매개변수 없는 정적 메서드 전용이다.
*/
@Retention(RetentionPolicy.RUNTIME)  // 런타임에도 유지되어야 한다.
@Target(ElementType.METHOD)  // 반드시 메서드 선언에서만 사용돼야 한다. 
public @interface Test {  
}
```
@Retentsion @Traget 같은 애너테이션 선언에 다는 에네터이션을 메타애너테이션이라고한다. 
주석에 쓰여 있는 제약을 컴파일러가 강제할 수 있으면 좋겠지만, 그렇게 하라면 적절한 애너테이션 처리기를 직접 구현해야 한다. 관련 방법은 [[javax.annotaion.processing API문서 참고 ]]
```java
public class Sample {  
    @Test public static void m1(){}  //성공  
 public static void m2(){}  
    @Test public static void m3(){   //실패  
 throw  new RuntimeException("실패");  
 }  
    public static void m4(){}  
    @Test public void m5(){}  //잘못 사용한  예  
 public static void m6(){}  
    @Test public static void m7(){ // 실패  
 throw new RuntimeException("실패");  
 }  
    public static void m8(){}  
}
```

@Test 어노테이션이 Sample클래스의 의미에 직접적인 영향을 주지는 않는다. 
즉, 대상코드의 의미는 그대로 둔채 그 애너테이션에 관심 있느 도구에서 특별한 처리를 할 기회를 준다. 
```java
public class RunTests {  
    public static void main(String\[\] args) throws Exception{  
        int tests = 0;  
 int passed = 0;  
  
 Class<?> testClass = Class.forName("effectivejava.item39.Sample");  
 for (Method m : testClass.getDeclaredMethods()){  
            if(m.isAnnotationPresent(Test.class)){  
                tests++;  
 try{  
                    m.invoke(null);  
 passed++;  
 }catch (InvocationTargetException wrappedExc){  
                    Throwable exc = wrappedExc.getCause();  
 System.out.println(m + "실패 : "\+ exc);  
 }catch (Exception exc){  
                    System.out.println(exc);  
 System.out.println("잘못 사용한 예 @Test "\+ m );  
 }  
            }  
        }  
        System.out.printf("성공: %d, 실패: %d%n", passed, tests- passed);  
 }  
}
```

테스트 메서드가 예외를 던지면 리플레션 메커니즘이 InvocationTargetException으로 감싸서 다시 던진다. 그래서 이프로그램은 InvocationTargetException을 잡아 원래 예외에 담긴 실패 정보를 출력한다. InvocationTargetException외의 예외가 발생한다면 어노티에션을 잘못 사용했다는 뜻이다. [[reflaction 정리 ]]

```java
/**  
 * 명시한 예외를 던져야만 성공하는 테스트 메스도용 애너테이션  
 */  
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.METHOD)  
public @interface ExceptionTest {  
    Class<? extends Throwable> value();  
}
```
이 애너테이션의 매개변수 타입은 Class\<? extends Throwable\> 이다. 
즉 Throwable을 확장한 클래스의 Class객체 라는 뜻이다. 따라서 모든 예외 타입을 다 수용한다. [[item33]]의 또다른 활용 사례 

```java
public class Sample2 {  
    @ExceptionTest(ArithmeticException.class)  
    public static void m1(){ //성공  
 int i =0;   
i = i / i;  
 }  
      
    @ExceptionTest(ArithmeticException.class)  
    public static void m2(){  //실패 다른예외 발생  
 int\[] a = new int[0];  
 int i = a[1];  
 }  
      
    @ExceptionTest(ArithmeticException.class)  
    public static void m3(){ //실패 예외가 발생하지않음   
          
}  
}
```

```java
public static void main(String\[\] args) throws ClassNotFoundException {  
    int tests = 0;  
 int passed = 0;  
  
 Class<?> testClass = Class.forName("effectivejava.item39.Sample2");  
 for (Method m : testClass.getDeclaredMethods()){  
        if(m.isAnnotationPresent(ExceptionTest.class)){  
            tests++;  
 try{  
                m.invoke(null);  
 System.out.printf("테스트 %s 실패: 예외를 던지지 않음 %n", m);  
 }catch (InvocationTargetException wrappedExc){  
                Throwable exc = wrappedExc.getCause();  
 Class<? extends Throwable> excType =  
                        m.getAnnotation(ExceptionTest.class).value();  
 if (excType.isInstance(exc)){  
                    passed++;  
 }else{  
                    System.out.printf("테스트 %s 실패 : 기대한 예외 %s, 발생한 예외 %s%n",m, excType.getName(), exc);  
 }  
            }catch (Exception exc){  
                System.out.println("잘못 사용한 예 @ExceptionTest "\+ m );  
 }  
        }  
    }  
    System.out.printf("성공: %d, 실패: %d%n", passed, tests- passed);  
}
```
올바른 예외를 던지는 지 확인하는 코드가추가 되었음 

#### 배열 매개변수를 받는 애너테이션 타입
```java
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.METHOD)  
public @interface ExceptionTest {  
    Class<? extends Throwable>\[\] value();  
}
```

```java
@ExceptionTest({IndexOutOfBoundsException.class,  
 NullPointerException.class})  
public static void doublyBad(){  
    List<String> list = new ArrayList<>();  
 // IndexOutOfBoundsException이나  
 // NullPointException 을던지는 메서드  
 list.add(5, null);  
}
```

```java
public static void main(String\[\] args) throws ClassNotFoundException {  
    int tests = 0;  
 int passed = 0;  
  
 Class<?> testClass = Class.forName("effectivejava.item39.Sample2");  
 for (Method m : testClass.getDeclaredMethods()) {  
        if (m.isAnnotationPresent(ExceptionTest.class)) {  
            tests++;  
 try {  
                m.invoke(null);  
 System.out.printf("테스트 %s 실패: 예외를 던지지 않음 %n", m);  
 } catch (Throwable wrappedExc) {  
                Throwable exc = wrappedExc.getCause();  
 int oldPasse = passed;  
 Class<? extends Throwable>\[\] execTypes =  
                        m.getAnnotation(ExceptionTest.class).value();  
 for (Class<? extends Throwable> excType : execTypes) {  
                    if (excType.isInstance(exc)) {  
                        passed++;  
 break; }  
                }  
                if (passed == oldPasse) {  
                    System.out.printf("테스트 %s 실패: %s %n", m, exc);  
 }  
            }  
        }  
    }  
}
```

#### @Repeatable
java8 에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다. 
그것이 바로 @Repeatable이다.
주의할점 
- @Repeatable을 단 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더정의하고, @Repeatable에 이 컨테이너 애너테이션의 class객체를 매개변수로 전달 해야한다. 
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의 해야한다. 
- 컨테이너 애너테이션 타입에는 적잘한 보존정책(@Retention)과 적용대상(@Target)을 명시해야한다. 

```java
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.METHOD)  
@Repeatable(ExceptionTestContainer.class)  
public @interface ExceptionTest {  
    Class<? extends Throwable>\[\] value();  
}
```
```java
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.METHOD)  
public @interface ExceptionTestContainer {  
    ExceptionTest\[\] value();  
}
```
```java
@ExceptionTest(IndexOutOfBoundsException.class)  
@ExceptionTest(NullPointerException.class)  
public static void doublyBad(){  
    List<String> list = new ArrayList<>();  
 // IndexOutOfBoundsException이나  
 // NullPointException 을던지는 메서드  
 list.add(5, null);  
}
```
주의사항
여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다. getAnotationByType 메서드는 이 둘을 구분하지 않아서 반보 가능 애터테이션과 그 컨테이너 애너테이션을 모두 가져오지만, isAnotaionPresent 메서드는 둘을 명확히 구분한다.

```java
public static void main(String\[\] args) throws ClassNotFoundException {  
    int tests = 0;  
 int passed = 0;  
  
 Class<?> testClass = Class.forName("effectivejava.item39.Sample2");  
 for (Method m : testClass.getDeclaredMethods()) {  
        if (m.isAnnotationPresent(ExceptionTest.class)) {  
            tests++;  
 try {  
                m.invoke(null);  
 System.out.printf("테스트 %s 실패: 예외를 던지지 않음 %n", m);  
 } catch (Throwable wrappedExc) {  
                Throwable exc = wrappedExc.getCause();  
 int oldPasse = passed;  
 Class<? extends Throwable>\[\] execTypes =  
                        m.getAnnotation(ExceptionTest.class).value();  
 for (Class<? extends Throwable> excType : execTypes) {  
                    if (excType.isInstance(exc)) {  
                        passed++;  
 break; }  
                }  
                if (passed == oldPasse) {  
                    System.out.printf("테스트 %s 실패: %s %n", m, exc);  
 }  
            }  
        }  
    }  
}
```
반복 가능 애너테이션을 사용해 하나의 프로그램 요소에 같은 애너테이션을 여러 번 달 때의 코드 가독성을 높여보았따. 

#### 결론
자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야한다[[Item40]][[Item27]]