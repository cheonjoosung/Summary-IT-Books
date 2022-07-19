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
    +

### 1-2. 가독성
가독성이란?

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