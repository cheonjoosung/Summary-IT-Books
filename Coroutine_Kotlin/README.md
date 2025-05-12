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


## 8장 잡과 자식 코루틴 기다리기
- 개요
  + Job 은 코루틴 취소, 상태 파악 등 다양하게 사용 가능

### Job 이란 무엇인가?
- Job 은 수명을 가지고 있으면 취소가능함
- 인터페이스긴 하지만 구체적인 사용법과 상태를 가지고 있다는 점에서 추상클래스 처럼 다룰 수 있음
- 상태변화
  + New -> Active -> Completing -> Completed
  + Active/Completing -> Cancelling -> Canceled
  + 대부분의 코루틴은 Active 에서 시작하고 지연 코루틴만 New 상태에서 실행
  + isActive, isCompleted, isCancelled 프로퍼티 사용 가능

### 코루틴 빌더는 부모의 잡을 기초로 자신들의 잡을 생성한다
- launch 반환 타입은 job
  ```kotlin
  fun main(): Unit = runBlocking {
    val job: Job = launch {
        delay(1000L)
        println("test")
    }
  }
  ```
- async 함수 반환 타입은 job
  ```kotlin
  fun main(): Unit = runBlocking {
    val deferred: Deferred<String> = async {
        delay(1000L)
        "test"
    }
    val job: Job = deferred
  }
  ```
- Job 은 코루틴에서 상송하지 않는 유일한 코루틴 컨텍스트이며 이는 코루틴에서 중요한 법칙
- 모든 코루틴은 자신만의 Job을 생성하며 인자 또는 부모 코루틴으로부터 온 잡은 새로운 잡의 부모로 사용
  ```kotlin
  fun main(): Unit = runBlocking {
    val name = CoroutineName("Some name")
    val job = Job()
  
    launch(name + job) {
        val childName = coroutinecontext[CoroutineName]
        println(childName == name) // true
        val childJob = coroutineContext[Job]
        println(childJob == job) // false
        println(childJob == job.children.first()) // true
    }
  
  }
  ```
  
### 자식들 기다리기
- join()
  + 지정한 잡이 Completed or Canceled 될 때 까지 중단하는 함수

### 잡 팩토리 함수 
- job 은 Job() 팩토리 함수 사용 시 코루틴 없이 Job 만들 수 있음
  + 어떤 코루틴과도 연관되지 않으며
  + 컨텍스트로 사용될 수 있음
  + 한 개 이상의 자식 코루틴을 가진 부모 잡으로 사용할 수도 있음
  ```kotlin
  suspend fun main(): Unit = runBlocking {
    val job = Job()
    launch(job) { // 새로운 잡이 부모로부터 상속받은 ㅈ밥을 대체
        delay(1000L)
        println("Text 1")
    }
  
    launch(job) { // 새로운 잡이 부모로부터 상속받은 ㅈ밥을 대체
        delay(2000L)
        println("Text 2")
    }
  
    // job.join() 이렇게 쓰면 영원히 대기함
    job.children.forEach { it.join() }
  }
  ```
  + 생성자처럼 보이지만 인터페이스라 가짜 생성자임
  + 하위인터페이스인 CompletableJob 이 반환
    * 2가지 메소드를 추가하여 기능성 확장
    * complete(): Boolean 잡 완료하는데 사용
    * completeExceptionally(exception: Throwable): Boolean 인자로 받은 예외로 잡을 완료하는데 사용


## 9장 취소
- 개요
  + 중단 함수 사용하는 몇몇 클래스와 라이브러리는 취소 기능 반드시 제공
  
