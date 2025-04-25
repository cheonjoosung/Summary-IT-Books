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


## 3장 중단은 어떻게 동작할까?
### 재개
- 코루틴에서 중단가능한 함수 키워드로 suspend 사용
  ```kotlin
  suspend fun main() {
    println("before")
  
    suspendCoroutine<Unit> { }
  
    println("after")
  }
  ```
- 위 코드 실행 시 before 까지 출력된 상태로 유지 됨
  + after 를 출력하고 싶을 때 코루틴 코드 블록 안에서 resume 을 호출해서 복귀해야 함
- delay vs thread.sleep
  + delay 는 코루틴 내에서 동작 & 비동기(지연시간 동안 다른 작업 가능) & 코루틴을 정지하며 쓰레드를 차지하지 않음
  + thread.sleep 은 쓰레드를 차지 & 쓰레드를 중지
  
### 값으로 재개하기
- resume 함수에 Unit 을 전달할까?
  + 제너릭으로 된 기본타입이며 원하는 타입을 넣으면 그 값으로 리턴해야 함
  ```kotlin
  val i: Int = suspendCoroutine<Int> { cont: Continuation<Unit> -> 
    cont.resume(42)
  }
  println(i) // 42
  ```
  + 네트워크 상황에서 데이터를 요청하고 기다릴 때 사용하며 쓰레드에 비해 비용을 덜 씀. 안드로이드와 같은 메인 쓰레드 시스템에서는 엄청난 장점이 존재
  

### 예외로 재개하기
- 값 대신 exception 이 발생한 경우 예외를 던집니다
  + 호출한 곳에서 .isSuccessful 체크 또는 sealed 를 사용해서 처리

### 함수가 아닌 코루틴을 중단시킨다
- 탑 클래스 변수로 코루틴을 선언해두고 코드 내에 호출하는 것은 의미가 없음


## 4장 코루틴의 실제 구현
### 개요
- 중단 함수(suspend) 함수가 시작/중단 될 때 상ㅌ내를 가진다
- continuation 객체는 상태를 나타내는 숫자와 로컬 데이터를 가짐
  + 실행 재개 또는 완료 시 사용된느 콜 스택으로 사용
  + 비동기 흐름 제어용 콜백 핸들러이자 코루틴의 상태 저장
  ```kotlin
  suspend fun greet(): String {
    delay(1000)
    return "Hello"
  }
  
  fun greet(continuation: Continuation<String>): Any {
    // delay 이후 결과를 받아 다시 continuation.resumeWith(Result.success("Hello")) 호출
  }
  ```
  
### 컨티뉴에이션 전달 방식
- 중단 함수가 불리면 자동으로 continuation 객체가 생성 됨
  + 샘플
  ```kotlin
  interface Continuation<in T> {
    val context: CoroutineContext
    fun resumeWith(result: Result<T>)
  }
  
  suspend fun getUser(): User?
  fun getUser(continuation: Continuation<*>): Any? // 중단함수 내부 변형
  ```
  
### 아주 간단함 함수
- 지연함수를 통해 알아보기
  ```kotlin
  suspend fun myFunction() {
    println("before")
    delay(1000L)
    println("after")
  }  
  
  // 위 코드는 아래의 형태로 변환
  fun myFunction(continuation: Continuation<Unit>): Any {
    // 컴파일러가 생성한 상태 저장용 continuation 구현체
    val cont: MyFunctionContinuation = 
        if (continuation is MyFunctionContinuation) continuation
        else MyFunctionContinuation(continuation)

    when (cont.label) {
        0 -> {
            println("before")
            cont.label = 1
            // suspend point - delay 함수 호출, 중단
            return delay(1000L, cont)
        }
        1 -> {
            // delay 가 끝난 뒤 resume 됐을 때 실행
            println("after")
            return Unit
        }
    }
  }
  
  class MyFunctionContinuation(
      val completion: Continuation<Unit>
  ) : ContinuationImpl(completion) {
    var label = 0
    override fun invokeSuspend(result: Any): Any {
        return myFunction(this) // 상태 머신을 다시 호출하여 다음 단계 실행
    }
  }

  ```
  + continuation 의 상태(lable) 에 따라 before 이 출력되고 1초 지연 후 상태가 바뀌면 after 가 출력
  + delay(1000L, cont) 를 리턴시켜 재개될 시점을 알려줌

