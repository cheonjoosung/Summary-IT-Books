# Summary-Coroutine-Kotlin

## 1장 코틀린 코루틴을 배워야 하는 이유
### 개요
- 코틀린은 제시되고 수십년이 지나서야 현업에 적용되었음
- 약간의 수고로 코루틴 사용이 가능
- 멀티플랫폼에서 사용 가능

### 안드로이드(그리고 다른 프론트엔드 플랫폼)에서의 코루틴 사용
```kotlin
fun onCreate() {
    val new = getNewsFromApi()
    view.showNews(news)
}

fun onCreateWithThread() {
    thread {
        val new = getNewsFromApi()
        runOnUiThread {
            view.showNews(news)
        }
    }
}
```
- 쓰레드 전환 문제
  + 스레드 실행 시 멈출 수 없기에 메모리 누수
  + 스레드 많이 생성시 비용
  + 자주 전환 시 복잡도 증가시키며 관리가 어려움
  + 코드가 길어지며 이해하기 어려워 짐

### 콜백
- 함수를 논블로킹으로 만들고 작업 완료 시 콜백함수를 넘겨주는 것
```kotlin
fun onCreate() {
    getNewsFromApi { news -> 
        view.showNews(news)
    }
}
```
- 콜백 문제
  + 두 작업을 동시에 처리 불가능
  + 콜백을 취소 로직을 구현 시 많은 노력이 필요
  + 콜백지옥의 문제에 빠질 수 있음

### RxJava 와 리액티브 스트림
- 리액티브 스트림은 스레드 전환과 동시성 처리 지원하기에 병렬 처리하는데 사용
```kotlin
fun onCreate() {
    disposables += getNewsFromApi()
        .subscribeOn(Schedulers.io())
        .onserveOn(AndroidSchedulers.mainThread())
        .map { news ->
            news.sortedByDescending { it.publishedAt }
        }
        .subscribe { sortedNews ->
            view.showNews(sortedNews)
        }
}
```
- RxJava 문제
  + subcribeOn, observeOn, map 등 RxJava 를 익혀야 하고 데이터 클래스의 많은 래핑작업 필요
  + 많은 코드 작업과 함께 복잡스러운 면이 존재

### 코틀린 코루틴 사용
- 특정 지점에서 멈추고 이후에 재개 가능
- 스레드는 블로킹되지 않으며 뷰를 바꾸거나 다른 코루틴 실행하는 등의 또 다른 작업 가능
- 데이터 준비되면 코루틴은 메인 스레드에서 대기하고 있다가 메인 스레드가 준비되면 멈춘 지점에서 다시 작업 수행
```kotlin
fun onCreate() {
    viewModelScope.launch {
        val news = getNewsFromApi() //suspend method
        view.showNews(news)
    }
}

fun onCreateAsync() {
    viewModelScope.launch {
        val config = async { getConfigFromApi() } //suspend method
        val news = async { getNewsFromApi(config.await()) } //suspend method
        val user = async { getUserFromApi() } //suspend method
        view.showNews(news.await(), user.await())
    }
}
```
- await() 함수의 호출 결과를 기다림
- async 를 사용하면 이전의 결과를 기다리지 않고 병렬로 실행이 가능

### 백엔드에서의 코루틴 사용
- suspend 제어자를 추가하는 것으로 충분하며 쉽게 구현 가능
- 스레드보다 비용이 작음


## 2장 시퀀스 빌더
### 개요
- 코틀린의 시퀀스는 List or Set 과 같은 컬렉션이랑 비슷한 개념이지만 필요할 때마다 값을 하났기 계산하는 지연 처리
- 특징
  + 요구되는 연산 최소한 수행
  + 무한정
  + 메모리 사용 효율적
```kotlin
fun main() {
    for (num in seq) {
        println("number is $num")
    }
}

val seq = {
  yield(1)
  yield(2)
  yield(3)
}
```
- 작동방식
  + 첫 번째 수 요청 시 빌더 내부 진입
  + 작업이 끝나면 중단된 지점에서 다시 실행 됨 
  + main 함수와 제너레이터가 번갈아가면서 실행
  + 코루틴으로 동작하며 쓰레드 사용보다 효율적임 (비용)

### 실제 사용 예
- 피보나치 수열
  ```kotlin
  val fibonacci: Sequence<BigInteger> = sequence {
    var first = 0.toBigInteger()
    var second = 1.toBigInteger()
  
    while(true) {
        yield(first)
        val temp = first
        first += second
        second = temp  
    }
  }
  
  fun main() {
    print(fibonacci.take(10).toList())
  }
  ```
- 시퀀스 빌더는 중단 함수가 아니라 반환(yield)를 사용해야 함
- 중단 함수를 사용하고 싶은 경우 플로우 사용