### 기본적인 취소
- Job 인터페이스 취소하게 하는 cancel 메서드 가짐
  + 호출한 코루틴 첫 번째 중단점에서 잡을 끝냄
  + 잡이 자식을 가지면 그ㄷ들 또한 취소 되나 부모는 영향을 받지 않음
  + 잡이 취소되면 취소된 잡은 새로운 코루틴의 부모로 사용될 수 없습니다. 취소된 잡은 Cancelling 상태가 되었다가 Cancelled 상태로 바뀜
  ```kotlin
  suspend fun main(): Unit = coroutineScope {
    val job = launch {
        repeat(1000) { i ->
            delay(200)
            println("Printing $i")
        } 
    }
  
    delay(1100)
    job.cancel()
    job.join()
    println("Cancelled successfully")
  }
  // Printing 0
  // Printing 1
  // Printing 2
  // Printing 3
  // Printing 4
  // Cancelled successfully
  ```
  + cancel -> join 을 호출해서 기존의 작업을 기다려야 함
  ```kotlin
  public suspend fun Job.cancelAndJoin() {
    cancel()
    return join()
  }
  ```

### 취소는 어떻게 동작하는가?
- 잡이 취소되면 Cancelling 상태로 바뀝니다. 상태가 바뀐 뒤 첫 번째 중단점에서 CancellationException 예외를 던집니다
  + try-catch 구문을 사용하여 잡을 수도 있지만 다시 던지는 것이 좋습니다.

### 취소 중 코루틴을 한 번 더 호출하기
- Job은 이미 Cancelling 상태가 되면 중단되거나 다른 코루틴을 시작하는 건 절대 불가능
- 반드시 호출해야 하는 케이스는 DB롤백 하는 경우 : withContext(NonCancellable)
  + withContext 내부에서는 취소될 수 없는 Job인 NonCancellable 객체를 사용
  + 블록 내부에서 잡은 앩티브 상태를 유지하며 중단 함수를 원하는 만큼 호출

### invokeOnCompletion
- 자원 해제하는 데 자주 사용되는 또 다른 방법은 Job의 invokeOnCompletion 메소드를 호출
- invokeOnCompletion 메서드는 잡이 Completed or Cancelled 와 같은 마지막 상태에 도달했을 때 호출된 핸들러를 지정하는 역할
  ```kotlin
  suspend fun main(): Unit = coroutineScope {
    val job = launch {
        delay(1000)
    }
    job.invokeOnCompletion { exception: Throwable? -> 
        println("Finished")
    }
    delay(400)
    job.cancelAndJoin()
  }
  ```
  + 잡이 예외 없이 끝나면 null
  + 코루틴 취소되었으면 CancellationException 이 됨
  + 코루틴을 종료시킨 예외일 수 있습니다

### 중단될 수 없는 걸 중단하기
- 취소는 중단점에서 일어남
- Thread.sleep() 을 절대 사용하지 않는 것이 좋음

### suspendCancellableCoroutine
- CancellableContinuation<T> 로 래핑

## 10장 예외 처리
- 개요
  + 스레드의 경우도 예기치 않은 에러가 발생한 경우 종료 함
  + 코루틴 빌더는 부모도 종료시키며 취소된 부모는 자식들 모두를 취소시킨다는 점이 스레드와 다름
  + 예외는 자식 -> 부모로 가며 다시 부모 -> 자식으로 쌍방 전파하는 구조

### 코루틴 종료 멈추기
- 코루틴 내에서 try-catch 는 아무런 도움이 되지 않음
- SupervisorJob
  + 해당 키워드를 사용하면 자식에서 발생한 모든 예외룰 무시할 수 있음
  + 부모 코루틴의 인자로 사용하게 되면 쓰지않는 것과 동일하므로 주의가 필요함
- supervisorScope
  + 부모와의 연결은 유지되지만 부모에게 예외를 전파하지 않음
  + 중단 함수 본체를 래핑
  + 서로 무관한 다수의 작업을 해당 스코프내에서 실행을 주로 함
- coroutineScope
  + try-catch 로 던지는 예외를 잡을 수 있음

## await
- launch 처럼 부모 코루틴을 종료하고 부모와 관련있는 코루틴 빌더도 종료시킴
- 그러나 supervisorScope 와 같이 사용하면 해당 코루틴빌더만 종료되고 나머지는 그대로 실행 됨