### 상태를 가진 함수
- counter 를 변화 시키는 함수가 있다고 가정
  ```kotlin
  suspend fun myFunction() {
    println("before")
    var count = 0
    delay(1000L)
    count++
    println("count=$count")
    println("after")
  }
  
  fun myFunction(continuation: Continuation<Unit>): Any {
    val cont: MyFunctionContinuation =
        if (continuation is MyFunctionContinuation) continuation
        else MyFunctionContinuation(continuation)

    var count = cont.count

    when (cont.label) {
        0 -> {
            println("before")
            count = 0
            cont.count = count
            cont.label = 1
            return delay(1000L, cont)
        }

        1 -> {
            count = cont.count
            count++
            cont.count = count
            println("count=$count")
            println("after")
            return Unit
        }

        else -> throw IllegalStateException("Invalid coroutine label")
    }
  }
  
  class MyFunctionContinuation(
  val completion: Continuation<Unit>
  ) : ContinuationImpl(completion) {
  var label = 0           // 현재 상태 추적
  var count = 0           // 지역 변수 count 저장
  
      override fun invokeSuspend(result: Any): Any {
          return myFunction(this)
      }
  }
  ```
  + continuation 객체가 상태와 지역 변수(count 저장) 을 가짐

### 값을 받아 재개되는 함수
- 중단 함수로부터 값을 받아야 하는 경우 더 복잡해짐
  ```kotlin
  suspend fun printUser(token: String) {
    println("before")
    val userId = getUserId(token) //중단함수
    println("Got userId: $userId")
    val userName = getUserName(userId, token) //중단함수
    println("Got userName: $userName")
    println("after")
  }
  
  // w
  fun printUser(token: String, continuation: Continuation<Unit>): Any {
    val cont: PrintUserContinuation =
        if (continuation is PrintUserContinuation) continuation
        else PrintUserContinuation(continuation)

    var userId: String
    var userName: String

    when (cont.label) {
        0 -> {
            println("before")
            cont.token = token
            cont.label = 1
            return getUserId(token, cont)
        }

        1 -> {
            userId = cont.result as String
            println("Got userId: $userId")
            cont.userId = userId
            cont.label = 2
            return getUserName(userId, cont.token, cont)
        }

        2 -> {
            userName = cont.result as String
            println("Got userName: $userName")
            println("after")
            return Unit
        }

        else -> throw IllegalStateException("Invalid state")
    }
  }

  class PrintUserContinuation(
      completion: Continuation<Unit>
  ) : ContinuationImpl(completion) {
  
      var label = 0
      lateinit var token: String
      lateinit var userId: String
      lateinit var result: Any
  
      override fun invokeSuspend(result: Any): Any {
          this.result = result
          return printUser(token, this)
      }
  }
  ```
  + token 상태 0과 1에서 사용
  + userId 상태 1과 2에서 사용
  + Result 타입인 result 함수가 어떻게 재개되었는지 나타냄

### 콜 스택
- 함수 a -> b 호출 시 b가 종료되면 실행이 될 지점을 어딘가 저장해야 하는데 이를 콜 스택이라는 자료 구조에 저장

### 실제코드
- 예외 발생 시 더 나은 스택 트레이스 생성
- 코루틴 중단 인터셉션
- 사용하지 않는 변수 제거하거나 테일콜 최적화 등 포함

### 중단함수 성능
- 비용이 크지 않으며 컨티뉴에이션 객체에 상태를 저장하는 것 또한 간단


