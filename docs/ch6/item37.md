# 아이템 37: 데이터 집합 표현에 data 한정자를 사용하라

data 한정자를 붙이면 다음의 함수가 자동으로 생성된다.
- toString
- equals, hashCode
- copy
- componentN (component1, component2 ...)

## toString
클래스 이름, 모든 프로퍼티와 값을 출력한다.

```kotlin
print(player) // Player(id=0, name=Name, points=99999)
```

## equals
기본 생성자의 프로퍼티가 같은지 확인한다. hashCode는 equals와 같은 결과를 낸다.

```kotlin
player == Player(0, "Name", 99999) // true
player == Player(0, "Name?", 888) // false
```

## copy
기본 생성자 프로퍼티가 같은 새로운 객체를 복제한다. immutable 클래스를 만들때 편리하다.
복사할때 argments를 이용해 값을 변경할 수 있다.

```kotlin
val newObj = player.copy(name = "Thor")
print(newObj) // Player(0, "Thor", 99999)
```

copy는 얕은 복사를 하지만 immutable이라면 상관없다.
immutable은 값이 변경되지 않기 때문에 깊은 복사가 필요없다.

## componentN
위치기반으로 객체의 프로퍼티에 접근할 수 있게 해준다.

```kotlin
val (id, name, points) = player
```
이러한 destructuring은 다음과 같이 구현되었을 것이다.

```kotlin
val id = player.component1()
val name = player.component2()
val points = player.component3()
```

변수의 이름을 원하는 대로 지정할 수 있다.
```kotlin
val visited = listOf("China", "Russia", "India")
val (first, second, third) = visited

val trip = mapOf("China" to "Tianjin", "Russia" to "Petersburg")
for ((country, city) in trip) {
    //...
}
```

하지만 순서를 혼동하면 문제가 생길 수 있다.
```kotlin
data class FullName(
    val firstName: String,
    val secondName: String,
    val lastName: String
)

val elon = FullName("Elon", "Reeve", "Musk")
val (name, surname) = elon 
```

값을 하나만 갖는 클래스에 사용하면 문제가 생길 수 있다.
```kotlin
data class User(val name: String)
val user = User("John")
user.let { a -> print(a) } // User(name=John)
user.let { (a) -> print(a) } // John
```

## 튜플 대신 데이터 클래스 사용하기

튜플은 Serializable 기반으로 만들어진 toString을 사용할 수 있는 데이터 클래스
- https://github.com/JetBrains/kotlin/blob/30788566012c571aa1d3590912468d1ebe59983d/libraries/stdlib/src/kotlin/util/Tuples.kt

```kotlin
public data class Pair<out A, out B>(
    public val first: A,
    public val second: B
) : Serializable {
    public override fun toString(): String = "($first, $second)"
}

public data class Triple<out A, out B, out C>(
    public val first: A,
    public val second: B,
    public val third: C
) : Serializable {
    public override fun toString(): String = "($first, $second, $third)"
}
```

튜플보다 데이터 클래스를 사용하는 것이 좋은데 Pair와 Triple은 몇가지 지역적인 목적으로 인해 남아있다.
- 값에 간단한 이름을 붙일때
- 미리 알 수 없는 집합을 표현할 때 (mapOf 등)

이 경우를 제외하면 데이터 클래스를 사용하는 것이 좋다. 다음의 parseName은 첫번째, 두번째를 착각하면 문제가 발생할 수 있다.
```kotlin
fun String.parseName(): Pair<String, String>? {
    val indexOfLastSpace = this.trim().lastIndexOf(' ')
    if (indexOfLastSpace < 0) return null
    val firstName = this.take(indexOfLastSpace)
    val lastName = this.drop(indexOfLastSpace)
    return Pair(firstName, lastName)
}
```

데이터 클래스를 사용하면 이런 착각을 방지할 수 있고 변겡에 비용도 거의 발생하지 않는다.
함수의 리턴타입이 더 명확, 짧고 전달하기도 쉽다.
```kotlin
data class FullName(val firstName: String, val secondName: String)
fun String.parseName(): FullName? {
    val indexOfLastSpace = this.trim().lastIndexOf(' ')
    if (indexOfLastSpace < 0) return null
    val firstName = this.take(indexOfLastSpace)
    val lastName = this.drop(indexOfLastSpace)
    return FullName(firstName, lastName)
}
```
