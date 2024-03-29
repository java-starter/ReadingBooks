# [이펙티브 자바] Item41- 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

아무 메서드도 담고 있지 않고 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스를 **마커 인터페이스**라 한다. 

대표적으로 Serializeable 인터페이스를 예로 들 수 있다. Serializeable은 자신을 구현한 클래스의 인스턴스는 직렬화할 수 있음을 알려주는 역할을 한다.

마커 애너테이션이 등장하면서 마커 인터페이스가 더 이상 쓸모없게 느껴질 수 있겠지만, 사실 마커 인터페이스는 두 가지 측면에서 마커 애너테이션보다 더 나은 점이 있다.

# 마커 애너테이션 VS 마커 인터페이스

## 마커 인터페이스가 가지는 강점

### 1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스를 구분하는 타입으로 사용할 수 있다.

**마커 애너테이션과**는 다르게 **마커 인터페이스**는 타입이기 때문에 마커 애너테이션을 사용했다면 런타임에야 발견될 오류를 컴파일 타임에 잡아낼 수 있다.

예를 들어 ObjectOutputStream,writeObject 메서드는 인수로 받은 객체가 Serializable을 구현했을 거라 가정한다. 그런데 이 메서드는 Object 객체를 받도록 설계되어 있다. 따라서 Serializable을 구현하지 않은, 즉 직렬화할 수 없는 객체를 넘기면 런타임에야 문제를 확인할 수 있게된다. 

### 2. 적용 대상을 보다 정밀하게 지정할 수 있다.

적용 대상(@Target)을 타입으로 주었을 경우(ElementType.Type)에는 클래스, 인터페이스, 열거 타입, 애너테이션등 모든 타입에 애너테이션을 달 수 있다. 더 세밀하게 애너테이션을 달 수 없다는 뜻이다. 하지만 **마커 인터페이스는 특정 구현 클래스에만 적용하는 것이 가능하다.**

## 마커 애너테이션이 가지는 강점

### 1. 거대한 애너테이션 시스템의 지원을 받을 수 있다.

애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 사용하는 쪽이 일관성을 지키는 데 유리하다.

# 사용 기준?

그렇다면 언제 마커 애너테이션을 사용해야하고, 또 어떤 때 마커 인터페이스를 사용해야 하는걸까? 

**클래스와 인터페이스 외의 요소(모듈, 패키지, 필드, 지역변수)에 마킹을 해야한다면 애너테이션을 사용할 수 밖에 없다.** 클래스와 인터페이스만이 인터페이스를 구현하거나 확장할 수 있기 때문이다.

**마킹된 객체를 매개변수로 받는 메서드를 작성할 일이 있다면 마커 인터페이스를 사용하자.** 이렇게 하면 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일타임에 오류를 잡아낼 수 있다. 이런 메서드를 작성하지 않는다는 확신이 있다면 마커 애너테이션이 더 나은 선택이 될 것이다.

# 핵심 정리

새로 추가하는 메서드 없이 단지 타입 정의가 목적이라면 마커 인터페이스를 사용하자.

클래스나 인터페이스 외의 요소에 마킹해야 하거나, 애너테이션을 적극 활용하는 프레임워크에 사용될 마커라면 애너테이션을 사용하자.