## CancellationException은 부모까지 전파되지 않는다
- 해당 클래스의 서브 클래스라면 부모로 전파되지 않고 현재 코루틴을 취소시킴
- open Class 로 다른 클래스나 객체로 확장 가능

## 코루틴 예외 핸들
- 예외를 다룰 때 예외를 처리하는 기본 행동을 정의하는 것이 유용할 때가 있음
- coroutineExceptionHandler 컨텍스트를 사용하면 편리함
  ```kotlin
  fun main(): Unit = runBlocking {
    val handler = CoroutineExceptionHandler { ctx, exception ->
        println("Caught $exception")
    } 
  }
  ```


## 11장 코루틴 스코프 함수
### 코루틴 스코프 함수가 소개되기 전에 사용한 방법들
  ```kotlin
  suspend fun getUserProfile(): UserProfileData {
    val user = getUserData()                // 1초
    val notifications = getNotifications() // 1초
  
    return UserProfielData(user = user, notifications = notifications)
  }
  ```
  + 문제 : 작어빙 동시에 진행되지 않는다는 점(하나의 엔드포인트에서 데이터를 얻는데 1초씩 걸리기 때문에 함수기ㅏ 끝나는 데 2초가 걸림)
  + 두 개의 중단 함수를 동시에 실행하려면 async 로 래핑필요하고 GlobalScope 사용은 바람직하지 않음
    * GlobalScope 는 그저 EmptyCoroutineContext 가진 스코프임
    * 글로벌 스코프로 aysnc 실행 시 부모 코루틴과 아무런 관련이 없어짐
      + 취소가 불가능해짐
      + 부모로부터 스코프 상속받지 않음
      + 메모리 누수발생
      + 단위 테스트 도구 동작하지 않아 테스크 수행 어려움

### coroutineScope
- 스코프를 시작하는 중단 함수이며 인자로 들어온 함수가 생성한 값을 반환
  ```kotlin
  suspend fun <R> coroutineScope(
    block: suspend CoroutineScope.() -> R
  ): R
  ```
  + 코루틴 스코프 본체는 리시버 없이 곧바로 호출
  + 새로운 코루틴을 생성하지만 새로운 코루틴이 끝날 때까지 코루틴 스코프를 호출한 코루틴을 중단하기 때문에 호출한 코루틴 작업을 동시에 시작하지 않음
  + 생성된 스코프는 바깥의 스코프에서 coroutineContext 를 상속받지만 컨텍스트의 Job을 오버라이딩합니다
    * 부모가 해야할 책임을 이어받음
    * 자신의 작업 끝내기 전까지 모든 자식 기다림
    * 부모 취소시 자식도 취소
  ```kotlin
  suspend fun getUserProfile(): UserProfileData = 
    coroutineScope {
        val user = async { getUserData() }                // 1초
        val notifications = async { getNotifications() } // 1초
  
        UserProfielData(user = user.await(), notifications = notifications.await())
    }
  ```
  
### 코루틴 스코프 함수
- supervisorScope 는 coroutineScope 와 비슷하지만 Job 대신 SupervisorJob 사용
- withContext 는 커ㅗ루틴 컨텍스트를 바꿀 수 있는 coroutineScope
- withTimeOut 은 타임아웃이 있는 coroutineScope
- 스코프 함수
  + let, with, apply
- 코루틴 스코프 함수
  + 스코프 함수와 확연히 구분 
  + 코루틴 빌더와 혼동되지만 개념적으로 다름
  ## 📌 코루틴 빌더 vs 코루틴 스코프 함수

