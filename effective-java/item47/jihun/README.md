# 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다.

## 언제 Stream을 반환하고 언제 Iterable을 반환할까?

Stream은 Iterable 인터페이스를 상속받지 않아 반복이 불가능 하기 때문에 Stream을 return 하거나 Iterable을 return 해야한다.

```java
//컴파일 에러 발생
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {}
```

메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 마음 놓고 스트림을 반환하는게 낫고, 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable을 반환하자.

다만 공개 API일 경우에는 Collection 인터페이스를 return하자. Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하므로 Iterable과 Stream을 동시에 지원할 수 있다.

## 정리

Stream으로 처리하기 원하는 사용자도 있고, 반복으로 처리하길 원하는 사용자도 있으므로 양쪽 다 만족시키기 위해 노력해야 하며, 컬렉션을 사용하면 Stream과 Iterable를 모두 사용할 수 있다는 장점이 있기 때문에 컬렉션을 return 하는게 좋다.