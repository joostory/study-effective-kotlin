# 아이템 40: equals의 규약을 지켜라

Any의 equals, hashCode, toString는 잘 설정된 규약을 가지고 있다.  
Any를 상속한 클래스는 이 규약을 잘 지켜주는게 중요하다.

## 동등성

- 구조적 동등성: equals와 이를 기반으로 한 ==, !=로 확인하는 동등성
- 레퍼런스적 동등성: ===, !==와 같이 같은 레퍼런스를 가리키는지 확인하는 동등성

```kotlin
open class Animal
class Book
Animal() == Book() // 오류: 다른 타입의 객체는 비교할 수 없다
Animal() === Book() // 오류

class Cat: Animal()
Animal() == Cat() // 가능
Animal() === Cat() // 가능
```

## equals가 필요한 이유
Any에 구현된 equals는 디폴트로 === 로 실제 같은 객체인지 비교한다.  
하지만 객체 내 값으로 비교를 해야하는 경우도 있다. data 클래스와 같은 경우 생성자 프로퍼티를 비교한다.

일부의 프로퍼티만 비교하는 경우에도 유용하다.
```kotlin
class DataTime(
  private var millis: Long = 0L,
  private var timeZone: TimeZone? = null
) {
  private var asStringCache = ""
  private var changed = false

  override fun equals(other: Any?): Boolean =
    other is DateTime &&
      other.millis == millis &&
      other.timeZone == timeZone
}
```

data 클래스의 equals는 생성자 프로퍼티만 비교하므로 아래와 같이 data 클래스를 사용해도 같은 결과를 얻는다.
```kotlin
data class DataTime(
  private var millis: Long = 0L,
  private var timeZone: TimeZone? = null
) {
  private var asStringCache = ""
  private var changed = false
}
```
data클래스를 사용하면 euqals뿐만 아니라 copy도 생성자 프로퍼티만 사용하므로 일반적으로 직접구현하지 않고 data 클래스를 사용하는 것이 편리하다.

equals를 직접 구현해야하는 경우
- 기본동작과 다른 경우
- 일부 프로퍼티만 비교해야하는 경우
- 비교해야하는 프로퍼티가 기본생성자에 없는 경우

## equals의 규약
코틀린에서의 equals 규약
- 반사적(reflexive) 동작: null이 아니라면 x.equals(x)는 true
- 대칭적(symmetric) 동작: null이 아니라면 x.equals(y) == y.equals(x)
- 연속적(transitive) 동작: null이 아니라면 x.equals(y), y.equals(z)가 true라면 x.equals(z)도 true
- 일관적(consistent) 동작: null이 아니라면 x.equals(y)는 항상 같은 값
- null과 관련된 동작: null이 아니라면 x.qeuals(null)은 항상 false

### 반사적 동작
x.equals(x)는 true  

위반하기 쉽지 않지만 실수로 위반할 수도 있다.
```kotlin
class Time(
  val millisArg: Long = -1,
  val isNow: Boolean = false
) {
  val millis: Long get() =
    if (isNow) System.currentTimeMillis()
    else millisArg

  override fun equals(other: Any?): Boolean = 
    other is Time && millis == other.millis
}

val now = Time(isNow = true)
now == now // true, false 모두 가능.
```
이 코드는 millis를 가져올때마다 System.currentTimeMillis()를 실행하기 때문에 값이 달라지는 경우가 있다.

```kotlin
sealed class Time
data class TimePoint(val millis: Long): Time()
object Now: Time()
```

### 대칭적 동작
x.equals(y) == y.equals(x)  

일반적으로 다른 타입과 동등성을 확인하려고 할 때 위반된다.

```kotlin
class Complex(
  val real: Double,
  val imaginary: Double
) {
  override fun equals(other: Any?): Boolean {
    if (other is Double) {
      return imaginary == 0.0 && real == other
    }
    return other is Complex &&
      real == other.real &&
      imaginary == other.imaginary
  }
}
```
Complex는 Double과 비교할 수 있지만 Double은 Complex와 비교할 수 없다.

