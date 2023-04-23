# 아이템 4: inferred 타입으로 리턴하지 말라

코틀린의 타입추론은 가장 널리 알려진 타입추론이다. 다만 몇가지 위험한 부분들이 있다.
위험한 부분을 피하려면 할당 때 inferred타입은 정확하게 오른쪽에 있는 피연산자에 맞게 설정된다는 것을 기억해야한다.

```kotlin
open class Animal
class Zebra: Animal()

var animal = Zebra() // Zebra 타입
animal = Animal() // 오류: Zebra 타입에 Animal을 할당하려고 한다.
```
animal이 Zebra타입으로 추론되므로 Animal을 할당할 수 없다.
아래와 같이 명시적으로 지정해서 해결한다.

```kotlin
var animal: Animal = Zebra()
animal = Animal()
```

API에 추론을 사용하면 위험하다.
```kotlin
val DEFAULT_CAR: Car = Fiat126P()

interface CarFactory {
  fun produce(): Car = DEFAULT_CAR
}
```

이 코드에서 produce DEFAULT_CAR의 타입이 추론가능하니 리턴타입을 지운다.
```kotlin
val DEFAULT_CAR: Car = Fiat126P()

interface CarFactory {
  fun produce() = DEFAULT_CAR
}
```

여기까지는 문제가 없다. 또 다른 누군가가 DEFAULT_CAR의 타입마저 불필요하다고 판단해서 지우면
```kotlin
val DEFAULT_CAR = Fiat126P()

interface CarFactory {
  fun produce() = DEFAULT_CAR
}
```
produce는 항상 Fiat126P 타입만 리턴하게 된다. API에는 타입을 명시적으로 제공하는 것이 좋다.

