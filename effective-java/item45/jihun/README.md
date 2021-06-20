# 1. Stream이란?

다량의 데이터 처리 작업을 돕고자 자바 8에 추가되었다.

# 2. Stream 핵심 개념

## 2-1. Stream

데이터 원소의 유한 혹은 무한 시퀸스(sequence)를 뜻한다.

데이터 원소는 객체 참조나 기본 타입 값(int, long, double)이 가능하다.

## 2-2. Stream pipeline

원소들로 수행하는 연산 단계를 표현하는 개념이다.

Stream pipeline은 Source Stream으로 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산(intermediate operation)이 있을 수 있다.

### 2-2-1. **중간 연산(intermediate operation)**

Stream을 어떠한 방식으로 변환(transform)하는 연산으로, 대표적으로 map, filter 등이 있다.

### 2-2-2. **종단 연산(terminal operation)**

마지막 중간 연산이 내놓은 Stream에 최후의 연산을 하며, 대표적으로 collect가 있다.

### 2-2-3. 지연 평가(lazy evaluation)

종단 연산이 호출될 때 실제로 평가가 이루어 지는 개념이다.

만약 종단 연산이 호출되지 않으면 해당 Stream pipeline은 아무일도 하지 않는다.

# 3. 언제 Stream 을 사용할까?

## 3-1. 반복문 사용이 더 나을 때

- 반복문의 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 것은 불가능하다.
- 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다로는 이 중 어떤 것도 할 수 없다.
- 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃기 때문에 각 단계에서의 값들에 동시에 접근해야 한다면 반복문이 낫다.

## 3-2. Stream 사용이 더 나을 때

- 원소들의 시퀸스를 일관되게 변환한다.
- 원소들의 시퀸스를 필터링 한다.
- 원소들의 시퀸스를 하나의 연산을 사용해 결합한다.
- 원소들의 시퀸스를 컬렉션에 모은다.
- 원소들의 시퀸스에서 특정 조건을 만족하는 원소를 찾는다.

# 4. Stream 사용시 유의사항

- Stream을 과용하면 코드의 가독성이 떨어져 유지보수 하기가 어려워 진다.
    - 특히 무조건 반복문을 Stream으로 변경하기 보다, Stream으로 변경시 가독성이 더 좋으면 변경해야 한다.
- char 값에 stream 사용시 IntStream을 사용하기 때문에 혼란이 올 수 있으므로 char 값들을 처리할 때는 Stream을 삼가는 편이 낫다.

# 5. Stream 사용 Tip

- 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
- 도우미 메서드를 적절히 활용하는 일의 중요성은 일반 반복 코드에서보다 스프림 파이프라인에서 훨씬크다. 파이프라인에서는 타입 정보가 명시되지 않거나 임시 변수를 자주 사용하기 때문이다.
- 스트림을 반환하는 메서드 이름은 원소의 정체를 알려주는 복수 명사로 쓰는게 좋다.

# 정리

스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라.