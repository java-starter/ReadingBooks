# 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

## 1. 한정적 와일드카드를 사용해야하는 이유 예제

일련의 원소를 Stack에 넣는 메서드인 pushAll 메서드를 추가해야 한다고 해보자.

```java
public class Stack<E> {

    private E[] elementData;
    private int size;

    public Stack() {
    }

    public void push(E e){}

    public void pushAll(Iterable<E> src) {
        for (E e : src) {
            push(e);
        }
    }
}
```

Stack 클래스와 위와 같을때 아래 `numberStack.push(iterator)` 코드는 컴파일 오류가 발생한다.

왜냐하면 Stack의 실제 타입 매개변수가 Number이기 때문이다. Stack의 pushAll 메서드에서 받을 수 있는 인자는 `Iterable<Number>`만 가능한데 `Iterable<Integer>`를 넘겨 줬기 때문이다.

```java
public class Main {

    public static void main(String[] args) {
				Stack<Number> numberStack = new Stack<>();
        List<Integer> list = List.of(1, 2, 3, 4, 5);
        Iterable<Integer> iterable = list;
				//컴파일 오류
        numberStack.pushAll(iterator);
    }
}
```

이러한 문제는 한정적 와일드 카드를 사용함에 따라 해결할 수 있다.

pushAll 메서드의 인자를 `Iterable<E>` 에서 `Iterable<? extends E>`로 변경하였다.

이제 `Iterable<E>` 는 E의 Iterable만 가능하나, `Iterable<? extends E>` 는 E의 하위 타입의 Iterable이 가능하다.

```java
public class Stack<E> {

    private E[] elementData;
    private int size;

    public Stack() {
    }

    public void push(E e){}

    public void pushAll(Iterable<? extends E> src) {
        for (E e : src) {
            push(e);
        }
    }

}
```

Stack 클래스의 pushAll 메서드는 이제 E의 하위 타입의 Iterable이 가능하기 때문에 컴파일 오류가 발생하지 않는다.

```java
public class Main {

    public static void main(String[] args) {
				Stack<Number> numberStack = new Stack<>();
        List<Integer> list = List.of(1, 2, 3, 4, 5);
        Iterable<Integer> iterable = list;
        numberStack.pushAll(iterator);
    }
}
```

반대로 Stack 안의 모든 원소를 주어진 컬렉션으로 옮겨 담는 popAll 메서드를 추가한다고 해보자.

```java
public class Stack<E> {

    private E[] elementData;
    private int size;

    public Stack() {
    }

    public E pop(){return elementData[size--];}

    public boolean isEmpty(){return elementData.length == 0 ? true : false;}

    public void popAll(Collection<E> dst) {
        while (!isEmpty()) {
            dst.add(pop());
        }
    }
}
```

Stack 클래스와 위와 같을때 아래  numberStack.popAll(objects) 코드는 컴파일 오류가 발생한다.

왜냐하면 Stack의 실제 타입 매개변수가 Number이기 때문이다. Stack의 popAll 메서드에서 받을 수 있는 인자는 `Collection<Number>`만 가능한데 `Collection<Object>`를 넘겨 줬기 때문이다.

```java
public class Main {

    public static void main(String[] args) {
        Stack<Number> numberStack = new Stack<>();
        Collection<Object> objects = new ArrayList<>();
        numberStack.popAll(objects);
    }
}
```

이러한 문제도 한정적 와일드 카드를 사용함에 따라 해결할 수 있다.

popAll 메서드의 인자를 `Collection<E>` 에서 `Collection<? super E>`로 변경하였다.

이제 `Collection<E>` 는 E의 Collection만 가능하나, `Collection<? super E>` 는 E의 상위 타입의 Collection이 가능하다.

```java
public class Stack<E> {

    private E[] elementData;
    private int size;

    public Stack() {
    }

    public E pop(){return elementData[size--];}

    public boolean isEmpty(){return elementData.length == 0 ? true : false;}

    public void popAll(Collection<? super E> dst) {
        while (!isEmpty()) {
            dst.add(pop());
        }
    }
}
```

Stack 클래스의 popAll 메서드는 이제 E의 상위 타입의 Collection이 가능하기 때문에 컴파일 오류가 발생하지 않는다.

```java
public class Main {

    public static void main(String[] args) {
				Stack<Number> numberStack = new Stack<>();
        List<Integer> list = List.of(1, 2, 3, 4, 5);
        Iterable<Integer> iterable = list;
				numberStack.popAll(objects);
    }
}
```

이처럼 **유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용**하는 것이 좋다.

## 2. 펙스(PECS) 공식

언제 와일드카드 타입을 써야하는지에 대한 공식으로 **produce-extends, consumer-super**을 의미한다.

Stack 예제에서 pushAll 메서드의 src 매개변수는 Stack이 사용할 E 인스턴스를 생산하는 **생산자**이기 때문에  `Iterable<? extends E>` 를 사용하였으며, popAll 메서드의 dst 매개변수는 Stack으로부터 E 인스턴스를 소비하는 **소비자**이기 때문에 `Collection<? super E>` 를 사용하였다.

```java
public class Stack<E> {

    private E[] elementData;
    private int size;

    public Stack() {
    }

    public void push(E e){}

    public E pop(){return elementData[size--];}

    public boolean isEmpty(){return elementData.length == 0 ? true : false;}

    public void pushAll(Iterable<? extends E> src) {
        for (E e : src) {
            push(e);
        }
    }

    public void popAll(Collection<? super E> dst) {
        while (!isEmpty()) {
            dst.add(pop());
        }
    }
}

```

## 3. 언제 타입 매개변수를 쓰고 언제 와일드카드를 써야할까?

**메서드 선언에 타입 매개변수가 한 번만 나오면** 와일드 카드로 대체하면 된다.

비 한정적 타입 매개변수라면 비한정적 와일드 카드로, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

아래 제네릭 메서드는 메서드 선언에 타입 매개변수가 한 번만 나왔기 때문에 와일드 카드로 대체하는 것이 좋다.

```java
public static <E> void swap(List<E> list, int 1, int j);
```

와일드카드로 교체시 아래와 같은 코드로 변경된다.

```java
public static void swap(List<?> list, int 1, int j);
```

대신 와일드카드로 교체시 문제가 있다.

**와일드카드 사용시에는 null 외에는 어떤 값도 넣을 수 없다는 점**이다.

이러한 문제를 해결하기 위해 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성해서 해결할 수 있다.

```java
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

private 도우미 메서드인 swapHelper 메서드는 list가 `List<E>`임을 알고 있다. 즉, 이 list에서 꺼낸 값의 타입은 항상 E이고, E 타입의 값이라면 이 list에 넣어도 안전함을 알고 있다.

```java
public static void swap(List<?> list, int i, int j) {
      swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

## 정리

- **메서드 선언에 타입 매개변수가 한 번만 나오면** 와일드카드로 대체하자. 와일드카드를 적용함에 따라 API가 훨씬 유연해 진다.
- 와일드카드 적용시 PECS 공식을 잘 활용하자.
- Comparable과 Comparator는 모두 소비자라는 사실도 잊지 말자.
