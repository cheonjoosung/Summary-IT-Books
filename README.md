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
    + v


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