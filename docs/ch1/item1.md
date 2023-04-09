# 아이템 1: 가변성을 제한하라

```kotlin
val account = BankAccount()
account.deposit(100.0)
account.withdraw(50.0)
```

변하는 상태를 가지는 객체의 어려움
- 프로그램을 이해하고 디버그하기 힘들다
- 코드 실행을 추론하기 어렵다
- 멀티스레드에서 적절한 동기화가 필요하다
- 테스트하기 어렵다
- 상태변경을 다른 부분에 알려야할 수 있다

## 코틀린에서 가변성 제한하기

- 읽기 전용 프로퍼티 (val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분
- 데이터 클래스의 copy

### 읽기 전용 프로퍼티

val은 값이 변하지 않지만 `val list = mutableList(1,2,3)`과 같은 경우 값이 변할 수 있다.
또한 내부에서 다른 var프로퍼티를 사용하는 경우 값이 변할 수 있다.
```kotlin
val fullName
    get() = "$name $surname"

val buzz
    get() = calculate()
```

val은 읽기 전용 프로퍼티이지만 불변은 아니다.
완전히 변경할 수 없다면 final 프로퍼티를 사용하는 것이 좋다.

### 가변 컬렉션과 읽기 전용 컬렉션 구분하기

List, Collection은 읽기 전용이고 이를 MutableList, MutableCollection이 상속받아 변경을 위한 메서드를 추가했다.
진짜 불변하게 만들지 않고 읽기 전용으로 만드는 것으로 플랫폼 고유의 컬렉션을 사용할 수 있도록 했다.
하지만 개발자가 이를 우회하여 사용하면 문제가 된다. 이는 약속의 문제다. => "읽기 전용이라고 써놓으면 읽기만 하라"

```kotlin
val list = listOf(1,2,3)

// 하지 마세요.
if (list is MutableList) {
    list.add(4)
}
```

변경을 해야한다면 mutable로 복제 후 사용한다.

```kotlin
val list = listOf(1,2,3)
val mutableList = list.toMutableList()
mutableList.add(4)
```

### 데이터 클래스의 copy

불변객체를 사용하면 다음의 장점이 있다.
1. 한번 정의된 상태가 유지
2. 객체를 공유해도 충돌이 발생하지 않아 병렬처리를 안전하게 할 수 있다
3. 참조가 변경되지 않아 캐시를 쉽게 할 수 있다.
4. 방어적 복사본(defensive copy)을 만들 필요가 없고 깊은 복사를 할 필요가 없다.
5. 다른 객체를 만들때 활용하기 좋다.
6. set, map의 키로 사용할 수 있다. '아이템 41: hashCode의 규약을 지켜라'에서 자세히 설명.

불변객체는 변경할 수 없으므로 copy를 하면서 새로운 객체를 만드는 방법으로 변경을 해야한다. data 클래스는 이를 편하게 제공한다.

```kotlin
class User (
    val name: String,
    val surName: String
) {
    fun withSurname(surname: String) = User(name, surname)
}
val user = User("A", "B")
val user2 = user.withSurname("C")
```

```kotlin
data class User (
    val name: String,
    val surname: String
)
val user = User("A", "B")
val user2 = user.copy(surname = "C")
```

## 다른 종류의 변경 가능 지점

변경가능한 리스트를 만든다고 한다면 mutable 컬렉션, var로 읽고 쓸 수 있는 프로퍼티를 만들 수 있다.
멀티스레드 안정성을 위해서 변경 가능 지점에서 동기화를 사용해야한다.

상태를 변경할 수 있는 불필요한 방법을 만들지 않아야 한다. 상태를 변경하는 모든 코드를 이해해야하므로 비용이 발생한다.

## 변경 가능 지점 노출하지 말기

```kotlin
class UserRepository {
    private val storedUsers: MutableMap<Int, String> = mutableMapOf()
    
    fun loadAll(): MutableMap<Int, String> {
        return storedUsers
    }
}
```

loadAll이 MutableMap을 리턴하므로 수정에 위험할 수 있다. 이를 방어하기 위해서 방어적 복제를 사용하거나 읽기 전용 타입인 Map으로 리턴하는 것이 좋다.

```kotlin
class UserRepository {
    fun loadAll(): Map<Int, String> {
        return storedUsers
    }
}
```

