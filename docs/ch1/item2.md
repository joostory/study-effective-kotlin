# 아이템 2: 변수의 스코프를 최소화하라

- 스코프를 좁게 만드는 것이 좋은 가장 중요한 이유는 프로그램을 추적하고 관리하기 쉽기 때문이다.

## 캡처링

```kotlin
val primes = sequence<Int> { 
    var numbers = generateSequence(2) { it + 1 }
    var prime: Int
    while (true) {
        prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1)
            .filter { it % prime != 0 }
    }
}
```
prime을 스코프 외부에서 만들어 한번만 생성하도록 만들 수 있으나 캡처링으로 인해 잘못된 결과가 나온다.