```kotlin
Complex(1.0, 0.0).equals(1.0) // true
1.0.equals(Complex(1.0, 0.0)) // false
```

이는 contains, 단위 테스트 등에서도 예측하지 못한 동작이 발생할 수 있다는 것
```kotlin
val list = listOf(Any)(Complex(1.0, 0.0))
list.contains(1.0) // 현재 JVM은 false. 하지만 구현에 따라 달라질 수 있다.

val list2 = listOf<Any>(1.0)
list2.contains(Complex(1.0, 0.0)) // true
```

예측하지 못한 동작이 발생하는 것은 디버깅으로 찾기가 정말 어렵다. 따라서 다른 클래스는 동등하지 않게 만드는 것이 좋다. `other is Double` 조건을 제거하면 문제가 발생할 여지가 줄어든다.

### 연속적 동작
x.equals(y), y.equals(z)가 true라면 x.equals(z)도 true  

연속적인 동작을 설계할 때 가장 큰 문제는 타입이 다른 경우다.
```kotlin
open class Date(
  val year: Int,
  val month: Int,
  val day: Int
) {
  override fun equals(o: Any?): Boolean = when(o) {
    is DateTime -> this == o.date
    is Date -> o.day == day && o.month == month && o.year == year
    else -> false
  }
}

class DateTime(
  val date: Date,
  val hour: Int,
  val minute: Int,
  val second: Int
) {
  override fun equals(o: Any?): Boolean = when(o) {
    is DateTime -> o.date == date && o.hour == hour && o.minute == minute && o.second == second
    is Date -> date == o
    else -> false
  }
}
```
위 코드는 DateTime과 Date를 비교할 때보다 DateTime과 DateTime을 비교할 때 더 많은 프로퍼티를 확인한다.

```kotlin
val o1 = DateTime(Date(1992, 10, 20), 12, 30, 0)
val o2 = Date(1992, 10, 20)
val o3 = DateTime(Date(1992, 10, 20), 14, 45, 30)

o1 == o2 // true
o2 == o3 // true
o1 == o3 // false
```
Date, DateTime은 상속 관계이기 때문에 비교를 못하게 할 수 없다. (리스코프 치환 원치 위배) 상속 대신 컴포지션을 사용하는 것이 좋다.

```kotlin
data class Date(
  val year: Int,
  val month: Int,
  val day: Int
)

data class DateTime(
  val date: Date,
  val hour: Int,
  val minute: Int,
  val second: Int
)

val o1 = DateTime(Date(1992, 10, 20), 12, 30, 0)
val o2 = Date(1992, 10, 20)
val o3 = DateTime(Date(1992, 10, 20), 14, 45, 30)

o1 == o2 // false
o2 == o3 // false
o1 == o3 // false

o1.date == o2 // true
o2 == o3.date // true
o1.date == o3.date // true
```

### 일관성 동작
x.equals(y)는 항상 같은 값  

- 비교대상이 되는 두 객체에만 의존하는 순수함수여야 한다.
- 이를 위반한 것이 위의 `Time`이고, 대표적으로 `java.net.URL` 이 있다.

### URL과 관련된 equals 문제

```kotlin
URL("https://en.wikipedia.org") === URL("https://wikipedia.org") // true or false
```
- 이 코드는 두 URL의 ip를 비교하므로 true를 리턴한다.
- 하지만 인터넷 연결이 없으면 false다.
- equals, hashCode는 빠를 거라 기대하지만 네트워크를 사용하므로 느리다.
- virtual hosting을 사용하는 경우 관련없는 사이트가 true일 수 있다.

### equals 구현하기
- 특별한 이유가 없는 이상 직접 구현하는 것은 좋지 않다.
- 기본적인 것을 그대로 사용하거나 data class를 사용
- 직접 구현해야한다면 반사적, 대칭적, 연속적, 일관적 동작을 확인하고 final로 만드는 것이 좋다.
