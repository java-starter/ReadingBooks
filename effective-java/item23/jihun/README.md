# 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

# 1. 태그 달린 클래스란?

태그란 **해당 클래스가 어떠한 타입인지에 대한 정보를 담고있는 멤버 변수**를 의미한다.

아래 클래스에서는 shape라는 필드에 Figure 클래스가 어떤 타입인지 정보를 담고 있다.

```java
public class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radis;

    // 원(CIRCLE)용 생성자
    public Figure(double radis) {
        shape = Shape.CIRCLE;
        this.radis = radis;
    }

    // 사각형(RECTANGLE)용 생성자
    public Figure(double length, double width) {
        this.shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radis * radis);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

# 2. 태그 달린 클래스의 단점

- 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다.
- 여러 구현이 한 클래스에 혼합돼 있어서 가독성도 나쁘다.
- 다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다.
- 필드를 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화 해야 한다.
- 새로운 의미를 추가할 때마다 모든 switch 문을 찾아 새 의미를 처리하는 코드를 추가해야한다.
    - 예를들어 삼각형을 추가한다면 switch 문을 수정해야한다.
- 인스턴스의 타입만으로는 현재 나타내는 의미를 전혀 알 길이 없다.

  아래 클래스만 봐도 무슨 의미인지 전혀 할 수없다.

    ```java
    public class Main {
        public static void main(String[] args) {
            Figure figure = new Figure(3.0);
        }
    }
    ```

위와 같은 문제점들을 보면 **태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율 적이다**

# 3. 서브 타이핑(subtyping)

자바에서는 타입 하나로 다양한 의미의 객체를 표현하는 훨씬 나은 수단으로 서브타이핑(subtyping)을 제공한다.

- **서브타이핑(subtyping)** 이란 타입 계층을 구성하기 위해 상속을 사용하며, 이러한 계층을 구성하기 위해 **인터페이스 상속**(interface inheritance)**을 사용**한다.

> **서브클래싱(subclassing)**: 다른 클래스의 코드를 재사용할 목적으로 상속을 사용하며, 이러한 목적을 달성하기 위해 구현상속(implementation inheritance) 또는 클래스 상속 (class inheritance)을 사용한다.

아래 코드처럼 서브 타이핑(subtyping)을 통해 Figure이라는 타입 하나로 Circle(원)을 표현할 수도 있으며, Rectange(사각형)을 표현할 수 있다.

```java
public interface Figure {
}

class Circle implements Figure{
    
}

class Rectangle implements Figure{
    
}

class Main {
    public static void main(String[] args) {
        Figure circle = new Circle();
        Figure rectangle = new Rectangle()
    }
}
```

# 4. 태그달린 클래스를 클래스 계층구조로 바꾸기

1. 계층구조의 root가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 root class의 추상 메서드로 선언한다.

    ```java
    abstract class Figure {
    	double area();
    }
    ```

2. 태그 값에 상관없이 동작이 일정한 메서드들을 root class에 일반 메서드로 추가한다.
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 root class에 추가한다.
4. root class를 확장한 구체 클래스를 의미별로 하나씩 정의한다.

    ```java
    abstract class Figure {
        abstract double area();
    }

    class Circle extends Figure {
        @Override
        double area() {
            return 0;
        }
    }

    class Rectangle extends Figure {
        @Override
        double area() {
            return 0;
        }
    }
    ```

5. 각 하위 클래스에 각자의 의미에 해당하는 데이터 필드들을 넣는다.

    ```java
    abstract class Figure {
        abstract double area();
    }

    class Circle extends Figure {
        final double radius;

        public Circle(double radius) {
            this.radius = radius;
        }

        @Override
        double area() {
            return 0;
        }
    }

    class Rectangle extends Figure {
        final double length;
        final double width;

        public Rectangle(double length, double width) {
            this.length = length;
            this.width = width;
        }

        @Override
        double area() {
            return 0;
        }
    }
    ```

6. root class가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다.

    ```java
    abstract class Figure {
        abstract double area();
    }

    class Circle extends Figure {
        final double radius;

        public Circle(double radius) {
            this.radius = radius;
        }

        @Override
        double area() {
            return Math.PI * (radius * radius);
        }
    }

    class Rectangle extends Figure {
        final double length;
        final double width;

        public Rectangle(double length, double width) {
            this.length = length;
            this.width = width;
        }

        @Override
        double area() {
            return length * width;
        }
    }
    ```

태그를 달린 클래스를 위와 같이 계층구조로 변경하니 태그 달린 클래스의 모든 단점이 사라졌으며, 도형이 추가되어도 추상클래스만 상속받으면 되므로, 계층구조가 확장되어도 깔끔하다.

# 참고

- [https://jaehun2841.github.io/2020/07/18/object-chapter13/#서브클래싱과-서브-타이핑](https://jaehun2841.github.io/2020/07/18/object-chapter13/#%EC%84%9C%EB%B8%8C%ED%81%B4%EB%9E%98%EC%8B%B1%EA%B3%BC-%EC%84%9C%EB%B8%8C-%ED%83%80%EC%9D%B4%ED%95%91)
- [https://javabom.tistory.com/45](https://javabom.tistory.com/45)