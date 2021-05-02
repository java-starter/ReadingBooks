# 아이템 33. 타입 안전 이종 컨테이너를 고려하라

일반적으로 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다.

예를들어 아래 stringSet 컨테이너 같은 경우 매개변수화 타입은 String 이기 때문에 String 외에 다른 것은 저장할 수 가 없다.

```java
Set<String> stringSet = new HashSet<>();
```

이 때 타입 안전 이종 컨테이너를 사용하면 이러한 매개변수화 할 수 있는 타입을 하나가 아닌 여러개를 사용할 수 있으며, 동시에 제네릭의 타입 안전성을 누릴 수 있다.

# 1. 타입 안전 이종 컨테이너 패턴이란(type safe heterogeneous container pattern)?

컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 설계 방식을 의미한다.

각 타입의 Class 객체를 매개변수화한 키 역할로 사용하며, class 리터럴 타입은 Class<T> 이다.

`putFavorite` 메서드를 보면 Class 객체를 매개변수화한 키 역할로 사용하고 있으며, Class 객체의 정규 타입 매개변수를 값의 타입으로 사용하고 있기 때문에 문제가 없다.

```java
class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

> 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 타입 토큰(type token)이라 한다.

# 2. 타입 안전 이종 컨테이너의 제약

## 2-1. 악의적인 클라이언트가 Class 객체를 row type으로 넘기면 타입 안전성이 깨진다.

Class 객체를 row type으로 넘김에 따라 `putFavorite` 메서드에서 타입을 정상적으로 체크하지 못해서 `favorites` 변수에 저장이 되며, `getFavorite` 메서드 호출시 String을 Integer로 캐스팅 하기 때문에 `ClassCastException` 이 발생한다.

```java
public class Application {

    public static void main(String[] args) {
        Favorites f = new Favorites();
				//row type으로 넘김
        f.putFavorite((Class)Integer.class, "Integer");

        Integer favoriteInteger = f.getFavorite(Integer.class);

        System.out.printf("%x", favoriteInteger);
    }
}

class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

위와 같이 타입 불변식을 어기는 일이 없도록 할려면 `putFavorite` 메서드에서 인수로 주어진 instance의 타입이 type으로 명시한 타입인지 같은지 확인하면 된다.

동적 형변환인 `type.cast(instance)` 를 사용해서 타입 불일치시 `ClassCastException` 이 발생하기 때문에 아예 저장되지 않는다.

```java
class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

## 2-2. 실체화 불가 타입에는 사용할 수 없다.

`List<String>, List<Integer>` 같은 것은 저장할 수 없다.

왜냐하면 `List<String>, List<Integer>` 의 Class 객체는 List.class 이기 때문이다.

만약 해당 문법이 허용된다면 List<String>을 사용해도 모든 List를 반환하기 때문에 전혀 다른 결과가 발생할 것이다.

> List<String>.class, List<Integer>.class 는 애초애 문법 오류이다.

# 3. 한정적 타입 토큰을 사용한 타입 제한

한정적 타입 토큰이란 한정적 타입 매개변수나 한정적 와일드 카드 처럼 표현 가능한 타입을 제한하는 타입 토큰이다.

아래 코드에서는 String의 하위 타입만 들어올 수 있도록 타입을 제한한 코드이다.

```java
class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T extends String> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

## 3-1. 한정적 타입 토큰에 와일드 카드 타입 토큰 넘겨주는 방법

아래 `getAnnotation` 메서드 처럼 한정적 타입 토큰에 와일드 카드 타입 토큰 `Class<?>` 를 넘겨줄려면 Class 클래스에 있는 asSubclass 메서드를 사용해서 형변환 후 넘겨주면 된다.

```java
public interface AnnotatedElement {
		<T extends Annotation> T getAnnotation(Class<T> annotationClass);
}
```

asSubclass 메서드는 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다(형변환된다는 것은 이 클래스가 인수로 명시한 클래스의 하위 클래스라는 뜻이다). 형변환에 성공하면 인수로 받은 클래스 객체를 반환 하고, 실패하면 CIassCastException을 던진다

```java
static Annotation getAnnotation(AnnotatedElement element,
                                  String annotationTypeName) {
    Class<?> annotationType = null;
    try {
        annotationType = Class.forName(annotationTypeName);
    }catch (Exception e) {
        throw new IllegalArgumentException(e);
    }
    return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```

# 정리

컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다. 하지만 컨데이너 자체가 아닌 키를 타입 매개변수로 바 꾸면 이런 제약이 없는 타입 안전 이종 컨데이너를 만들 수 있다. 타입 안전 이종 컨테이너는 CIass를 키로 쓰며, 이런 식으로 쓰이는 CIass 객체를 타입 토큰이라 한다.