| 구분 | 코루틴 빌더 (Coroutine Builders) | 코루틴 스코프 함수 (Coroutine Scope Functions) |
|------|----------------------------------|---------------------------------------------|
| **정의** | 코루틴을 **생성**하고 실행하는 함수 | 코루틴이 실행되는 **범위(scope)**를 지정하는 함수 |
| **대표 함수** | `launch`, `async`, `withContext` | `coroutineScope`, `supervisorScope` |
| **목적** | 실제로 코루틴을 시작하고 수행함 | 코루틴의 구조와 자식 코루틴의 동작 방식을 관리 |
| **반환값** | `launch` → Job<br>`async` → Deferred<br>`withContext` → 결과 값 | 결과 값 반환 (suspend 함수) |
| **컨텍스트 전환 여부** | `withContext`만 컨텍스트 전환<br>나머지는 그대로 실행 | 현재 컨텍스트를 유지하며 새로운 스코프 생성 |
| **에러 전파** | 기본적으로 부모에게 예외 전파 | `coroutineScope` → 예외 전파<br>`supervisorScope` → 예외 격리 |
| **사용 예시** | ```kotlin<br>launch { doSomething() }<br>async { getResult() }``` | ```kotlin<br>coroutineScope {<br>   launch { ... }<br>}``` |
| **기본 특성** | 새 Job을 만들어 실행<br>병렬 작업 가능 | 자식 코루틴들이 모두 끝날 때까지 suspend됨 |
| **종료 조건** | Job이 cancel되면 중단 | 스코프 내 모든 코루틴 완료 시 반환 |

### withContext
- coroutineScope 와 비슷하지만 스코프의 컨텍스트를 변경할 수 있다는 점이 다름
  ```kotlin
  launch(DIspatchers.Main) {
    view.showProgressBar()
    withContext(Dispatchers.IO) {
        fileRepostiory.saveData(data)
    }
    view.hideProgressBar()
  }
  ```
  + Dispatchers 와 주로 쓰임
  + async 는 스코프를 필요로 하지만 withContext 와 coroutineScope 는 해당 함수를 호출한 중단점에서 스코프 들고옴

### supervisorScope
- 호출한 스코프로부터 상속받은 coroutineScope를 만들고 지정된 중단 함수를 호출한다는 점에서 coroutineScope 와 비슷
- 둘의 차이점은 Job을 SupervisorJob 으로 오버라이딩하는 것이기 때문에 자식 코루틴이 예외를 던지더라도 취소되지 않음
- 서로 독릭적인 작업을 시작하는 함수에서 주로 사용
- async 로 처리하려면 모든 작업마다 try-catch 문으로 예외를 잡아야 하는 문제가 있음

### withTimeout
- coroutineScope 와 비슷한 함수
- 실행할 때 제한시간을 두는것이 다른 점
- 함수 테스트 시 유용하게 쓰임. 얼마나 오래걸리는지 적게 걸리는지 확인 시

### 코루틴 스코프 함수 연결하기
- 서로 다른 코루틴 스코프 함수의 두가지 기능이 모두 필요하다면 코루틴 스코프 함수에서 다른 기능을 가지는 코루틴 스코프 함수 호출
  ```kotlin
  suspend fun calculateAnswerOrNull(): User? = 
    withContext(Dispatchers.Default) {
        withTimeoutOrNull(1000) {
            calculateAnswer()
        }
    }
  ```

### 추가적인 연산
- 안드로이드에서 뷰와 상관없는 로직이 있는 경우 launch 보다는 생성자 주입을 통해 전달해줘서 기다리지말고 다른 작업을 수행하도록 만드는것이 좋음
  ```kotlin
  class ShowUserDataUseCase(
    val repo: UserDataRepository,
    val view: UserDataView,
    val analyticsScope: CoroutineScope //CoroutineScope(SupervisorJob())
  ) {
    suspend fun showUserData() = coroutineScope {
        val name = async { repo.getName() }
        val friends = async { repo.getFriends() }
        val profile = asysc { rpo.getProfile() }
        val user = User(name.await(), frineds.await(), profile.await())
  
        view.show(user)
        analyticsScope.launch { repo.notifyProfileShown() } // launch { repo.notifyProfileShown() } 보다 좋음
    }
  }
  ```
  - 취소도 가능하며
  - 중요하지 않은 작업을 대기하지 않아도 됨


