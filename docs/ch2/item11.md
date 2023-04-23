# 아이템 11: 가독성을 목표로 설계하라

'코드를 작성하는데 1분이 걸리지만 읽는데는 10분이 걸린다'는 말이 있다.  
항상 가독성을 생각하면서 코드를 작성해야한다.

## 인식 부하 감소

```kotlin
// 구현 A
if (persion != null && persion.isAdult) {
  view.showPerson(person)
} else {
  view.showError()
}

// 구현 B
person?.takeIf { it.isAdult }
  ?.let(view::showPerson)
  ?: view.showError()
```

B가 더 간결하지만 A가 더 쉽게 이해할 수 있다.  
게다가 조건이 추가되거나 해야할 일이 추가되면 B도 더이상 간결하지 않게 된다.

```kotlin
// 구현 A
if (persion != null && persion.isAdult) {
  view.showPerson(person)
  view.hideProgressWithSuccess()
} else {
  view.showError()
  view.hideProgress()
}

// 구현 B
person?.takeIf { it.isAdult }
  ?.let {
    view.showPerson(person)
    view.hideProgressWithSuccess()
  } ?: run {
    view.showError()
    view.hideProgress()
  }
```

## 극단적이 되지 않기

let은 안전호출을 위해서, 연산을 아규먼트 처리 후로 이동시킬때, 데코레이터를 사용해서 객체를 랩할때 사용할 수 있다. let의 사용은 비용을 발생시킨다. 하지만 비용을 발생시킬 가치가 있다면 사용해도 좋다. 균형을 맞추는 것이 중요하다.

## 컨벤션

사람에 따라 가독성에 대한 관점이 다르므로 이에 대해 토론을 하고 규칙을 정한다.
