# [이펙티브 자바] Item59 - 라이브러리를 익히고 사용하라

Java 생태계에는 java.io, java.lang, java.util 등 자바 표준 라이브러리나 고품질의 서드파티 라이브러리 등 수 많은 라이브러리들이 존재한다. 아주 특별한 나만의 기능이 아니라면 이미 누군가가 라이브러리 형태로 구현해놓았을 가능성이 크다. 이러한 라이브러리들은 일반적으로 코드의 품질이 좋고 지속해서 개선된다.

구현해야할 기능이 라이브러리로 존재한다면 그것을 사용하자. 있는지 잘 모르겠다면 찾아보자. 우리는 그 라이브러리가 어떤 영역의 기능을 제공하는지 살펴보고, 익히고, 사용하면 된다.

# 표준 라이브러리를 잘 활용하지 못한 예제

아주 흔히 마주치는 문제로, 다음과 같은 메서드를 만들곤한다.

```java
// 문제가 많은 코드!
static Random rnd = new Random();

static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```

위 코드는 세 가지의 문제점을 내포하고 있다.

1. n이 크지 않은 2의 제곱수라면 조금 뒤 같은 수열이 반복된다.
2. n이 2의제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다.
3. 지정한 범위를 벗어난 수가 종종 튀어나온다.

### 지정한 범위를 벗어나는 이유

위의 코드에서 Math.abs()로 정수를 반환하기 때문에 발생한다. 예를들어, nextInt()가 Integer.MIN_VALUE를 반환하면 Math.abs도 Interger.MIN_VALUE를 반환한다. 따라서 `Math.abs(rnd.nextInt()) % n`은 음수를 반환하게 된다.

> Math.abs(Integer.MIN_VALUE)가 음수를 반환하는 이유? <br/>
Integer의 범위는 2^31(-2,147,483,648) ~ 2^31-1(2,147,483,647) 이다. -2,147,483,648의 절대값은 Integer 최대 범위를 벗어나고, 이 수는 Integer로 표현할 수 없다. 따라서 최솟값을 가장 잘 표현할 수 있는 동일한 값을 내어준다.

## 해결 방법

Random.nextInt(n) 메서드를 사용하면 이 결함은 간단히 해결된다. 우리는 이러한 문제점을 직접 해결하려 하지 않아도 된다. 앞서 말했듯, 표준 라이브러리는 지속적으로 개선되고 있기 때문이다. 우리는 이러한 라이브러리들을 잘 익히고 사용하면 될 뿐이다.

Java 7부터는 Random을 더 이상 사용하지 않는 게 좋다. ThreadLocalRandom으로 대체하면 대부분 잘 동작하고 더 빠르다.

# 표준 라이브러리의 이점

## 1. 전문가의 지식과 다른 프로그래머들의 경험을 활용할 수 있다.

라이브러리의 코드를 작성한 전문가의 지식과 이를 사용해본 다른 프로그래머들의 많은 경험을 토대로 손쉽게 원하는 기능을 만들수 있다.

## 2. 핵심적인 일과 관련 없는 문제를 해결하는데 드는 시간을 줄일 수 있다.

애플리케이션 기능 개발에 집중할 수 있다.

## 3. 따로 노력하지 않아도 성능이 개선된다.

표준 라이브러리는 더 나은 방법을 꾸준히 모색한다. 자바 플랫폼 라이브러리의 많은 부분들이 지속적으로 개선되어지며 성능 개선이 이루어진 것처럼 말이다.

## 4. 기능이 점점 많아진다.

라이브러리가 개선되어지며 부족한 기능들도 발견되는데, 이 부분이 다음 릴리즈때 추가되곤 한다.

## 5. 작성한 코드가 자연스럽게 좋은 코드가 된다.

라이브러리를 사용하면 사람들에게 낯익은 코드가 된다. 자연스럽게 다른 개발자들이 읽기 좋고, 유지보수하기 좋고, 재활용하기 쉬운 코드가 된다.

# 메이저 릴리스에 관심을 가지자

실상 많은 개발자들이 라이브러리를 사용하기 보다 직접 구현해서 사용하고 있다. 그 이유는 아마도 라이브러리에 그런 기능이 존재하는지 모르기 때문일 것이다.

Java 메이저 릴리스마다 주목할 만한 수많은 기능이 라이브러리에 추가된다. Java는 이런 메이저 릴리스마다 새로운 기능을 소개하는 웹 페이지를 제공하니 관심을 가지고 살펴볼만 하다.

Java의 라이브러리는 너무 방대해서 모든 API 문서를 공부하기 어려울 수 있다. 하지만 자바 개발자라면 java.lang, java.util, java.io와 그 하위 패키지들에는 익숙해져야 한다.

컬렉션 프레임워크, 스트림 라이브러리, java.util.concurrent의 동시성 기능도 알아두면 좋다.

# 핵심 정리

바퀴를 다시 발명하지 말자. 특별한 기능이 아니라면 라이브러리에 이미 구현되어있을 가능성이 크다. 그러므로 라이브러리 사용을 우선적으로 고려해보자. 좋은 품질의 코드를 손쉽게 만들어낼 수 있다.

우선순위

- 우선 라이브러리를 사용하려 시도하자.
- 원하는 기능이 없다면 고품질의 서드파티 라이브러리를 고려하자.
- 그래도 없다면 직접 구현하자.