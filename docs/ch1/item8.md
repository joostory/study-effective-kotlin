# 아이템 8 : 적절하게 null을 처리하라

null은 lack of value를 나타낸다. 함수는 다음과 같이 명확한 의미를 가지는 것이 좋다.
- String.toIntOrNull(): String을 int로 적절하게 변환하지 못할때 null 리턴
- Iterable<T>.firstOrNull(): 주어진 조건에 맞는 요소가 없을 경우 null 리턴

## null을 안전하게 처리하기
```kotlin
printer?.print() // 안전 호출
if (printer != null) printer.print() // 스마트 캐스팅

// Elvis 연산자
val name1 = printer?.name ?: "unnamed"
val name2 = printer?.name ?: return
val name3 = printer?.name ?: throw Error("Printer must be named")
```

- 방어적 프로그래밍: 모든 가능성을 올바른 방식으로 처리하는 것
- 공격적 프로그래밍: 예상치 못한 상황 발생시 개발자에게 알려서 수정하도록 함


## 오류를 throw하기
안전호출같은 방법을 사용하면 오류를 찾기 어렵다. 오류를 알려주기 위해서는 throw, !!, requireNotNull, checkNotNull등을 활용.

```kotlin
fun process(user: User) {
  requireNotNull(user.name) // user.name의 null 처리
  val context = checkNotNull(context) // context의 null 처리
  val networkService = getNetworkService(context)
    ?: throw NoInternetConnection() // getNetworkService의 null 처리
  networkService.getDate { data, userData ->
    show(data!!, userData!!) // data, userData의 null 처리
  }
}
```

## not-null assertion(!!)과 관련된 문제
!!를 사용하면 자바에서와 같이 NPE가 발생한다. 예외가 발생할때 어떠한 설명도 없이 generic exception이 발생한다.
따라서 가능하면 !!사용을 피하고 명시적 오류를 사용하는 것이 좋다.

## 의미없는 nullability 피하기
nullability는 어떻게든 적절하게 처리해야 하므로 추가 비용이 발생한다. 가능하면 피하는게 좋다.

- get, getOrNull, Failure와 같이 명시적으로 null을 처리
- lateinit, notNull 델리게이트 사용
- 빈 컬렉션 대신 null을 사용하지 말라

## lateinit 프로퍼티와 notNull 델리게이트

JUnit에서 @BeforeEach에서 프로퍼티에 값을 지정하는 경우가 있다. 이때 프로퍼티가 nullable이라면 사용할때마다 unpack해야한다.
lateinit을 사용하면 사용전에 설정될 것을 알려줄 수 있다.

```kotlin
class UserControllerTest {
  private lateinit var dao: UserDao
  private lateinit var controller: UserController

  @BeforeEach
  fun init() {
    dao = mockk()
    controller = UserController(dao)
  }

  @Test
  fun test() {
    controller.doSomething()
  }
}
```
- lateinit은 라이프 사이클을 갖는 클래스처럼 명황한 순서가 있는 경우 사용된다.
- Int, Long, Double, Boolean과 같은 기본 타입은 lateinit을 사용할 수 없다. Delegates.notNull을 사용한다.
