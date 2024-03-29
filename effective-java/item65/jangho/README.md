# [이펙티브 자바] Item65 - 리플렉션보다는 인터페이스를 사용하라

# 리플렉션 기능

리플렉션 기능을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다. 

Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드 인스턴스를 가져올 수 있고, 해당 인스턴스들로부터 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다.

나아가 각 인스턴스를 이용해 연결된 실제 생성자, 메서드, 필드를 조작할 수 있다. 이는 클래스의 인스턴스를 생성하거나, 메서드를 호출하고, 필드에 접근할 수 있음을 나타낸다.

리플렉션을 이용하면 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다.

# 리플렉션의 단점

### 1. 컴파일타임 타입 검사가 주는 이점을 누릴 수 없다.

타입 검사나 예외 검사의 이점을 누릴 수 없다. 리플렉션 기능을 이용해 존재하지 않거나 접근할 수 없는 메서드를 호출하면 런타임 오류가 발생한다.

### 2. 코드가 지저분하고 장황해진다.

### 3. 성능이 떨어진다.

리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다.

코드 분석 도구나 의존관계 주입 프레임워크처럼 리플렉션을 써야 하는 복잡한 애플리케이션들도 리플렉션 사용을 점차 줄이고 있다. 단점이 명확하기 때문이다.

# 리플렉션은 아주 제한된 형태로만 사용하자

## 리플렉션으로 생성하고 인터페이스로 참조해 활용

컴파일 타임에 이용할 수 없는 클래스를 사용해야 하는 프로그램은 컴파일 타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수 있다. 이런 경우, **리플렉션은 인스턴스 생성에만 쓰고, 만들어진 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.**

## 예제

```java
// 리플렉션으로 생성하고 인터페이스로 참조해 활용한다.
public static void main(String[] args) {
    
    // 클래스 이름을 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) Class.forName(args[0]); //비검사 형변환
    } catch (ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }
    
    // 생성자를 얻는다.
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }
     
    //집합의 인스턴스를 만든다.
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스턴스화할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Set을 구현하지 않은 클래스입니다.");
    }
    
    //생성한 집합을 사용한다.
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
}
```

> 이 예제는 간단하지만 강력하다. 손쉽게 Set 규약을 잘 지키는지 검사해볼 수 있고, 비슷하게 제네릭 집합 성능 분석 도구로 사용할 수 도 있다. 대부분의 경우 리플렉션 기능은 이 정도만 사용해도 충분하다.

## 리플렉션의 단점

위 예제는 리플렉션의 단점 두 가지를 보여준다.

### 1. 런타임에 총 6개의 예외를 던질 수 있다.

총 6개의 예외 모두 리플렉션 없이 생성했다면 컴파일타임에 잡아낼 수 있는 예외들이다.

### 2. 클래스 이름으로 인스턴스 생성을 하기위해 25줄의 코드를 작성했다.

리플렉션이 아니라면 생성자 호출 1줄로 끝났을 일이지만 리플렉션을 사용함으로써 25줄의 코드를 작성했다.

# 리플렉션의 활용도

리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다. 이 기법은 버전이 여러개 존재하는 외부 패키지를 다룰 때 유용하게 사용할 수 있다.

주로 가장 오래된 버전만을 지원하도록 컴파일한 후, 이후 버전의 클래스와 메서드 등은 리플렉션으로 접근하는 방식이다.

이렇게 하려면 클래스나 메서드가 런타임에 존재하지 않을 수 있다는 사실을 반드시 감안해야 한다. 즉, 같은 목적을 이룰 수 있는 대체 수단을 이용하거나 기능을 줄요 동작하는 등 적절한 조취를 취해야 한다.

# 핵심 정리

- 리플렉션은 복잡한 특수 시스템을 개발할 때 필요한 강력한 기능이지만, 단점도 많다.
- 컴파일 타임에는 알 수 없는 클래스를 사용하는 프로그램을 작성한다면 리플렉션을 사용해야 한다.
- 이때 되도록 리플렉션은 객체 생성에만 사용하고, 생성한 객체를 사용할 때는 적절한 인터페이스나 상위 클래스로 형변환해 사용하자.