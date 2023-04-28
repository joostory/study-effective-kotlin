# 아이템 28: API 안정성을 확인하라

자동차마다 운전방법이 모두 다르다면 귀찮고 의미없는 시간이 많아질 것이다. 운전방법은 안정적이고 표준적인 것이 좋다.  
프로그래밍에서도 안정적이고 최대한 표준적인 API를 선호한다.

1. API가 변경되면 변경에 대응하거나 다른 대안을 찾는 것은 매우 어려운 일이다.
   - 라이브러리의 작은 변경에도 활용하는 코드는 많은 부분을 변경해야할 수 있다.
   - 이전 라이브러리를 사용하는 경우가 있다. 그럴수록 점점 업데이트가 어려워지고 버그와 취약성이 발생할 수 있다.
   - 안정적인 라이브러리로 업데이트하는 것을 두려워한다는 것은 매우 좋지 않은 상황이다.
2. 새로운 API를 배운다는 것은 꽤 힘들고 고통스러운 일이다.
   - 안정적이지 않은 모듈을 공부히는 것보다 안정적인 모듈을 공부하는 것이 좋다.

좋은 API를 한번에 설계할 수는 없다. 계속해서 개선을 해야하는데 불안정함을 명확히 알려주는 방법을 사용할 수 있다.  
일반적으로 [시멘틱 버저닝 시스템](https://simver.org/)을 이용해서 이를 알려줄 수 있다.

- MAJOR: 호환되지 않는 수준의 API 변경
- MINOR: 이전 버전과 호환되는 기능을 추가
- PATCH: 간단한 버그 수정

새로운 요소를 추가할 때 Experimental 어노테이션을 사용해서 아직 안정적이지 않다는 것을 알려줄 수 있다.

```kotlin
@Experimental(level = Experimental.Level.WARINING)
annotation class ExperimentalNewApi

@ExperimentalNewApi
suspend fun getUser(): List<User> { /* ... */ }
```

안정적인 API를 변경해야한다면 전환하는데 시간을 두고 Deprecated 어노테이션으로 알려줄 수 있다.
```kotlin
@Deprecated("use suspending getUsers", ReplaceWith("getUser()"))
fun getUsers(callback: (List<User>) -> Unit) { /* ... */ }
```

## 정리
- API가 안정적이지 않으면 수많은 사람들에게 고통을 줄 수 있다.
- 버전, 어노테이션, 문서 등의 커뮤니케이션이 중요하다.
- 변경을 가할때는 적응할 시간을 줘야 한다.
