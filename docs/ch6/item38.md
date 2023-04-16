# 아이템 38: 연산 또는 액션을 전달할 때는 인터페이스대신 함수 타입을 사용하라

SAM(Single-Abstract Method): 연산 액션을 전달할때 사용하는 메서드가 하나만 있는 인터페이스
```kotlin
interface OnClick {
  fun clicked(view: View)
}

fun setOnClickListener(listener: OnClick) { /* ... */ }

setOnClickListener(object: OnClick {
  override fun clicked(view: View) { /* ... */ }
})
```

이 코드를 함수 타입을 사용하는 코드로 변경하면 더 많은 자유를 얻을 수 있다.
```kotlin
fun setOnClickListener(listener: (View) -> Unit) { /* ... */ }
```

람다 표현식 또는 익명함수로 전달
```kotlin
setOnClickListener { /* ... */ }
setOnClickListener(fun(view) { /* ... */ })
```

함수 레퍼런스 또는 제한된 함수 레퍼런스로 전달
```kotlin
setOnClickListener(::println)
setOnClickListener(this::showUsers)
```

선언된 함수 타입을 구현한 객체로 전달
```kotlin
class ClickListener: (View) -> Unit {
  override fun invoke(view: View) {
    /* ... */
  }
}
setOnClickListener(ClickListener())
```

SAM과 마찬가지로 함수타입도 type aliase를 사용하면 이름을 붙일 수 있다.
이로 인해 IDE의 지원도 받을 수 있다.
```kotlin
typealias OnClick = (View) -> Unit
fun setOnClickListener(listener: OnClick) { /* ... */ }
```


```kotlin
class CalendarView {
  var listener: Listener? = null

  interface Listener {
    fun onDateClicked(date: Date)
    fun onPageChanged(date: Date)
  }
}
```

API를 소비하는 사용자 관점에서 함수타입을 따로따로 갖는 것이 훨씬 사용하기 쉽다.
```kotlin
class CalendarView {
  var onDateClicked: ((date: Date) -> Unit)? = null
  var onPageChanged: ((date: Date) -> Unit)? = null
}
```

## 언제 SAM을 사용해야 할까?
다른 언어에서 사용할 클래스를 설계할 때는 SAM을 사용하는 것이 좋다.

```kotlin
class CalendarView {
  var onDateClicked: ((date: Date) -> Unit)? = null
  var onPageChanged: OnDateClicked? = null
}

interface OnDateClicked {
  fun onClick(date: Date)
}
```

다른언어에서 함수타입을 사용하려면 Unit을 명시적으로 리턴하는 함수가 필요하다.
```java
CalendarView c = new CalendarView();
c.setOnDateClicked(date -> Unit.INSTANCE)
c.setOnPageChanged(date -> {})
```