## 12장 디스패처
- 개요
  + 코루틴이 실행되어야 할 스레드를 결정할 수 있는데 이 때 디스패처를 이용함
  + RxJava 의 스케줄러와 비슷한 개념?

### 기본 디스패처
- CPU 집약적인 연산을 수행하도록 설계된 Dispatchers.Default
- CPU 개수와 동일한 수의 스레드 풀을 가짐
  ```kotlin
  suspend fun main() = coroutineScope {
    repeat(1000) {
        launch {
            List(1000) { Random.nextLong() }.maxOrNull()
            val threadName = Thread.currentThread().name
            println("name $threadName")
        }
    }
  }
  ```
  
### 기본 디스패처를 제한하기
- limitedParallelism(Int) 사용하면 특정 수 이상의 스레드 사용못하도록 막을 수 있음
  + 모든 스레드를 다 써버리는 상황을 막기위한 용도
  + kotlinx.coroutines 1.6 도입

### 메인 디스패처
- Dispatchers.Main 은 복잡한 연산이 필요하지 않을 때 이용
  + I/O 또는 네트워크 작업 시에는 추천하지 않음

### IO 디스패처
- Dispatchers.IO 파일 읽고 쓰는 경우 등 스레드를 블로킹할 때 사용하기 위해 설계
  + Dispatchers.Default 와 같은 스레드 풀을 공유함
  + Dispatchers.Default -> withContext(Dispatchers.IO) 로 변환되면 Default 쓰레드 한도가 아닌 IO 쓰레드 한도로 적용

### 커스텀 스레드 풀을 사용하는 IO 디스패처
- Dispatchers.IO = pool.limitedParallelism(64)
  + 스레드 블로킹하는 경우가 잦은 클래스에서 자기만의 한도를 가진 커스텀 디스패처 정의

### 정해진 수의 스레드 풀을 가진 디스패처
- ExecutorService.asCoroutineDispatcher() 로 만들어진 디스패처의 가장 큰 무넺점은 close 함수로 닫아야 함

### 싱글스레드로 제한된 디스패처
- 다수의 스레드가 변수 공유로 인한 문제
  ```kotlin
  var i = 0
  
  suspend fun main(): Unit = coroutineScope {
    repeat(10_000) {
        launcH(Dispatchers.IO) {
            i++
        }
    }
    delay(1000)
    println(i) // ~9930 컴퓨터마다 다를 수 있음
  }
  ```
  + Executors 를 사용하여 싱글스레드 디스패처 만들어야 함
  ```kotlin
  val dispatcher = Executors.newSingleThreadExecutor().asCoroutineDispatcher()
  ```
  + limitedParallelism(1) 를 사용도 가능하나 블로 킹되면 순차적으로 처리되는 것이 단점

### 프로젝트 룸의 가상 스레드 사용하기
- JVM 플랫폼은 프로젝트 Loom 이라는 기존 쓰레드보다 가벼운 가상 스레드 도입 함
  + JVM 19 이상부터 사용가능하며 안정화전까지 엔터프라이즈 애플리케이션 적용은 미루는 것이 좋음 

### 제한받지 않는 디스패처
- Dispatchers.Unconfined 스레드를 바꾸지 않는다
  + 단위 테스트 시 유용

### 메인 디스패처 즉시 옮기기
- 코루틴 배정도 비용이 듬
- withContext 호출 시 코루틴 중단되고 큐에서 기다리다가 재개함
- 메인 스레드를 기다리는 큐가 쌓여있다면 withContext 때문에 사용자 데이터는 약간 지연뒤에 보여질것임
  + Dispatchers.Main.immediate 호출하면 즉시 실행 됨

### 컨티뉴에이션 인터셉터
- 디스패칭은 코틀린 언어세ㅓ 지원하는 컨티뉴에이션 인터셉션을 기반으로 작동
  + ContinuationInterceptor 라는 코루틴 컨텍스트는 코루틴이 중단 시 interceptContinuation 메서드로 컨티뉴에시녀 객체를 수정하고 포함

