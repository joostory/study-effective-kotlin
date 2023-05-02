# 아이템 30: 요소의 가시성을 최소화하라

- 가시성에 관련된 제한을 변경하면 코드를 사용하는 모든 부분이 영향을 받는다.
- 클래스의 상태를 외부에서 변경할 수 있다면 클래스는 자신의 상태를 보장할 수 없다.

```kotlin
class CounterSet<T> (
  private val innerSet: MutableSet<T> = setOf()
) : MutableSet<T> by innerSet {
  var elementsAdded: Int = 0
    private set

  override fun add(element: T): Boolean {
    elementsAdded++
    return innerSet.add(element)
  }

  override fun addAll(elements: Collection<T>): Boolean {
    elementsAdded += element.size
    return innerSEt.addAll(elements)
  }
}
```
- 일반적으로 코틀린에서는 접근자의 가시성을 제한해서 모든 프로퍼티를 캡슐화하는 것이 좋다.
- 만약 lazy ininitalize를 구현하려고 내부에 initialized 프로퍼티를 가지고 있을때 setter가 노출되면 안된다.
- 가시성이 제한될수록 클래스의 변경을 쉽게 추적할 수 있고 상태를 더 쉽게 이해할 수 있다.
- 동시성을 처리할 때 중요하다.

## 가시성 한정자 사용하기

클래스 멤버의 가시성 한정자
- public
- private: 클래스 내부
- protected: 클래스, 서브 클래스 내부
- internal: 모듈 내부

톱레벨 요소의 가시성 한정자
- public
- private: 같은 파일 내부
- internal: 모듈 내부

코트린에서의 모듈
- gradle 소스 세트
- maven 프로젝트
- intelliJ IDEA 모듈
- 한번의 ant 태스크로 컴파일되는 파일 세트

- DTO에는 이런 가시성 한정자를 사용하지 않는 것이 좋다.
- API를 상속하여 override할 때 가시성을 제한할 수 없다. 또 상속할 수 있기 때문. => 상속보다는 컴포지션
