# 아이템 26. 로 타입은 사용하지 말라

## 1. 로 타입(raw type)이란?

제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.

```jsx
List list = new ArrayList(); //raw type
List<String> list = new ArrayList(); //parameterized type
```

## 2. 로 타입(raw type)을 지원하는 이유

로 타입(raw type)을 지원하는 이유는 제네릭을 도입되기 전인 JDK 5 이전의 코드와 호환을 위해서이다.

## 3. 로 타입(raw type)을 쓰면 안되는 이유

로 타입(raw type)을 쓰게 되면 제네릭이 안겨주는 타입 안정성과 표현력을 모두 잃기 때문이다.

타입 안정성 : 컴파일시 타입을 체크하기 때문에 안정성이 높아진다.
표현력 : 제네릭을 사용하면 타입을 명확히 할 수 있으며 불필요한 캐스팅 코드가 사라진다.

## 4. 로 타입(raw type) 대신 비한정적 와일드 카드 타입을 사용하자

타입 매개변수를 신경 쓰고 싶지 않아서 로 타입(raw type)을 사용할 수도 있는데, 그 보다 비한정적 와일드 카드타입을 사용하는 것이 좋다.

비한정적 와일드 카드를 사용하면 타입을 신경쓰지 않으면서, 제네릭의 이점을 얻을 수 있기 때문이다.

### 4-1. wild card 대신 type parameter를 쓰면 안될까?

wild card 대신 type parameter를 사용해도 된다.

다만, 두 개를 사용해야하는 시점이 미묘하게 다르다.

type parameter는 현재 generic type은 모르지만 generic type이 정해지면 해당 type이 무엇인지 알고 사용 하겠다는 의미이기 때문에 generic type의 값을 사용한다면 type parameter를 사용해야한다.

wild card는 현재 generic type을 모르지만 generic type이 정해져도 해당 type에 전혀 관심이 없다라는 의미이기 때문에 generic type의 값을 사용하지 않는다면 wild card를 사용해야한다.

예를들어 List가 비어있는지 체크하는 메서드를 만들 때 해당 메서드에서는 generic type의 값을 사용하지 않으므로 wild card로 만드는게 더 적합하다.

```java
//제네릭 메서드
static <T> boolean isEmpty(List<T> list) {
    return list.size() == 0;
}

//isEmpty 메서드는 제네릭 타입에 관심이 없기 때문에, 타입과 관련된 메서드가 사용되지 않으므로 wild card를 사용해도 문제가 없다. 
static boolean isEmpty(List<?> list) {
    return list.size() == 0;
}
```

wild card를 사용해야하는 것을 type parameter로 사용하게 되면 아래와 같은 단점이 있다.
1. 제네릭 메서드는 type parameter와 관련된 작업을 하겠다는 것이 외부로 노출된다.
2. API를 설계한 사람의 의도를 바르게 드러내지 못한다.

## 5. 로 타입(raw type)을 써야할 때

로 타입(raw type)을 안쓰는게 좋지만 몆 개의 예외가 존재한다.

1. class 리터럴에는 로 타입(raw type)을 써야한다. 그냥 규약이다.

    ```jsx
    String.class, List.class
    ```

2. instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에서는 적용할 수 없다. 왜냐하면 매개변수화 타입은 런타임시 다 소거되기 때문이다. 만약 instanceof 연산자를 사용하고 싶다면 로 타입(raw type)으로 비교 후 와일드 카드 타입으로 변경하면 된다.

## 정리

로타입은 단지 JDK5 이전 코드와의 호환성을 위해 제공 될 뿐이며, 사용하지 않을 경우 컴파일시 타입 관련 오류를 발견하기 힘드므로 로타입을 사용하지 말자!

## 용어 정리

- parameterized type (매개변수화 타입) : List<String>
- actual type parameter (실제 타입 매개변수) : String
- generic type (제네릭 타입) : List<E>
- formal type parameter (정규 타입 매개변수) : E
- unbounded wildcard type (비한정적 와일드 카드 타입) : List<?>
- raw type (로 타입) : List
- bounded type parameter (한정적 타입 매개변수) : <E extencs Number>
- recursive type bound (재귀적 타입 한정) : <T extends Comparable<T>>
- bounded wildcard type (한정적 와일드카드 타입) <? extends Number>
- gereric method (제네릭 메서드) : static <E> List<E> asList(E[] a)
- type token (타입 토큰) : String.class

## 참고

- [https://www.youtube.com/watch?v=ipT2XG1SHtQ](https://www.youtube.com/watch?v=ipT2XG1SHtQ)
- [https://www.youtube.com/watch?v=PQ58n0hk7DI&t=1299s](https://www.youtube.com/watch?v=PQ58n0hk7DI&t=1299s)