작업의 종류에 따른 각 디스패처의 성능 비교
- 중요사항
  + 단지 중단할 경우에는 사용하고 있는 스레드 수가 얼마나 많은지는 문제가 되지 않음
  + 블로킹 경우 스레드 많은수록 코루틴 종료시간이 빨라짐
  + CPU 집약적인 연산에서는 Dispatchers.Default 가 가장 좋은 선택지
  + 메모리집약적인 연산을 처리하고 있다면 많은 스레드 사용하는 것이 좀 더 낫긴하나 별 차이 없음


## 13장 코루틴 스코프 만들기
- 안드로이드/백엔드 개발 시 많이 사용

### CoroutineScope 팩토리 함수
- CoroutineScope 는 coroutineContext 를 유일한 프로퍼티로 가지고 있는 인터페이스
  ```kotlin
  interface CoroutineScope {
    val coroutineContext: CoroutineContext
  }
  ```
- CoroutineScope 인터페이스를 구현한 클래스를 만들고 내부에서 코루틴 빌더 직접 호출 가능
  ```kotlin
  class SomeClass : CoroutineScope {
    override val coroutineContext: CoroutineContext = Job()
  
    fun onStart() { 
        launch {
            //...
        }
    }
  }
  
  class SomeClass {
    val scope: CoroutineScope = ...
  
    fun onStart() { 
        scope.launch {
            //...
        }
    }
  }
  ```
  + 위 방식보다 아래 방식 선호 (위의 경우 전체 스코프 취소시 코루틴 시작 불가능하다)
- CoroutineScope 팩토리 함수
  ```kotlin
  public fun CoroutineScope(context: CoroutineContext) : CoroutineScope =
    ContextScope(
        if (context[job] != null) context
        else context + Job()
    )
  ```
  
### 안드로이드에서 스코프 만들기
- onCreate() 를 통해 MainViewModel 이 데이터 가져오는 경우
  + BaeViewModel 스코프 단 한번으로 정의
  ```kotlin
  abstract class BaseViewModel(
    private val onError: (Throwable) -> Unit
  ): ViewModel() {
  
    private val exceptionHandler = CoroutineExceptionHandler { _, throwable ->
        onErro(throwable)
    }
  
    private val context = Dispatchers.Main + SupervisorJob() + exceptionHandler
  
    //protected val scope = CoroutineScope(Dispatchers.Main + Job())
    protected val scope = CoroutineScope(context)
  
    override fun onCleared() {
        //scope.cancel() 스코프 전체취소
        context.cancelChildren() // 자식 취소
    }
  }
  
  class MainViewModel(
    private val userRepo: UserRepository,
    private val newsRepo: NewsRepository
  ) : BaseViewModel {
    fun onCreate() {
        scope.launch {
            val user = userRepo.getUser()
            view.showUserData(user)
        } 
        scope.launch {
            val news = newsRepo.getUser()
            view.showNews(user)
        } 
    }
  }
  ```
  + 메인 쓰레드가 많은 수의 함수를 호출하기에 Dispatchers.Main 사용
  + onDestroy() 시 취소 가능하게 만들기 위해서 Job() 사용 (명시적)
  + 사용자 데이터 가져올 때 에러가 발생하더라더 뉴스를 봐야하므로 독립적인 SupervisorJob() 사용

### viewModelScope & lifecycleScope
- lifecycle-viewmodel-ktx 2.2.0 or lifecycle-runtime-ktx 2.2.0 이상 버전 필요
  + 특정 컨텍스트가 필요없다면 해당 스코프를 쓰는것이 편리하고 좋음

### 백엔드에서 코루틴 만들기
- Pass

### 추가적인 호출을 위한 스코프 만들기
- 추가적인 연산을 시작하기 위한 스코를 만드는데 함수나 생성자의 인자를 통해 주로 주입
  + 예외를 관제 시스템으로 보내고 싶으면 CoroutineExceptionHandler 사용


