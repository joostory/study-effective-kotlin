# 아이템 27: 변화로부터 코드를 보호하려면 추상화를 사용하라

추상화로 실질적인 코드를 숨기면 실질적인 코드를 원하는대로 수정할 수 있다.  
추상화를 통해 변화로부터 코드를 보호하는 행위는 자유를 가져온다.

## 상수
리터럴은 아무 것도 설명하지 않는다.
```kotlin
fun isPasswordValid(text: String): Boolean {
    if (text.length < 7) return false
    // ...
}
```
7은 '비밀번호의 최소 길이'를 나타내겠지만 코드를 이해해야 알 수 있다.
```kotlin
const val MIN_PASSWORD_LENGTH = 7
fun isPasswordValid(text: String): Boolean {
    if (text.length < MIN_PASSWORD_LENGTH) return false
    // ...
}
```
상수를 사용하면 의미를 훨씬 쉽게 이해할 수 있고 '비밀번호의 최소 길이'를 변경하기도 쉽다.

## 함수
```kotlin
Toast.makeText(this, message, Toast.LENGTH_LONG).show()
```
토스트 메시지를 추출하면 토스트를 출력하는 코드를 기억하지 않아도 된다.

```kotlin
fun Context.toast(
    message: String,
    duration: Int = Toast.LENGTH_LONG
) {
    Toast.makeText(this, message, duration).show()
}

toast("message")
```

만약 스낵바로 출력을 변경하고자 한다면 snackbar함수를 만들고 toast함수 대신 snackbar를 사용하도록 한꺼번에 수정하면 된다.

```kotlin
fun Context.snackbar(
    message: String,
    duration: Int = Snackbar.LENGTH_LONG
) {
    // ...
}
```
하지만 이렇게 함수의 이름을 직접 바꾸는 것은 위험할 수 있다. 다른 모듈이 이 함수를 쓰고 있다면 문제가 될 것이다.  
또한 함수이름을 바꾸긴 쉽지만 파라미터는 바꾸기 쉽지 않다.  
Toast.LENGTH_LONG -> Snackbar.LENGTH_LONG로 바꾸는 것도 문제를 발생시킬 수 있다.

메시지 출력방법이 바뀔 수 있다면 이제 중요한 것은 출력방법이 아니라 출력을 하고자 하는 의도다.  
토스트, 스낵바 개념과 무관한 showMessage라는 높은 레벨의 함수로 옮겨보자

```kotlin
fun Context.showMessage(
    message: String,
    duration: MessageLength = MessageLength.LONG
) {
    val toastDuration = when(duration) {
        SHORT -> Length.LENGTH_SHORT
        LONG -> Length.LENGTH_LONG
    }
    Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```
함수는 단순한 추상화지만 제한이 많다.
- 상태를 유지하지 않는다.
- 함수 시그니처 변경은 전체에 큰 영향을 준다.

## 클래스
```kotlin
class MessageDisplay(val context: Context) {
    fun show(
        message: String,
        duration: MessageLength = MessageLength.LONG
    ) {
        val toastDuration = when(duration) {
            SHORT -> Length.LENGTH_SHORT
            LONG -> Length.LENGTH_LONG
        }
        Toast.makeText(this, message, toastDuration).show()
    }
}

enum class MessageLength { SHORT, LONG }

// 사용
val messageDisplay = MessageDisplay(context)
messageDisplay.show("Message")
```
클래스가 함수보다 더 강력한 이유: 상태를 가질 수 있고, 많은 함수를 가질 수 있다는 점

```kotlin
// 클래스 생성 위임
@Inject lateinit var messageDisplay: MessageDisplay
// 의존하는 다른 클래스 테스트를 위한 mock
val messageDisplay: MessageDisplay = mockk()
// 다양한 종류의 메서드 생성
messageDisplay.setChristmasMode(true)
```
클래스가 더 많은 자유가 있지만 한계가 있다. final 클래스 아래에는 구현이 있다.  
open 클래스는 서비클래스를 만들 수 있어 좀 더 자유를 얻을 수 있다.  
더 많은 자유를 얻으려면 더 추상적이게 만들면 된다.

## 인터페이스
코틀린 라이브러리는 거의 모든 것이 인터페이스로 표현된다.

- listOf는 List인터페이스를 리턴하는 팩토리 메서드
- 컬렉션 처리 함수는 Iterable, Collection의 확장함수
- 프로퍼티 위임은 ReadOnlyProperty, ReadWriteProperty뒤에 숨겨진다

