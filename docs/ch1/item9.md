# 아이템 9 : use를 사용하여 리소스를 닫아라

더이상 필요하지 않을때 close메소드를 사용해서 명시적으로 닫아야 하는 리소스가 있다.  
코틀린/JVM에서는 AutoCloseable을 상속받는 Closeable인터페이스를 구현한다.  
리소스에 대한 레퍼런스가 없어질 때 가비지 컬렉터가 처리한다. 이는 굉장히 느리고 쉽게 처리되지 않는다.  
따라서 명시적으로 close를 호출하는 것이 좋다.

use를 사용하면 이를 쉽게 처리할 수 있다.

```kotlin
var reader = BufferedReader(FileReader(path))
try {
  return reader.lineSequence().sumBy { it.length }
} finally {
  reader.close()
}

// use는 Closeable객체의 close를 호출해준다.
BufferedReader(FileReader(path)).use {
  return reader.lineSequence().sumBy { it.length }
}

// useLines를 사용하면 한줄씩 읽어 더 리소스를 절약할 수 있다.
File(path).useLines { lines ->
  return lines.sumBy { it.length }
}
```

