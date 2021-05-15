# 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

# 1. EnumMap이란?

열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체 이다.

# 2. ordinal 한 번 사용하는 예제

식물의 이름과 생애주기를 가지는 클래스 이다.

```java
public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

정원에 사는 식물들을 생애주기별(LifeCycle)로 묶는 코드이다.

plantsByLifeCycle 변수에 생애주기별 집합으로 저장시 ordinal를 사용해서 순서를 지정하고 있다.

```java
public class Application {
    public static void main(String[] args) {
        Plant[] garden = new Plant[6];
        garden[0] = new Plant("가", Plant.LifeCycle.ANNUAL);
        garden[1] = new Plant("나", Plant.LifeCycle.BIENNIAL);
        garden[2] = new Plant("다", Plant.LifeCycle.PERENNIAL);
        garden[3] = new Plant("마", Plant.LifeCycle.ANNUAL);
        garden[4] = new Plant("바", Plant.LifeCycle.BIENNIAL);
        garden[5] = new Plant("사", Plant.LifeCycle.PERENNIAL);

        Set<Plant>[] plantsByLifeCycle = (set<Plant>[])new Set[Plant.LifeCycle.values().length];

        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }

        for (Plant p : garden) {
            plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
        }

        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
        }
    }
}
```

위의 코드의 문제점은 다음과 같다.

1. 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 한다.
    - `Set<Plant>[] plantsByLifeCycle = (set<Plant>[])new Set[Plant.LifeCycle.values().length]`
2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
    - EnumMap은 열거 타입을 키로 사용하기 때문에 그 자체로 의미를 제공하지만 인덱스는 의미를 알 수가 없다.
3. 정확한 정숫값을 사용한다는 것을 보증해야 한다.
    - 만약 열거 타입의 순서가 변경되면 ordinal가 반환하는 값도 달라지기 때문이다.

위와 같은 문제는 열거타입을 key로 사용하는 EnumMap을 사용하면 해결할 수 있다.

1. 안전하지 않은 형변환을 사용하지 않는다.
2. Map의 Key인 결거타입 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다.
3. 배열 인덱스를 계산하는 과정에서 오류가 날 일도 없다.
    - EnumSet 내부적으로는 배열을 사용하지만 클라이언트 코드에서는 해당 배열에 접근하지 않기 때문이다.

```java
public class Application {
    public static void main(String[] args) {
        Plant[] garden = new Plant[6];
        garden[0] = new Plant("가", Plant.LifeCycle.ANNUAL);
        garden[1] = new Plant("나", Plant.LifeCycle.BIENNIAL);
        garden[2] = new Plant("다", Plant.LifeCycle.PERENNIAL);
        garden[3] = new Plant("마", Plant.LifeCycle.ANNUAL);
        garden[4] = new Plant("바", Plant.LifeCycle.BIENNIAL);
        garden[5] = new Plant("사", Plant.LifeCycle.PERENNIAL);

        Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

        for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
            plantsByLifeCycle.put(lc, new HashSet<>());
        }

        for (Plant p : garden) {
            plantsByLifeCycle.get(p.lifeCycle).add(p);
        }

        System.out.println(plantsByLifeCycle);
    }
}
```

위의 코드 대신 stream을 사용하면 코드를 더 줄일 수 있다.

```java
System.out.println(Arrays.stream(garden)
        .collect(Collectors.groupingBy(p -> p.lifeCycle,
        () -> new EnumMap<>(Plant.LifeCycle.class), Collectors.toSet())));
```

> stream을 사용하면 실제로 데이터가 존재할 때만 중첩 맵을 생성한다.
>
> 예를들어 garden 배열에 Plant.LifeCycle.ANNUAL가 없다면 해당 값에 해당하는 중첩맵은 생성하지 않지만 EnumMap은 해당 값이 존재하지 않아도 중첩 맵을 생성한다.


# 3. ordinal 두 번 사용하는 예제

다음 열거 타입은 세 가지 상태(Phase)를 전이(Transition)와 매핑하도록 구현한 프로그램이다.

예를들어 액체(LIQUID)에서 고체(SOLID)로의 전의는 응고(FREEZE)가 되고, 액체(LIQUID)에서 기체(GAS)로의 전이는 기화(BOLD)가 된다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                { null, MELT, SUBLIME},
                { FREEZE, null, BOIL},
                { DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

위의 코드의 문제점은 다음과 같다.

1. 컴파일러는 ordianl과 TRANSITIONS(배열 인덱스)의 관계를 알 수가 없다.
    - Phase나 Transition의 열거 타입을 수정하고, TRANSITIONS 메서드를 수정하지 않으면 문제가 발생할 수 있다.
2. Phase의 열거 타입이 늘어나면 TRANSITIONS의 배열도 커져야 한다.

위의 코드의 문제점은 맵 2개를 중첩하면 쉽게 해결할 수 있다.

안쪽 맵은(`Map<Phase, Transition>`) 이전 상태와 전이를 연결하고 바깥 맵(`Map<Phase, Map<Phase, Transition>>`)은 이후 상태와 안쪽 맵을 연결한다.

전이 전후의 두 상태를 전이 열거 타입 Transition의 입력으로 받아, 이 Transition 상수들로 중첩된 EnumMap을 초기화하면 된다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), BOIL(LIQUID, GAS);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> m = Stream.of(values())
                                                                        .collect(Collectors.groupingBy(t -> t.from,
                                                                                () -> new EnumMap<>(Phase.class),
                                                                                Collectors.toMap(t -> t.to, t -> t,
                                                                                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

만약 상태와 전이가 추가된다면 어떻게 될까?

ordian을 사용한 코드에서는 Phase와 Phase.Transition에 상태를 추가하고, TRANSITIONS 메서드에 상태전이에 관한 메서드를 추가해 줘야 한다.

만약 TRANSITIONS에 추가를 해주지 않거나 순서를 잘못해서 추가한다면 런타임에 문제를 일으킬 것이다.

반면 맵 2개를 중첩한 코드에서는 Phase와 Phase.Transition에만 추가해 주면 끝이나기 때문에 추가시 문제도 없고 오류가 발생할 일도 없기 때문에 훨씬 안전한 코드가 된다.

# 정리

배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라.