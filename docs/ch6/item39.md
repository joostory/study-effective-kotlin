# 아이템 39: 태그 클래스보다는 클래스 계층을 사용하라

상수모드를 태그라고 부르고, 태그를 포함한 클래스를 태그 클래스라고 부른다.

태그 클래스의 문제: 서로 다른 책임을 한 클래스에 태그로 구분해서 넣는다
```kotlin
class ValueMatcher<T> private constructor(
  private val value: T? = null,
  private val matcher: Matcher
) {
  fun match(value: T?) = when(matcher) {
    Matcher.EQUAL -> value = this.value
    Matcher.NOT_EQUAL -> value != this.value
    Matcher.LIST_EMPTY -> value is List<*> && value.isEmpty()
    Matcher.LIST_NOT_EMPTY -> value is List<*> && value.isNotEmpty()
  }

  enum class Matcher {
    EQUAL,
    NOT_EQUAL,
    LIST_EMPTY,
    LIST_NOT_EMPTY
  }

  companion object {
    fun <T> equal(value: T) = ValueMatcher<T>(value = value, matcher = Matcher.EQUAL)
    fun <T> notEqual(value: T) = ValueMatcher<T>(value = value, matcher = Matcher.NOT_EQUAL)
    fun <T> emptyList() = ValueMatcher<T>(matcher = Matcher.LIST_EMPTY)
    fun <T> notEmptyList() = ValueMatcher<T>(matcher = Matcher.LIST_NOT_EMPTY)
  }
}
```
- 한 클래스에 여러 모드를 처리하기 위한 상용구가 추가된다.
- 여러목적으로 사용되므로 일관적이지 않고 더 많은 프로퍼티가 필요하다.
- 요소가 여러목적을 가지고 여러방법으로 설정할 수 있는 경우 일관성을 지키기 어렵다.
- 팩토리 메서드를 사용해야하는 경우가 많다.

코틀린에서는 sealed 클래스를 많이 사용한다. 한 클래스에 여러모드를 만드는 대신 각 모드를 클래스로 만들고 타입시스템과 다형성을 활용하는 것이다. 이 클래스에 sealed 한정자를 붙여 서브 클래스 정의를 제한한다.

```kotlin
sealed class ValueMatcher<T> {
  abstract fun match(value: T): Boolean

  class Equal<T>(val value: T) : ValueMatcher<T>() {
    override fun match(value: T): Boolean = value == this.value
  }

  class NotEqual<T>(val value: T) : ValueMatcher<T>() {
    override fun match(value: T): Boolean = value != this.value
  }

  class EmptyList<T>() : ValueMatcher<T>() {
    override fun match(value: T): Boolean = value is List<*> && value.isEmpty()
  }

  class NotEmptyList<T>() : ValueMatcher<T>() {
    override fun match(value: T): Boolean = value is List<*> && value.isNotEmpty()
  }
}
```
- 각 클래스는 필요한 데이터만 있고 적절한 파라미터만 갖는다.

## sealed 한정자

sealed 한정자는 외부파일에서 서브 클래스를 만드는 행위를 제한하므로 타입이 추가되지 않을 것을 보장한다. 따라서 when구분에 else를 사용할 필요가 없다.

```kotlin
fun <T> ValueMatcher<T>.reversed(): ValueMatcher<T> = 
  when(this) {
    is ValueMatcher.EmptyList -> ValueMatcher.NotEmptyList<T>()
    is ValueMatcher.NotEmptyList -> ValueMatcher.EmptyList<T>()
    is ValueMatcher.Equal -> ValueMatcher.NotEqual(value)
    is ValueMatcher.NotEqual -> ValueMatcher.Equal(value)
  }
```

## 태그 클래스와 상태 패턴의 차이
태그 클래스와 상태 패턴은 혼동하면 안된다. 상태 패턴은 객체 내부 상태가 변화할 때 객체의 동작이 변하는 소프트웨어 디자인 패턴이다.

아침 운동을 위한 앱을 만든다고 해보자. 각각의 운동 전에 준비시간이 있고 완료 후 완료했다는 메시지를 출력한다.
상태 패턴을 사용한다면 서로 다른 상태를 나타내는 클래스 계층 구조를 만들게 된다. 그리고 현재 상태를 나타내기 위한 읽고 쓸 수 있는 프로퍼티도 만들게 된다.
```kotlin
sealed class WorkoutState

class PrepareState(val exercise: Exercise) : WorkoutState()
class ExerciseState(val exercise: Exercise) : WorkoutState()
object DoneState : WorkoutState()

fun List<Exercise>.toStates(): List<WorkoutState> =
  flatMap { exercise ->
    listOf(PrepareState(exercise), ExerciseState(exercise))
  } + DoneState

class WorkoutPresenter(/* .. */) {
  private var state: WorkoutState = states.first()
  // ...
}
```
- 상태는 더 많은 책임을 가진 큰 클래스
- 상태는 변경할 수 있다.

구체상태(concreate state)객체를 활용해서 표현하는 것이 일반적, 태그 클래스보다는 sealed 클래스 계층으로 만든다. 

```kotlin
private var state: WorkoutState by
  Delegates.observable(states.first()) { _, _, _ ->
    updateView()
  }
```

## 정리
- 코틀린에서는 태그 클래스보다 타입계층을 사용하는 것이 좋다.
- 타입계층은 일반적으로 sealed 클래스를 사용한다.