## 5장 코루틴 언어차원에서의 지원 vs 라이브러리
- 코루틴에서 2가지 개념
  + 언어 자체 지원 부분(컴파일러의 지원과 코틀린 기본 라이브러리의 요소)
    * 컴파일러가 지원
    * kotlin.coroutines
    * 기본적인 것과 suspend 키워드 최소 제공
    * 직접 사용하기 어려움
    * 거의 모든 동시성 스타일이 허용
  + 코틀린 코루틴 라이브러리(kotlinx.coroutines)  rntjd
    * 별도 의존성 추가
    * kotlinx.coroutines
    * launch, async, Deferred 처럼 다양한 기능 제공
    * 직접 사용하기 편리하게 설계
    * 단 하나의 명확한 동시성 스타일을 위해 설계

## 6장 코루틴 빌더
- 개요
  + 모든 중단 함수는 또 다른 중단 함수에 의해 호출되고 일반 함수는 호출 불가
  + 중단 함수 연속 호출 시작 지점이 있는데 이를 코루틴 빌더가 역할 함
    * launch
    * runBlocking
    * async

### launch 빌더
- 작동방식은 thread 를 호출하여 새로운 스레드를 시작하는 것과 유사
  ```kotlin
  fun main() {
    GlobalScope.launch {
        delay(1000L)
        println("World!!")
    }
  
    Thread.sleep(2000L)
  }
  ```
  + launch 함수는 CoroutineScope 인터페이스의 확장 함수
  + CoroutineScope 인터페이스의 부모 코루틴과 자식 코루틴 사의 관계 정리하긴 위한 목적으로 사용되는 구조화된 동시성의 핵심
  + 쓰레드를 호출하는 이유는 코루틴 실행하자마자 끝나므로 대기시간을 위해 설정

### runBlocking 빌더
- 중단 메인 함수와 마찬가지로 시작한 스레드를 중단 시킵니다 (새로운 코루틴을 실행한 뒤 완료될 때까지 현재 스레드를 중단 가능한 상태로 블로킹)
  ```kotlin
  fun main() {
    runBlocking {
        delay(1000L)
        println("World")
    }
  }
  ```
  + delay(1000L) 은 Thread.sleep(1000L) 와 비슷하게 동작함 
  + 프로그램 끝나는 걸 방지하기 위해 스레드 블로킹할 필요가 있는 경우
  + 현재는 거의 사용되지 않음

### async 빌더
- launch 와 비슷하지만 값을 생성하도록 설계
- Deferred<T> 타입 객체를 리턴하며 T는 생성되는 값의 타입
- 값을 반환하는 중단 메서드인 await 이 있음
  ```kotlin
  fun main() = runBlocking {
    val res1 = GlobalScope.async {
        delay(1000L)
        "Text1"
    }
  }
  ```
  + launch 와 비슷하지만 async 는 값을 반환한다는 특징이 있음
  + 병렬로 작업을 실행시킬 때 유용

### 구조화된 동시성
- GlobalScope 에서 시작되었다면 프로그램은 해당 코루틴을 기다리지 않음
- launch or async 가 CoroutineScope 의 확장 함수
  + runBlocking 정의에 block 파라미터가 리시버 타입이 CoroutineScope 함수형 타입
  + 굳이 GlobalScope 사용하지 않고 runBlocking 에서 this 를 이용해서 launch or async 사용 가능
  ```kotlin
  fun main() = runBlocking {
    this.launch {
        delay(1000L)
        println("World!")
    }
  }
  ```
  + 부모는 자식들을 위한 스코프를 제공하고 자식들을 해당 스코프 내에서 호출합니다 이를 통해 구조화된 동시성 관계 성립
    * 자식은 부모로부터 컨텍스트 상속
    * 부모는 모든 자식이 작업을 마칠 떄까지 기다림
    * 부모 코루틴이 취소되면 자식 코루틴도 취소
    * 자식 코루틴 에러 발생 시 부모 코루틴 또한 에러로 소멸

### 현업에서의 코루틴
- 안드로이드 내에서는 scope 사용

### coroutineScope 사용
- 중단 함수 내에서 coroutineScope 사용
- 람다 표현식이 필요로 하는 스코프 만들어 주는 중단 함수
- coroutineScope 로 함수를 래핑한 다음 스코프 내에서 빌더를 사용
- 프로젝트 내에서 스코프가 정의되면 변경될 일이 거의 없음 ?