라이브러리를 만드는 사람은 내부 클래스의 가시성을 제한하고 인터페이스로 이를 노출한다.  
이렇게 하면 사용자가 클래스를 직접 사용하지 못하므로 인터페이스만 유지하면 원하는대로 구현을 변경할 수 있다.  
결합(coupling)을 줄일 수 있는 것이다.

또한 멀티플랫폼을 지원할 수 있다. 플랫폼에 따라 다른 구현을 리턴할 수 있다. 같은 인터페이스를 사용하기 때문이다.

```kotlin
interface MessageDisplay {
    fun show(
        message: String,
        duration: MessageLength = LONG
    )
}
enum class MessageLength { SHORT, LONG }

class ToastDisplay(val context: Context): MessageDisplay {
    override fun show(
        message: String,
        duration: MessageLength = MessageLength.LONG
    ) {
        val toastDuration = when(duration) {
            SHORT -> Length.LENGTH_SHORT
            LONG -> Length.LENGTH_LONG
        }
        Toast.makeText(this, message, toastDuration).show()
    }
}
```
이렇게 구성하면 더 많은 자유를 얻을 수 있다. 안드로이드, iOS, 웹에서 MessageDisplay를 사용할 수 있다.  
또한 테스트할때 클래스 모킹대신 인터페이스 페이킹을 사용할 수 있다.

```kotlin
val messageDisplay: MessageDisplay = TestMeessageDisplay()
```

## ID 만들기(nextId)
고유 ID를 사용하는 상황을 가정
```kotlin
var nextId: Int = 0
val newId = nextId++
```
- ID가 생성되는 방식을 변경할 때 문제가 된다.
- ID는 무조건 0부터 시작
- 스레드 세이프하지 않다.

```kotlin
private var nextid: int = 0
fun getNextId(): Int = nextId++
val newId = getNextId()
```
- 생성 방식 변경으로부터는 보호되지만 ID 타입 변경에는 대응하지 못한다.

```kotlin
data class Id(private val id: Int)
private var nextId: Int = 0
fun getNextId(): Id = Id(nextId++)
```
- 타입 변경에 대응하려면 클래스를 사용하는 것이 좋다.

이런 더 많은 추상화는 더 많은 자유를 주지만 정의하고 사용하고 이해하는 것이 더 어려워졌다.

## 추상화가 주는 자유
추상화의 방법
- 상수로 추출
- 동작을 함수로 래핑
- 함수를 클래스로 래핑
- 인터페이스 뒤에 클래스 숨김
- 보편적 객체를 특수한 객체로 래핑

이를 구현하는 도구
- 제네릭 타입 파라미터 사용
- 내부 클래스 추출
- 생성을 제한

## 추상화의 문제
추상화는 거의 무한하게 할 수 있지만 어느 순간 득보다 실이 많아진다.  
[FizzBuzz Enterprise Edition](https://github.com/EnterpriseQualityCoding/FizzBuzzEnterpriseEdition)는 이를 풍자한 프로젝트다. 원래 10줄도 필요하지 않은 예지만 61개의 클래스와 26개의 인터페이스가 있다.

추상화는 많은 것을 숨길 수 있는 테크닉이지만 너무 많은 것을 숨기면 결과를 이해하는 것 자체가 어려워진다.  
추상화된 함수를 오해해거나 실제 구현을 찾기 위해서 시간을 보낼 수도 있다.  
추상화를 이해하려면 예제를 살펴보는 것이 좋다.

## 어떻게 균형을 맞춰야 할까?
추상화의위치는 다음과 같은 요소들에 따라 달라질 수 있다. 
- 팀의 크기
- 팀의 경험
- 프로젝트의 크기
- 특징 세트(feature set)
- 도메인 지식

프로젝트에 따라서 균형이 다를 수 있다. 경험이 필요한 일이다.  
사용할 수 있는 규칙 몇가지를 정리하면 다음과 같다.
- 많은 개발자가 참여하는 프로젝트는 이후에 객체 생성과 사용방법을 변경하기 어렵다. 최대한 모듈과 파트를 분리하는 것이 좋다.
- 의존성 주입 프레임워크를 사용하면 복잡도를 신경쓰지 않아도 된다.
- 테스트를 하거나 다른 앱을 기반으로 새로운 앱을 만드는 경우
- 프로젝트가 작고 실험적이라면 추상화를 하지 않아도 된다.

## 정리
- 추상화는 중복을 제거하기 위한 것이 아니다.
- 추상화는 코드를 변경해야할 때 도움이 된다.
- 추상화는 균형을 찾아야 한다. 너무 많거나 적지 않도록.
