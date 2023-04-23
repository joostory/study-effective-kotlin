# 아이템 5 : 예외를 활용해 코드에 제한을 걸어라

확실하게 동작해야하는 코드에는 제한을 걸어주는 것이 좋다.

- require 블록 : 아규먼트 제한
- check 블록 : 상태 제한
- assert 블록 : true 확인 (테스트 모드에서만 동작한다)
- return, throw와 사용하는 elvis 연산자

```kotlin
fun pop(num: Int = 1): List<T> {
  require(num <= size) {
    "Cannot remove more elements than current size"
  }
  check(isOpen) { "Cannot pop from closed stack" }
  val ret = collection.take(num)
  collection = collection.drop(num)
  assert(ret.size == num)
  return ret
}
```

## 아규먼트
- require는 IllegalArgumentException을 발생시킨다.

## 상태
- check는 IllegalStateException을 발생시킨다.
- 일반적으로 require 뒤에 위치시킨다.

## Assert 계열 함수 사용
- assert는 테스트모드에서만 동작하므로 production에서 사용하려면 check를 사용하는 것이 좋다.

## nullability와 스마트 캐스팅
- 코틀린에서는 require, check로 조건을 확인해서 true라면 이후에도 true일 것이라고 가정한다.