## 7장 코루틴 컨텍스트
- 개요
  + 코루틴 빌더에 보면 첫번째 파라미터가 코루틴 컨텍스트
  + 마지막 인자는 코루틴 스코프
    ```kotlin
    public interface CoroutineScope {
        public val coroutineContext: CoroutineContext 
    }
    ```
    * CoroutineContext 를 감싸는 래퍼처롬 보임 (Continuation 유사)

### CoroutineContext 인터페이스
- 원소나 원소들의 집합을 나타내는 인터페이스
  + Job, CoroutineName, CoroutineDispatcher 와 같은 Element 객체들이 인덱싱 된 집합
  + CoroutineContext 모든 원소는 CoroutineContext 되어있어서 직관적임
  + 연산자 + 로 쉽게 결합이 가능 
    ```kotlin
    val ctx = Dispatchers.IO + Job() + CoroutineName("MyCoroutine")
    ```

### CoroutineContext 에서 원소 찾기
- 컬렉션과 유사하기에 get 이용해 키를 가진 원소 찾을 수 있음
  ```kotlin
  fun main() {
    val ctx: CoroutineContext = CoroutineName("A Name")
    val coroutineName: CoroutineName? = ctx[CoroutineName]
  
    println(coroutineName?.name) // A Name
    val job: Job? = ctx[Job] // ctx.get(Job)
    printlnO(job) // null
  }
  ```
  + CoroutineName 은 Companion 객체로 되어있음


### CoroutineContext 더하기
- 두 개의 코루틴 컨텍스트를 합쳐서 하나의 컨텍스트로 만들 수 있음
  + 합쳐진 경우 두 가지 키를 모두 가짐
  + 같은 키를 가진 또 다른 원소가 더해지면 맵처럼 새로운 원소가 기존 원소를 대체합니다.

### 비어있는 코루틴 컨텍스트
- CoroutineContext 컬렉션이므로 빈 컨텍스트 또한 만들 수 있음
  + 비어잇는 경우 다른 컨텍스트 더해도 변하지 않음

### 원소제거
- 연산자(+)는 오버로딩 했2지만 - 연산자는 오버로딩 하지 않음
- minusKey 메소드를 사용해야 함

### 컨텍스트 폴딩
- fold 사용 조건
  + 누산기의 첫 번쨰 값
  + 누산기의 현재 상태와 현재 실행되고 있는 원소로 누산기의 다음 상태를 계산할 연산
  ```kotlin
  fun main() {
    val ctx = CoroutineName("Name") + Job
    ctx.fold("") { acc, element -> "$acc$element"}
        .also(::println)
  }
  ```

### 코루틴 컨텍스트와 빌더
- CoroutineContext 코루틴의 데이터를 저장하고 전달하는 방법
  ```kotlin
  fun CoroutineScope.log(msg: String) {
    val name = coroutineContext[CoroutineName]?.name
  println("[$name] $msg")
  }
  
  fun main() = runBlocking(CoroutineName("main")) {
    log("Started")
    
    val v1 = async {
        delay(500L)
        log("Running async")
        42
    }
  
    launch {
        delay(1000L)
        log("Running launch")
    }
  
    log("The answer is ${v1.await()}")
  }
  ```
  + 코루틴 컨텍스트 계산 공식
    * defaultContext + parentContext + childContext
    * 새로운 원소가 같은 키를 이전 원소를 대체 자식의 컨텍스트는 부모로부터 상속받은 컨텍스트 중 같은 키를 가진 원소로 대체

### 중단 함수에서 컨텍스트에 접근하기
- CoroutineScope 는 컨텍스트 접근할 때 사용하는 coroutineContext 접근

### 컨텍스트를 개별적으로 생성하기
- 코루틴 컨텍스트를 커스텀하게 만드는 경우는 흔치 않음
  + CoroutineContext.Element 인터페이스를 구현하는 클래스 만들기