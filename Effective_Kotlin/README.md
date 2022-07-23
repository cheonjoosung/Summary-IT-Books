# Summary-Effective-Kotlin

## 1. 좋은코드
### 1-1. 안정성
- 가변성을 제한하라
    + val 읽기전용 property 사용
      * get() 만 제공함
      * 재할당이 불가능한 것(읽기전용)의 개념과 값이 변할 수 있다는 것(가변성)은 다른 의미
      ```kotlin
      val list = mutalbeListOf(1, 2, 3)
      list.add(4) 
      list = mutalbeListOf(3) // compile Error
      ```
    + 가변 컬렉션과 읽기 전용 컬렉션 구분
      * Iterable, Collection, List 인터페이스는 읽기 전용
      * mutable~로 시작하는 인터페이스는 읽기/쓰기가 가능
      * 읽기 전용으로 선언하고 복제를 통해 mutable 컬렉션 이용
      ```kotlin
      val list = listOf(1,2,3)
      val mutableList = list.toMutableList()
      ```
    + data class copy
      ```kotlin
      data class Person(val name: String, val age: Int)
      
      val user1 = User("first", 1)
      val user2 = user.copy(name = "second")
      ```
<br></br>
- 변수의 스코프를 최소화하라
    + 프로그램 추적/관리에 용이함. 많은 요소가 생겨나면 코드를 분석하고 수정할 때
    변경부분이 많아지고 이는 프로그램을 이해하기 어려워질 수 있음
      ```kotlin
      val users = listOf(1, 2, 3)
      
      // 나쁜 예
      val user1: User
      for (i in users.indices) {
          user1 = users[i]
          println("User at $i is $user1")
      }
      
      // 좋은 예
      for ( (i, user3) in users.withIndex()) {
          println("User at $i is $user3")
      }
      ```
    + 여러 프로퍼티 한꺼번의 설정할 경우 (if 보다는 when 사)
      ```kotlin
      fun update(degree: Int) {
          val (description, color) = when {
            degree < 5 -> "cold" to Color.BLUE
            degree <23 -> "mild" to Color.YELLOW
            else -> "hot" to Color.RED 
          }     
      }
      ```
    + 캡쳐링
      - sequence 지연으로 인해 변수를 캡쳐해서 발생됨
      ```kotlin
      // 에라토스네체를 활용한 소수 구하기로 배수의 원리를 이용 prime_num 의 배수를 리스트에서 제거
      val primes: Sequence<Int> = sequence {
        var numbers = generateSequence(2) { it + 1 }

        while (true) {
            val prime = numbers.first()
            yield(prime)

            numbers = numbers.drop(1).filter { it % prime != 0 }
        }
      
        /** 에러발생 (2 3 5 6 7 8 9 10 11 12)
        var prime1: Int
        while (true) {
            prime1 = numbers.first()
            yield(prime1)

            numbers = numbers.drop(1).filter { it % prime1 != 0 }
        }
        */
      }
    
      print(primes.take(10).toList())
      ```
<br></br>
- 최대한 플랫폼 타입을 사용하지 마라
    + nullable 여부를 알 수 없는 타입 -> 어노테이션 활용(@NotNull, @Nullable)
    + NPE를 발생할 수 있는 요소 중 하나
<br></br>
- inferred 타입으로 리턴하지 마라
    + 추론타입은 피연산자에 맞게 설정 됨
    + 인터페이스에 fun produce(): Car -> fun produce() = default_car 로 바꾸면
    Car의 다형성을 이용하여 여러 자동차를 생성하는 것에서 default_car 타입으로 제한
<br></br>
- 예외를 활용해 코드에 제한을 걸어라
    + require 를 통해 argument 제한
    + check 를 통해 상태와 관련된 동작 제한
    + Elvis 연산자 활용
    ```kotlin
      val email = Person.email ?: return
      val age = Person.age ?: run { 
        log("age null") 
        return
      }
    ```
<br></br>
- 사용자 정의 오류보다는 표준 오류를 사용
    + 재정의 한 오류보다는 표준 오류가 많이 알려져있기에 개발자가 이해하기 쉬움
<br></br>
- 결과 부족이 생길떄 null & Failure 사용
    + try-catch 보다 명확하고 오류를 놓칠확률이 줄어듬
    + get() 보다는 getOrNull() 을 통해 리턴을 예측하여 처리할 수 있게
    ```kotlin
    sealed class Result<out T>
    class Success<out T>(val result: T): Result<T>()
    class Failure(val throwable: Throwable): Result<Nothing>()
  
    val person = userText.readObjectOrNull<Person>()
    val age = when (person) {
        is Success -> person.age
        is Failure -> -1  
    }
    ```
<br></br>
- 적절하게 null 을 사용하라
    + !!의 사용을 줄여라 -> npe 발생할 수 있음
    + lateinit(초기 호출이 명확할 때) 또는 Delegated.notNull 사
    ```kotlin

    private var id: Int by Delegates.notNull<Int>()
    ```
<br></br>
- use를 사용하여 리소스를 닫아라
    + 반드시 닫아야 하는 리소스들 사용할 때 try-catch 보다는 use -> try-catch 
      close() 추후 가비지 컬렉터가 다 수거해야 가므로 리소스 낭비
    ```kotlin
    BufferReader(FileReader(path)).use { reader -> 
        return reader.lineSequence().subMy { it.lengh }
    }
    ```
<br></br>
- 단위 테스트를 만들어라
    + 비즈니스 로직, 복잡, 수정, 문제가 빈번한 곳

### 1-2. 가독성
- 가독성을 목표로 삼아라
    + if 를 활용한 null check 인간이 인지하는 속도를 느리게 함
    ```kotlin
      if (person != null && person.isAdult) {
        // code
      } else {
        // error
      }
      //person 이 null 일때 에러인지, 성인이 아닐때 에러인지에 대한 이해가 더 필요
    
      person?.takeIf { it.isAdult }
      ?.let {
          //code
      }
      ?.run {
          //error 엘비스연산자 활
      }
    ```
<br></br>
- 연산자 오버로드를 할 때는 의미에 맞게 사용
    + 함수에 맞는 이름을 사용 계(예외적으로 도메인 특화 언어는 가능)

<br></br>
- Unit? 을 리턴하지 말라
    + if (!isSuccess(key)) return 와 success(key) ?: return 을 
    오해를 하기 쉽고 눈에 잘 들어오지 않는다.

<br></br>
- 변수 타입이 명확하지 않는 경우 확실하게 지정
    + 코틀린은 수준 높은 타입 추론 시스템을 가지고 있음
    + 그러나, val data = getData() 보다는 val data: UserData = getData()
    로 쓰는 것이 프로그래머가 개발자가 코드를 더 쉽게 파악할 수 있음
      
<br></br>
## 2. 코드설계
### 2-1. 재사용성
### 2-2. 추상화 설계
### 2-3. 객체 생성
### 2-4. 클래스 설계

<br></br>
## 3. 효율성
### 3-1. 비용 줄이기
### 3-2. 효율적인 컬렉션 처리