## 14장 공유 상태로 인한 문제
- 개요 
  ```kotlin
  suspend fun fetchUser(id: Int) {
    val newUser = api.fetchUser(id)
    users.add(newUser)
  }
  ```
  + 같은 시간에 두 개 이상의 스레드에서 함수가 호출될 수 있으므로 보호되어야 함
  + list 추가 되어서 사이즈 100인데 다른 스레드에서 99로 참조시 충돌이 발생할 수도 있음

### 동기화 블로킹
- Java 에서는 전통젃으로 synchronized 블록이나 동기화된 컬렉션 사용하여 해결함
  ```kotlin
  var count = 0
  
  fun main() = runBlocking {
    val lock = Any()
    massiveRun {
        sysnchronized(lock) {
            count++
        } 
    }
    println("Count=$count")
  }
  ```
  + 동기화 블록 내부에서 중단이 불가능 함
  + 동기화 블록에서 코루틴이 자기 차례를 기다릴 떄 스레드를 블로킹한다는 것

### 원자성
- 원자성 연산
  + 락 없이 로우 레벨로 구현되어 효율적이고 사용하기 쉬움
  + 빠르게 스레드 안전을 보장, AtomicInteger
  ```kotlin
  var count = AtomicInteger()
  
  fun main() = runBlocking {
    val lock = Any()
    massiveRun {
        count.incrementAndGet()
    }
    println("Count=${count.get()}")
  }
  ```
  + 하나의 연산에서 원자성을 가지지 전체 연산에서 원자성을 가지지 않음
- AtomicReference 래핑
  ```kotlin
  class UserDownloader(
    private val api: NetworkService
  ) {
    private val users = AtomicReference(listOf<User>())
  
    fun downloaded(): List<User> = users.get() // 원본 유지
  
    suspend fun fetchUser(id: Int) {
        val newUser = api.fetchUser(id)
        users.getAndUpdate { it + newUser }
    }
  }
  ```
  + getAndUpdate 원자성 보장 함수를 사용해 충돌 없이 값 갱신

### 싱글스레드로 제한된 디스패처
- 병렬성을 하나의 스레드로 제한하는 것인데 두 가지 방법으로 가능
  + 코스 그레인 스레드 한정
    * 디스패처를 싱글스레드로 제한한 withContext 로 전체함수를 래핑하는 방법
    * 사용하기 쉬우며 충돌 방지 가능하나 멀티스레딩 이점 누리지 못함 
  + 파인 그레인드 스레드 한정
    * 상태를 변경하는 구문들만 래핑
    * 코스 그레인보다 번거롭지만 크리티컬 섹션이 아닌 부분이 블로킹되거나 GPU 집약적인 경우에 더 나은 성능 제공

### 뮤텍스
- 단 하나의 열쇠가 있는 방이라 생각
  + 핵심 기능은 lock
  + 첫 번째 코루틴이 lock 호출하면 열쇠를 가지고 중단없이 실행
  + 두 번째 코루틴이 lock 호출하면 첫 번쨰 코루틴 unlock 까지 중단
  + lock & unlock 직접 사용하는 건 위험함. 에외발생 시 열쇠를 못받기에 deadLock 상태에 빠질 수 있다.
  + lock 시작해 finally 블록체엇 unlock 을 호출하는 withLock 함수 사용
  ```kotlin
  val mutex = Mutex()
  mutex.withLock {
  }
  ```
  + 위 방식은 스레드 블로킹 대신 코루틴 중단시키기에 조금 더 안전하고 가벼운 방식

### 세마포어
- 둘 이상이 접근할 수 있는 방법
  + acquire, release, withPermit 함수 가짐
  + 공유 상태 해결할 수 없지만 동시 요청을 처리할 수를 제한할 떄 사용할 수 있어 처리율 제한 장치 구현 떄 도움이 됨
  