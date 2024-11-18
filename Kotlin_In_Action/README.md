# Kotlin In Action

## 1부 코틀린 소개
### 1장 코틀린이랑 무엇이며, 왜 필요한가?
- 개요
  * 자바 환경에서 동작 & 간결하고 실용적 & 자바 상호운용성
  * 안드로이드 & 서버 등 다양한 분야에서 사용
  <br></br>
    
- 1.1 코틀린 맛보기용
  ```kotlin
  data class Person(val name: String,  age: Int? = null)
  
  val persons = listOf(Person("영희"), Person("철수", 14))
  val oldest = person.maxBy { it.age ?: 0 }
  ```
  * ? 연산자를 사용하여 받는 인자가 없는 경우 null 설정이 가능
  * 엘비스 연산자 ?: 를 사용하여 나이가 없는 경우 0으로 설정
  <br></br>
    
- 1.2 코틀린 주요 특성
  * 1.2.1 멀티 플랫폼
    + 서버, 안드로이드, iOS, 데스크탑 애플리케이션 등
  * 1.2.2 정적 타입 지정 언어
    + 컴파일 시점에 구성 요소의 타입을 알 수 있고 필드(field) or 메소드(method)드 사용할 때마다 컴파일러가 타입 검증
    + 컴파일러가 문맥을 고려해 변수타입을 결정하는 '타입 추론' 가능
    + 장점 (성능, 신뢰성, 유지 보수성, 도구지원)
  * 1.2.3 함수형 프로그래밍 & 객체지향 프로그래밍
    + 일급 시민인 함수 : 함수를 변수에 저장 또는 인자로 전달
    + 불변성 : 내부 상태가 바뀌지 않는 불변 객체를 사용
    + 부수 효과 없음 : 입력이 같으면 출력이 같음 (순수 함수 - 테스트편이성)
      - ```kotlin
        fun findAlice() = findPerson { it.name = "Alice" }
        fun findBob() = findPerson { it.name = "Bob" }
        ```
      - 익명 함수를 사용하면 명령형 코드에 비에 간결하고 추상화 가능
  <br></br>
        
  * 1.2.4 무료 오픈소스
  <br></br>
    
- 1.3 코틀린 응용
  * 1.3.1 서버 프로그래밍
    ```kotlin
    fun renderPersonList(persons: Collection<Person>) =
       createHTML().table{
           for (person in persons) {
               tr {
                   td { +person.name }
                   td { +person.age }
               }
           }
       }
    ```
    + HTML태그와 코틀린언어의 조합 가능
    + 데이터베이스에 질의를 날릴 수 있는 코드도 작성이 가능
    <br></br>
      
  * 1.3.2 코틀린 안드로이드 프로그래밍
    + Compose 구성
      ```kotlin
      verticalLayout {
          val name = editText()
          button("Say Hello") {
              onClick { toast("Hello, ${name.text}!")}
          }
      }
      ```
    + NPE 크게 줄어듬
    + 코틀린 표준 라이브러리 함수는 인자로 받은 람다 함수를 인라이닝 한다.
      람다를 사용해도 새로운 객체가 만들어지지 않으므로 가비지 컬렉션이 늘어나지 않음
    <br></br>
      
- 1.4 코틀린 철학
  * 1.4.1 실용성
    + 실제 문제를 해결하기 위해 만들어진 실용적인 언어
    + 새로운 것을 제시하기 보다는 검증된 언어의 해법과 기법을 적용
  <br></br>
      
  * 1.4.2 간결성
    + data class 묵시적으로 제공하는 함수들이 많음 (get, set, toString, hashCode 등)
    + 반복적으로 작성해야 하는 코드를 제거
  <br></br>
      
  * 1.4.3 안정성
    + 예외처리를 위해 더 많은 정보를 붙여야 하는 작업을 줄임 (trade off)
    + NPE 가능성을 크게 줄여줌 (? 연산자)
    + 타입 체크 (is String) 만 하면 별도 연산자 없이 해당 자료형의 메소드 사용 가능
  <br></br>
      
  * 1.4.4 상호운용성
    + 자바 콜렉션을 사용하고 이를 추가하여 확장하는 방식을 사용
  <br></br>
    
- 1.5 코틀린 도구 사용
  * 1.5.1 코틀린 코드 컴파일
    + .kt -> 컴파일러 -> .class -> jar -> 코틀린 런타임 -> 애플리케이션
    <br></br>
      
  * 1.5.2 인텔리J & 안드로이드 스튜디오
    <br></br>
    
  * 1.5.3 대화형 셀
    <br></br>
    
  * 1.5.4 이클립스 플러그인
    <br></br>
    
  * 1.5.5 온라인 놀이터 - 플레이그라운드
    <br></br>
    
  * 1.5.6 자바-코틀린 변환기
    <br></br>
    
<br></br>
### 2장 코틀린 기초
- 2.1 기본 요소: 함수와 변수
  * 2.1.1. Hello World
    + 함수 fun 키워드 사용
    + 변수명: 변수타입을 사용
  <br></br>
      
  * 2.1.2 함수 
    + 식이 본문인 함수
    ```kotlin
    fun add(a: Int, b: Int) = a + b
    fun max(a: Int, b: Int) = if (a > b) a else b
    ```
  <br></br>
  
  * 2.1.3 변수
    + val 변경 불가능한 참조를 저장하는 변수(참조가 가르키는 객체 내부 변경 가능)
    + var 변경 가능한 참조
  <br></br>
      
  * 2.1.4 문자열 템플릿
    ```kotlin
    val hello = "Hello $name"
    println("\$x") //$x 를 출력하고 싶을 때 escape \ 사용
    ```
  <br></br>
  
- 2.2 클래스와 프로퍼티
  * 기본 가시성은 public 이므로 생략 가능
  * 2.2.1 프로퍼티
    + 필드&접근자를 프로퍼티라 부름
    + val 읽기 전용 프로퍼티로 get 제공 
    + var 쓸수 있는 프로퍼티로 get, set 제공
  <br></br>
      
  * 2.2.2 커스텀 접근자
    ```kotlin
    val isGood: Boolean
      get() {
        return message == "Good"
      }
    ```
    <br></br>
    
  * 2.2.3 코틀린 소스코드 구조: 디렉터리와 패키지
    + 패키지.method 로 접근 가능
  <br></br>
      
- 2.3 선택 표현과 처리: enum & when
  * 2.3.1 enum
    + soft Keyword 
    ```kotlin
    enum class Color { RED, BLUE }
    enum class Color2 (val r:Int, val g: Int, val b: Int) {
        RED(255, 0, 0), BLUE(255, 165, 0); //상수 선언시 , 마지막 ;
    
        fun rgb() = (r*256 +g) * 256 +b
    }
    ```
    <br></br>
    
  * 2.3.2 when 으로 enum 사용
    + if 대신 많이 쓰임
    + java switch 와 달리 break 안씀
    + , 연산자를 이용해 여러 값 사용도 가능
    ```kotlin    
    fun getColor(color: Color) = 
       when (color) {  
         Color.RED -> "RED"
         Color.BLUE -> "BLUE"
         Color.GREEN, COLOR.YELLOW -> "NONE"
       }
    ```
    <br></br>
    
  * 2.3.3 when & 임의의 객체를 함께 사용
    + Java Switch 분기 조건(enum 상수 숫자 리터럴)에 비해 객체 사용이 가능
  <br></br>
      
  * 2.3.4 인자없는 when 가능
    + 인자 없는 경우만 각 항목은 불리언 결과를 계산하는 식이어야 함
  <br></br>
      
  * 2.3.5 스마트 캐스트 : 타입 검사와 타입 캐스트를 조합
    + is 사용 (java instanceof)
    + is 사용 시 컴파일러가 자동으로 타입 캐스팅 수행함 -> 스마트 캐스트라 부름
  <br></br>
      
  * 2.3.6 리팩토링: if -> when 으로 변경
    + when 은 해당 구문만 실행하지만 if-else는 전체를 실행할 수 잇음
  <br></br>
    
  * 2.3.7 if - when 분기에서 블록 사용
  <br></br>
    
- 2.4 대상을 이터레이션: while & for 루프
  * 2.4.1 while 루프
    + java 와 동일
  <br></br>
      
  * 2.4.2 수에 대한 이터레이션 범위와 수열
    + 범위 연산자 1..10
    + for (i in 1..100) print(i)
    + for (i in 100 downTo 1 step 2) print(i)
  <br></br>
      
  * 2.4.3 맵에 대한 이터레이션
    + for ((key, value) in map)
    + for ((index, element) in list.withIndex())
  <br></br>
  
  * 2.4.4 in으로 컬렉션이나 범위의 원소 검사
    + a in 1..4 a가 1에서 4사이에 있는지 
    ```kotlin
    fun isAlphabet(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
    ```
  <br></br>
  
- 2.5 코틀린의 예외처리
  * 2.5.1 try, catch, finally
    + throws 절이 없음
  <br></br>
      
  * 2.5.2 try 를 식으로 사용
  <br></br>

### 3장 함수 정의와 호출
- 3.1 코틀린에서 컬렉션 만들기
  * 코틀린이 자체 컬렉션을 제공하지 않는 이유는
    + 자바 컬렉션 활용하면 상호작용이 쉬움
    + 코틀린 컬렉션 = 자바 컬렉션 + 추가기능 (ex. last(), max() 등)
  <br></br>
      
- 3.2 함수를 호출하기 쉽게 만들기
  * 3.2.1 이름 붙인 인자
    ```kotlin
    println(joinToString(list, separtor = ";", prefix = "(", postfix = ")"))
    ```
    + 변수가 많을 때 호출하는 곳에서 어떤 값을 전달할지 변수를 미리 파악하기가 쉬워짐
    + 제너릭 하기에 어떤 타입이든 사용이 가능
  <br></br>
      
  * 3.2.2 디폴트 파라미터 값
    + 자바 일부 클래스에서 오버로딩 메서드가 너무 많음
    + 함수 인자 받을 때 없는 경우 = "" or ?= null 식 등으로 값을 세팅할 수 있음
  <br></br>
      
  * 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티
    + 상수를 사용하고 싶을 때 const 사용 (= public static final) 
  <br></br>
  
- 3.3 메서드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
  * 확장함수
    + 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스 밖에 선언된 함수
    + 기존 API가 수정될 때 재작성없이 이 기능을 활용하여 확장 기능 제공이 가능
    ```kotlin
    fun String.lastChar(): Char = get(length - 1) 
    ```
    + 클래스 이름을 '수신 객체 타입' 이라 부르고 호출되는 대상이 되는 값(객쳬 인스턴스) '수신 객체'
    + 수신 객체 멤버에 this 없이 접근 가능
    + private or protected 멤버는 접근 불가
  <br></br>
      
  * 3.3.1 임포트와 확장 함수
    + 선언된 패키지의 함수를 선언해야 사용 가능
    + as 를 사용하여 별칭으로 사용 가능
    + 기존 함수와의 충돌을 회피해야하기에 짧은 이름 또는 바꿔서 사용해야 함
    <br></br>
      
  * 3.3.2 자바에서 확장 함수 호출
    + 수신 객체를 첫 번쨰로 인자로 받는 정적 메서드
    + 확장 함수를 호출해도 다른 어댑터 객체나 실행 시점 부가 비용이 들지 않는다
    <br></br>
      
  * 3.3.3 확장 함수로 유틸리티 함수 정의
    <br></br>
    
  * 3.3.4 확장 함수는 오버라이드할 수 없음
    + 확장 함수는 오버라이드 할 수 없다.
    ```kotlin
    open class View {
        open fun click() = println("View Clicked")
    }
    class Button : View {
        override fun click() = println("Button Clicked")
    }
    val view: View = Button()
    view.click() //-> Button Clicked 호출 
    ```
    + 안드로이드의 경우 view 가르키는 객체인 Button 의 clicked 를 호출함
    ```kotlin
    fun View.showOff() = println("I am View")
    fun Button.showOff() = println("I am Button")
    val view: View = Button()
    view.showOff() // -> I am View
    ```
    + 확장 함수는 정적 메서드기에 컴파일 시 View 타입을 전달하기에 View의 showOff 가 호출 됨
    + 확장 함수의 이름과 멤버 변수의 이름이 같을 경우 멤버 변수의 우선 순위가 높아서 호출이 됨
    <br></br>
      
  * 3.3.5 확장 프로퍼티
    ```kotlin
    val String.lastChar: Char
      get() = get(length - 1)
      set(value: Char) {
         this.setCharAt(lenght - 1, value)
      }
    ```
    + property 에 확장
  <br></br>
      
- 3.4 컬렉션 처리: 가변 길이 인자, 중위함수 호출, 라이브러리 지원
  * vararg 키워드 사용 시 인자 개수가 달라지는 함수 정의 가능
  * 중위(infix) 함수 호출 구문을 사용하면 인자가 하나뿐인 메서드를 간편하게 호출 가능
  * 구조 분해 선언(destructuring declaration) 사용하면 복합적인 값을 분해해서 여러 변수에 나눠 담을 수 있음
  <br></br>
    
  * 3.4.1 자바 컬렉션 API 확장
    + last() or maxOrNull() 도 확장함수
    ```kotlin
    public fun <T> List<T>.last(): T {
      if (isEmpty()) throw NoSuchElementException("List is empty.")
      return this[lastIndex]
    }
    
    public fun <T : Comparable<T>> Iterable<T>.maxOrNull(): T? {
      val iterator = iterator()
      if (!iterator.hasNext()) return null
      var max = iterator.next()
      while (iterator.hasNext()) {
          val e = iterator.next()
          if (max < e) max = e
      }
      return max
    }
    ```
  <br></br>
  
  * 3.4.2 가변 인자 함수: 인자의 개수가 달라질 있는 함수 정의 - vararg
    + listOf() 의 경우 요소 값이 가변적이므로 vararg 필요
    ```kotlin
    public fun <T> listOf(vararg elements: T): List<T> = if (elements.size > 0) elements.asList() else emptyList()
    ```
    <br></br>
    
  * 3.4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언
    ```kotlin
    val map = mapOf(1 to "one", 2 to "two") to 메서드 사용
    val (number, name) = 1 to "one" // 구조 분해 선언 (Pair)
    ```
    <br></br>
  
- 3.5 문자열과 정규식 다루기
  * 3.5.1 문자열 나누기 - split()
    + java 에서는 . 이 불가능하지만 코틀린은 가능
    <br></br>
    
  * 3.5.2 정규식과 3중 따옴표로 묶은 문자열
    ```kotlin
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    matchResult?.let {
        val (dir, fileName, ext) = it.destructured
    }
    ```
    <br></br>
  
- 3.6 코드 다듬기: 로컬 함수와 확장
<br></br>
  

### 4장 클래스 객체 인터페이스
- 개요
  * 코틀린은 기본적으로 public 이며 final 이다
  * data class 로 선언 시 유용한 메소드 들을 자동으로 생성해줌
  <br></br>
    
- 4.1 클래스 계층 정의
  * 4.1.1 코틀린 인터페이스
    ```kotlin
    interface Clickable {
        fun click()
        fun showOff() = println("I am clickable")
    }
    
    interface Focusable {
        fun showOff() = println("I am focusable")
    }
    
    class Button: Clickable, Focuable {
        override fun click() = println("I was clicked!")
    
        override fun showOff(){ 
            super<CLickable>.showOff()
            super<Focusable>.showOff()
        }
    }
    ```
    + 자바와 달리 코틀린 에서는 키워드로 : 와 override 사용
    + 코틀린 인터페이스에서 함수 구현이 가능하며 동일한 메소드를 override 할 때 super<Type> 을 통해 호출한 메소드를 정할 수 있음
    + 코틀린은 자바 6에 맞게 호환되어있고 코틀린 1.5부터 default 메서드 생성
    <br></br>
      
  * 4.1.2 open, final, abstract 변경자 : 기본 final
    + 코틀린은 기본적으로 final 임
      - 상속에 대해서 기본적으로 막혀있음
      - 취약한 기반 클래스(fragile base class : 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변) 와 같은 문제를 방지하기 위함
    + open 키워드를 통해 상속을 가능하게 할 수 있고 override 키워드 붙은 메소드도 열려있음(final 키워드로 제한 가능)
    + abstract 
      - class 에 붙은 경우 추상 클래스가 되며 인스턴스를 만들 수 없다
      - method 에 붙은 경우 항상 열려있으며 구현이 존재하지 않고 반드시 overrider 해야 함
    + 제어 변경 자
      - final : 오버라이드 할 수 없음
      - open : 오버라이드 할 수 있음
      - abstract : 반드시 오버라이드 해야 함
      - override : 상위 클래스나 상위 인스턴스의 멤버를 오버라이드하는 중
    <br></br>
        
  * 4.1.3 가시성 변경자: 기본적으로 공개(public)
    + 클래스 외부 접근을 제어
    + public, private, protected 는 기본적으로 자바와 같으나 default 가 public
    + internal 같은 모듈안에서만 볼 수 있음
    <br></br>
      
  * 4.1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스
    ```kotlin
    class Button : View {
        override fun getCurrentState(): State = ButtonState()
        class ButtonState: State { /* ... */ }
    }
    ```
    + 클래스 B 안에 정의된 클래스 A
      - 중첩클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음) java static class, kotlin class A 
      - 내부 클래스(바깥쪽 클래스에 대한 참조를 저장함) java class A, kotlin inner class A
    <br></br>
      
  * 4.1.5 봉인된 클래스(sealed class):클래스 계층 정의 시 계층 확장 제한
    + 기본적으로 open 이고 else 식 없이 사용이 가능
    ```kotlin
    interface Expr2
    class Num2(val value: Int) : Expr2
    class Sum2(val left: Expr2, val right: Expr2) : Expr2
    
    fun eval2(e: Expr2): Int =
      when (e) {
        is Num2 -> e.value
        is Sum2 -> eval2(e.right) + eval2(e.left)
        else -> throw IllegalArgumentException("unchecked exception")
      }
    
    
    sealed class Expr {
      class Num(val value: Int) : Expr()
      class Sum(val left: Expr, val right: Expr) : Expr()
    }
    
    fun eval(e: Expr): Int =
      when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
      }
    ```
    + selaed 를 쓰는 경우 else 를 빼먹어도 문제가 발생하지 않음 (when 에서 모든 하위 클래스를 검사하기에..)
    <br></br>
  
- 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언
  * 4.2.1 클래스 초기화: 주 생성자와 초기화 블록
    ```kotlin
    class User(_nickname: String) {
        val nickname = _nickname
    }
    class User2(val nickname: String) //val 은 이 파라미터에 상응하는 프로퍼티가 생성된다는 뜻
    ```
    + init 블록을 통해 값을 세팅 가능
    + 클래스를 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 private 으로 만들면 됨
    <br></br>
      
  * 4.2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화
    ```kotlin
    open class View {
        constructor(ctx: Context) { }
        constructor(ctx: Context, attr:AttributeSet) { }
    }
    ```
    + 안드로이드에서 많이 볼 수 있다. : super() 를 통해 상위 클래스 생성자 호출 가능
    + this() 를 통해 다른 생성자에게 위임 가능
    <br></br>
      
  * 4.2.3 인터페이스에 선언된 프로퍼티 구현
    ```kotlin
    interface User {
        val nickname: String
    }
    
    class SubscribingUser(val email:String) : User {
        override val nickname:String
            get() = email.substringBefore('@')
    }
    ```
    <br></br>
    
  * 4.2.4 게터 & 세터에서 뒷받침하는 필드에 접근
    + field 라는 특별식별자를 사용해서 기존에 사용한 프로퍼티에 접근할 수 있음
    <br></br>
      
  * 4.2.5 접근자의 가시성 변경
    + 접근자의 가시성은 프로퍼티의 가기성과 같지만 get/set 앞에 가시성 변경자를 붙여 변경 가능
    <br></br>
  
- 4.3 컴파일러가 생성한 메서드: data class & class 위임
  * 4.3.1 모든 클래스가 정의해야 하는 메서드
    + toString()
      - 클래스의 있는 정보를 문자열로 반환 (오버라이딩 가능)
    + equals()
      - 객체의 동등성 비교
      - == 동등성 비교(equals 호출), === 참조 비교
    + hashCode()
      - java 에서는 equals 오버라이드 할 때 반드시 hashCode() 도 오버라이드 해야 한다
      - HashSet의 경우 비용 줄이고자 먼저 해쉬 코드를 비교하고 해쉬코드가 같은 경우에만 실제 값을 비교 한다
    <br></br>
        
  * 4.3.2 data class: 메소드 자동 생성
    + class 보다는 data class 를 사용하면 equals(), toString(), hashCode() 반복적 코드 생략 가능
    + copy() - 일부 프로퍼티를 변경하여 새로운 객체를 만들수 있음
      ```kotlin
      val lee = Client(name = "이계인", age = 4)
      val lee2 = lee.copy(age = 5)
      ```
      <br></br>
    
  * 4.3.3 클래스 위임: by 키워드
    + 대규모 객체지향 설계를 하면서 하위클래스를 상위클래스에 종속하게 만들고 상위 클래스가 변경되면서 코드가 깨지는 경우가 많다
    + 데코레이터 패턴 : 상속을 허용하지 않는 클래스 대신 사용할 수 있는 새로운 클래스를 만들되 기존 클래스와 같은 인터페이스를
  데코레이터가 제공하게 만들고
    - : MutableCollection<T> by innerSet 뮤터블콜렉션의 구현을 innerSet에게 위임, 뮤터블 콜렉션에서
  사용하는 메소드를 직접구현
      <br></br>

- 4.4 object 키워드: 클래스 선언과 인스터스 생성
  * 4.4.1 객체 선언: 싱글턴 만들기
    + object 키워드를 사용
    + 생성과 동시에 인스턴스가 생성되기에 생성자 정의가 필요 없음
    <br></br>
      
  * 4.4.2 동반객체
    + 코틀린에는 static 키워드가 없기에 패키지 수준의 최상위 함수 & 객체 선언 시 사용
    + companion object { } 키워드 사용
    + 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있다
    + 어떤 클래스의 연관있는 메서드와 팩토리 메서드를 담을 때 쓰임
    + 동반객체를 일반 객체 메서드에 접근할 때 동반 객체가 포함된 클래스으 ㅣ이름 사용
    <br></br>

  * 4.4.3 동반 객체를 일반 객체처럼 사용
    + 동반 객체에서 인터페이스 구현
    + 동반 객체 확장
    <br></br>
    
  * 4.4.4 객체 식: 무명 내부 클래스를 다른 방식으로 확장
    + 추상 클래스 또는 인터페이스를 구현한 클래스없이 확장하여 사용 가능 (android clickListener)
    + SAM : Single Abstract method 추상 메소드가 단 하나만 있ㄲ는 인터페이스
    <br></br>


### 5장 람다로 프로그래밍
- 5.1 람다 식과 멤버 참조
  * 5.1.1 람다 소개: 코드 블록을 함수 인자로 넘기기
    + 람다식으로 간편하고 이해하기 쉽게 작성 가능
    ```kotlin
    btn.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View view) {
            
        }
    })
    btn.setOnClickListener { /* 동작수행 */    }
    ```
    <br></br>
    
  * 5.1.2 람다와 컬렉션
    + list.maxByOrNull { it.age }
    <br></br>
      
  * 5.1.3 람다 식의 문법
    ```
    val sum = { x: Int, y: Int -> x + y }
    println(sum(1,2))
    
    list.maxByOrNull { it.age } // 안쓰는 경우 it 자동 생성
    list.maxByOrNull { p: Person -> p.age } // 명확하게 구분 또는 중첩일 때 선언 필요
    list.maxByOrNull { p -> p.age } //type 추론 생략
    ```
    <br></br>
      
  * 5.1.4 현재 영역에 있는 변수에 접근
    + 코틀린은 자바와 달리 람다에서 람다 밖 함수에 있는 파이널이 아닌 변수에 접근 가능하며 변경도 가능
    + 이러한 외부 변수들을 람다가 포획한 변수라 부르고 생명주기가 끝나면 반환
    <br></br>
    
  * 5.1.5 멤버 참조
    + 클래스::멤버 멤버 참조
    + 최상위에 선언된 함수나 프로퍼티를 참조할 수도 있음
    <br></br>
      
- 5.2 컬렉션 함수형 API
  * 5.2.1 필수적인 함수: filter, map
    ```kotlin
    val list = listOf(1,2,3,4)
    list.filter { it % 2 == 0 }
    list.map { it * it }
    list.filter {it % 2 == 0}.map { it * it}
    ```
    + filter 
      - Iteration 하면서 주어진 람다가 true 만 반환
    + map
      - 주어진 람다를 각 원소에 적용한 결과를 모아 새 컬렉션을 만듬
    <br></br>
    
  * 5.2.2 all, any, count, find: 컬렉션에 술어 적용
    ```kotlin
    val canBeInClub27 = { p: Persion -> p.age <=27 }
    val people = listOf(Person("A", 27), Person("B", 31))
    people.all(canBeInClub27)
    people.any(canBeInClub27)
    people.count(canBeInClub27)
    ```
    + all : 모두 만족 하는가?
    + any : 하나라도 만족 하는가?
    + count() : 몇 개 인가? 
      - filter 와 같이 쓸 때 count 가 size 보다 효율적임 (size 는 중간 컬렉션을 생성함)
    <br></br>
      
  * 5.2.3 groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경
    ```kotlin
    val people = listOf(Person("A", 27), Person("B", 31), Person("C", 31))
    val ageGroup = people.groupBy { it.age }
    ```
    + key(주어진 람다 위 식에서 age) 를 기반으로 그룹핑한 결과를 반환함
    <br></br>
    
  * 5.2.4 flatMap & flatten: 중첩된 컬렉션 안의 원소 처리
    ```kotlin
    val people = listOf(Person("A", 27), Person("B", 31), Person("C", 31))
    people.flatMap { it.name }.toSet() // people 컬렉션에 있는 이름의 집합
    ```
    + "abc", "def" -> "abcdef" 를 flatten
    <br></br>
    
- 5.3 지연 계산(lazy) 컬렉션 연산
  * map or filter 컬렉션을 즉시 생성함 -> 도중 계산 결과를 새로운 컬렉션에 임시로 담음
  ```kotlin
  people.map(Person::name).filter { it.startsWith("A") }
  people.asSequenc()
    .map(Person::name)
    .filter { it.startsWith("A") }
    .asList()
  ```
  * 시퀀스 연산 시 중간 임시 컬렉션이 생기지 않고 원소수가 많을 때 성능이 크게 좋아짐
  * 5.3.1 시퀀스 연산 실행: 중간 연산과 최종 연산
    + map &filter 변환이 늦춰져서 결과를 얻을 필요가 있을 때 적용 됨
    ```kotlin
    listOf(1,2,3,4).asSequence()
      .map { print("map($it) ") ; it*it }
      .filter { print("filter($it) ") ; it%2 == 0 }
      .toList()
    ```
    + filter 를 통해 불필요한 요소 제거 후 map 으로 변환 작업 수행
  * 5.3.2 시퀀스 만들기
  <br></br>
  
- 5.4 자바 함수형 인터페이스 활용
  ```java
  button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
    }
  });
  ```
  ```kotlin
  button.setOnClickListener { view -> }
  ```
  * 이런 것이 가능 한 이유는 OnClickListener 에 추상 메서드가 단 하나기 때문이다
    + 이 메소드를 함수형 인터페이스 또는 SAM(Single Abstract Method) 인터페이스라고 함
    + 이 안에 메소드를 SAM method 라 함
    + Runnable, Callable 도 있음
  <br></br>
  
  * 5.4.1 자바 메서드에 람다를 인자로 전달
    ```kotlin
    void postponeComputation(int delay, Runnable computation)
    postponeComputation(1000) { println(42) }
    ```
    + 컴파일러가 자동으로 람다를 Runnable 인스턴스로 변환 (무명 클래스의 인스턴스)
    <br></br>
    
  * 5.4.2 SAM생성자 : 람다를 함수형 인터페이스로 명시적으로 변경
    ```kotlin
    val listener = OnClickListener { view ->
    }
    button1.setOnClickListener(listener)
    ```
    <br></br>
    
- 5.5 수신 객체 지정 람다: with 와 apply
  * with 함수
    + 메서드를 호출하려는 수신 객체를 전달하고 내부적으로 this 를 사용해도 되고 생략도 가능
    + 일반 함수는 일반 람다, 확장 함수는 수신 객체 지정 람다
  * apply 함수
    + 객체를 초기화 할 때 주로 사용
    + 인스턴스를 만들고 apply에 이 값을 전달함
    + 블럭 안에서 해당 수신 객체를 프로퍼티를 설정할 수 있음
    <br></br>
    
### 6장 코틀린 타입 시스템
- 6.1 널 가능성
  * NPE(NullPointerException 오류)를 피할 수 있도록 도와주는 코틀린 타입 시스템 특징
  * 6.1.1 Null 이 될 수 있는 타입
    + Type? = Type or Null 로써 해당 인스턴스가 널이 될 수 있다는 것을 알려줘야 함
    
  * 6.1.2 타입의 의미
    + java 의 경우 annotaion 으로 제공하나 이는 표준이 아니라 항상 일정한 결과를 보장하지 않음
    + kotlin 의 null 타입은 여러 해법을 제공하고 컴파일 시점에 확실히 알 수 있기에 실행시간에 오류를 피할 수 있음
    
  * 6.1.3 안전한 호출 연산자: ?.
    + 해당 값이 null 이 아닐 때 뒤의 연산을 수행한다
    ```kotlin
    if (s != null) s.toUpperCase() else null //if-else 반복적 의미없는 코드 작성 필요함
    s?.toUpperCase() //훨씬 간단함
    ```
    + 안전한 호출을 가능하게 함
    
  * 6.1.4 엘비스 연산자: ?:
    + 이항 연산자로 좌항을 계산한 값이 널인지 체크하고 널인 경우 우항 값을 결과라 할당함
    ```kotlin
    fun getName(user: User) user?.getName() ?: "no name"
    ```
    + ?: 연산자를 통해 default 값 설정도 가능하고, return & throw 사용도 가능
    
  * 6.1.5 안전한 캐스트: as?
    + as? 를 통해 타입을 원하는 타입인지 쉽게 검사하고 casting 까지 가능
    ```kotlin
    foo as? type ?: return false
    ```
  
  * 6.1.6 널 아님 단언: !!
    + 널이 될 수 없다고 단정하고 강제로 NotNull 형태로 타입 강제
    + 잘못 생각해도 예외를 감수하겠다는 의미로 위험한 코드다
    + !! 포함한 식을 한번에 쓰면 NPE 발생 정보를 안알려주기에 여러줄로 나누어서 구현
    
  * 6.1.7 let 함수
    + Object?.let { } null이 아닐 경우 블록 안의 코드를 실행
    ```kotlin
    s?.let { print(s.length) }
    ```
    
  * 6.1.8 나중에 초기화할 프로퍼티
    + lateinit null 로 할당하는 대신에 사용 가능하고 var 사용
    ```kotlin
    lateinit var user: User
    if (::user.isInitialized) { /* 초기화여부 체크 */ }  
    ```
    
  * 6.1.9 널이 될 수 있는 타입 확장
    + String 의 isNullOrBlank(), isNullOrEmpty() 널을 명시적으로 검사함
    
  * 6.1.10 타입 파라미터의 널 가능성
    + t: T 인 경우 널이 될 수 없는 타입 상한을 지정해야 함
    
  * 6.1.11 널 가능성과 자바
    + 플랫폼 타입
      - 코틀린이 널 관련 정보를 알 수 없는 타입
      - 자바에서는 널을 체크를 따로 하지 않기에 메소드에 어노테이션을 통해 알려줘야 함
    + 상속
      - 자바 클래스 또는 인터페이스를 코틀린에서 구현할 경우널 가능성을 제대로 처리하는 것이 중요
  <br></br>
      
- 6.2 코틀린의 원시 타입
  * 6.2.1 원시 타입: Int, Boolean 등
    + java 에서는 int/Integer 를 때에 따라 나눠 사용하지만 코틀린에서는 Int 하나만 사용
    + 비효율적일것 같지만 실행실점에 가장 효율적인 타입으로 맞춰서 실행 됨
    + 정수 타입 (Byte, Short, Int, Long)
    + 부동소수점 수 타입 (Float, Double)
    + 문자 타입 Char
    + 불리언 Boolean
    
  * 6.2.2 널이 될 수 있는 원시 타입: Int?, Boolean? 등 
    + 원시타입에 null 붙여서 사용하는 경우 컴파일 시 java 의 Wrapper Type 으로 컴파일 됨
    
  * 6.2.3 숫자 변환
    + 다른 타입의 숫자로 자동 변환하지 않고 toLong, toInt 등 직접 호출 필요
    + 1.1 이상부터 숫자 사이에 _ 넣을 수 있음
    ```kotlin
    val i = 1
    val list = listOf(1L, 2L, 3L)
    i in list // false 이고 묵시적 변환을 허용 안함
    i.toLong() in list // true 이고 명시적 변환이 필요 
    ```
    + Long-l, Float-f or F, 16진수-0x or 0X, 2진수-0b, 0B
    
  * 6.2.4 Any, Any?: 최상위 타입
    + Any 는 코틀린에서 최상위 타입, Object 가 java 최상위 클래스 인 것 처럼
    + Any 선언 시 Object 로 컴파일 됨
    
  * 6.2.5 Unit 타입: 코틀린의 void
    + 반환 타입을 명시하지 않거나 : Unit 으로 쓴 경우
    + void 와 달리 타입 인자로 사용 가능
    
  * 6.2.6 Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다
    + 정상적으로 끝나지 않는다는 표현 제공
    + 함수의 반환 타입이나 타입 파라미터로 쓰임
  <br></br>
  
- 6.3 컬렉션과 배열
  * 6.3.1 널 가능성과 컬렉션
    + List<Int?> List<Int>? 는 전혀 다름. 원소가 null이 될수있는 것과 전체 리스트가 null 이 될수 있는 것의 차이
    + filterNotNull 를 통해 List 에서 Null 이 아닌것을 필터할 수 있음
    ```kotlin
    fun check(list : List<Int?>) {
      var invalidCount = 0
      var validCount = 0
      for (item in list) {
        if (item == null) invalidCount++
        else validCount++
      }
    }
    ```
    
  * 6.3.2 읽기 전용과 변경 가능한 컬렉션
    + Collection 을 이용하여 만든 것이 MutableCollection 이고 add,remove, clear 등의 원소 추가,삭제,클리어 등 기능
    + 읽기 전용 컬렉션이 항상 스레드 안전하지 않다는 점 명심(한 객체를 여러 컬렉션이 바라보고 있는 경우)
    
  * 6.3.3 코틀린 컬렉션과 자바
    + Iterable 를 구현한 MutableIterable, Collection 를 구현한 MutableCollection 자바 컬렉션 인터페이스의 구조를 그대로 옮겨 놓음
    
  * 6.3.4 컬렉션을 플랫폼 타입으로 다루기
    + 컬렉션이 널이 될 수 있는가? 
    + 원소가 널이 될 수 있는가? 
    + 오버라이드하는 메서드가 컬렉션을 변경할 수 있는가 등  의 사항을 반영해서 코드를 작성해야 함
    
  * 6.3.5 객체의 배열과 원시 타입의 배열
    + arrayOf 함수에 원소를 넘기면 배열 가능
    + arrayOfNulls 함수에 정구밧을 인자로 넘기면 모든 원소가 null 이고 같은 크기의 배열
    ```kotlin
    val alphabet = Array<String>(26){ i -> ('a' + i).toString() }
    ```
  <br></br>


### 7장 연산자 오버로딩과 기타 관례
- 7.1 산술 연산자 오버로딩
  * 7.1.1 이항 산술 연산 오버로딩
    + operator 키워드를 붙여서 관례 표시
    ```kotlin
    data class Point(val x: Int, val y: Int) {
        operator fun plus(other: Point): Point {
            return Point(x + other.x, y + other.y)
        }
    }
    
    operator fun Point.plus(other: Point) : Point {
        reutnr Point(x + other.x, y + other.y)
    }
    ```
    + plus, minus, mod(1.1 부터 rem), div, times 가 함수이름
    + 코틀린 연산자가 자동으로 교환법칙을 지원하지 않음 (ex: p*1.5, 1.5*p 는 다른 오버로딩 구현이 필요)
    + shl(<< 왼쪽시프트), shr(>> 오른쪽시프트), ushr(오른쪽 시프트 >>>), and(& 비트 곱), or(| 비트합), xor(^ 비트배타합), inv(~비트반전)
    
  * 7.1.2 복합 대입 연산자 오버로딩
    + +=, -=, *=, /= 와 같은 연산자 (ex: point += Point(3,4) -> point = point + Point(3,4)
    + plus(a = a.plus(b)), plusAssign(a.plusAssing(b)) 을 동시에 정의 no
  
  * 7.1.3 단항 연산자 오버로딩
    + unaryPlus(+a), unaryMinus(-a), !(!a), (++a, a++)(inc), (--a, a--)(dec)
    ```kotlin
    operator fun Point.unaryMinus(): Point {
        return Point(-x, -y)
    }
    ```
    <br></br>

- 7.2 비교 연산자 오버로딩
  * 7.2.1 동등성 연산자: equals
    + 자바에서는 객체 비교를 위해 equals or compareTo 를 사용하지만 코틀린은 필요없음
    + a == b => a?.equals(b) ?: (b == null)
    ```kotlin
    class Point(val x: Int, val y: Int) {
        override funb equals(obj: Any?) : Boolean {
            if (obj === this) return true // 같은 객체 가리키는 지 확인
            if (obj !is Point) return false // 같은 타입인지 체크
            return obj.x == x && obj.y == y // 값 비교
        }  
    }
    ```
    
  * 7.2.2 순서연산자: compareTo
    + 원시타입의 값만 비교 가능
    + a >= b 연산은 a.compareTo(b) >= 0 으로 변경되어 계산
    ```kotlin
    class Person(val firstName: String, val secondName: String) : Comparable<Person> {
        override fun compareTo(other: Person): Int {
            return compareValuesBy(this, other, Person::lastName, Person::firstName)
        }
    }
    ```
    <br></br>
  
- 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례
  * 7.3.1 인덱스로 원소에 접근: get & set
    + 인덱스 연산자 [] 를 활용 함
    + x[a] -> x.get(a) 로 치환
    + x[a, b] -> x.get(a, b) 로 치환
    + x[a, b] = c -> x.get(a, b, c) 로 치환
    ```kotlin
    operator fun Point.get(index: Int) : Point {
        reutnr when(index) {
            0 -> x 
            1 -> y 
            else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
        }   
    }
    
    operator fun Point.set(index: Int, value: Int) : Point {
        reutnr when(index) {
            0 -> x = value 
            1 -> y = value
            else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
        }   
    }
    ```
    
  * 7.3.2 in 관례
    + in 은 컬렉션에 들어가있는지 검사로 contains() 와 대응한다.
    + a in c -> c.contains(a) 로 치환되며 내부적으로 범위 안을 탐색
    
  * 7.3.3 rangeTo 관례
    + start..end -> start.rangeTo(end) 치환
    
  * 7.3.4 for 루프를 위한 iterator 관례
    + for (x in list) -> list.iterator() 를 호출
    + 내부적으로 hasNext(), next() 호출을 반복하는 식으로 변환
    <br></br>
    
- 7.4 구조 분해 선언과 component 함수
  * val (a, b) = p 는 a = p.component1, b = p.component2 순차적 할당으로 구조 분해 선언을 의미한다.
  * 컴파일러가 자동으로 componentN 함수를 만들어 줌(5개 까지 값이 할당이 가능 함)
  * 7.4.1 구조 분해 선언과 루프
    + map 의 엔트리에서 component1, component2 를 꺼내 key,value 대입 가능
    ```kotlin
    val map = mapOf("1" to "one", "2" to "two")
    for (entry in map.entries) {
        val key = entry.component1()
        val value = entry.component2()
    }
    ```
    <br></br>
  
- 7.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티
  * 위임 - 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴
  * 뒷받침하는 드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 쉽게 구현
  * 7.5.1 위임 프로퍼티 소개
    ```kotlin
    class Foo {
        var p: Type by Delegate()
    }
    
    class Delegate {
        operator fun getValue() { /*...*/ }
        operator fun setValue() { /*...*/ }
    }
    ```
    + 컴파일러에 의해 생성된 get/set Value 를 통해 p의 값을 설정함
    + 프로퍼티 위임을 사용해 초기화 지연이 가능
    
  * 7.5.2 위임 프로퍼티 사용: by lazy() 사용한 프로퍼티 초기화 지연
    + 객체의 일부분을 초기화하지 않고 **필요한 경우에 초기화하여 사용**하는 패턴
    ```kotlin
    class Email {}
    
    fun loadEmails(person: Person) : List<Email> {
        return listOf()
    }

    class Person(val name: String) {
    private var _emails: List<Email>? = null
    val emails by lazy { loadEmails(this) } // 쉬운 표현
    
        val emails: List<Email>
            get() {
                if (_emails == null) {
                    _emails = loadEmails(this)
                }
                return _emails!!
            }
    }
    
    ```
    + 뒷받침하는 프로퍼티 기법 객체의 인스터스를 만들때는 emails 이 없고 emails 접근할 때 초기화를 진행
  다만 쓰레드 안전성이 떨어지고 프러포티가 많아질 수록 코드가 기하급수적으로 커진다
    + lazy 함수는 기본적으로 쓰레드 안전함. 동기화에 사용할 락을 lazy 함수에 전달할 수도 있고 다중 스레드 환경에서
  사용하지 않을 프로퍼티를 위해 lazy 함수가 동기화를 하지 못하게 막을 수 있음
    
  * 7.5.3 위임 프로퍼티 구현
    + 프로퍼티가 바뀔 때마다 리스너에게 변경 통지할 경우 (UI 변경)
    ```kotlin
    class Person(val name: String, val age: Int, val salary: Int) : PropertyChangeAware() {
        private val observer = {
            prop: KProperty<*>, oldValue: Int, newValue: Int ->
            changeSupport.firePropertyChange(Prop.name, oldValue, newValue)
        }
        
        var age: Int by Delegates.observable(age, observer)
        var salary: Int by Delegates.observable(salary, observer)
    }
    ```
    
  * 7.5.4 위임 프로퍼티 컴파일 규칙
  
  * 7.5.5 프로퍼티 값을 맵에 저장
    + 직렬화? 역질렬화 같은 개별 API 가 존재

  * 7.5.6 프레임워크에서 위임 프로퍼티 활용
<br></br>
  
### 8장 고차 함수: 파라미터와 반환 값으로 람다 사용
- 8.1 고차 함수 정의
  * 다른 함수를 인자로 받거나 반환하는 함수
  * 함수 타입
    - val sum = {x: Int, y: Int -> x+y } 타입추론으로 인해 타입 지정없이 변수에 람다를 대입 가능
    - 함수타입이 Null 인 것과 반환타입이 Null 인 것의 차이를 인지해야 함
  * 인자로 받은 함수 호출
    ```kotlin
    fun add(sum: (Int, Int) -> Int) {
        println("sum is ${sum(2, 3)}")
    }
    ```
    - filter
      + 술어를 파라미터로 받음
  * 자바에서 코틀린 함수 사용
    - 컴파일된 코드의 함수 타입은 일반 인터페이스로 변환 됨
    - FunctionN<P, vargs...> 인자가 N개,
  * 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
    - if (callback != null) 또는 callback? 을 통해 명시적 검사 필요
  * 함수를 함수에서 반환
    - 연락처 정보를 반환하는 함수가 있을 때 이를 받아서 prefix, 내부정보를 변경시켜서 지속적인 활용 가능
  * 람다를 활용한 중복 제거
    - filter, map 을 통해 콜렉션에서 원하는 목록을 뽑을 수 있음
- 8.2 인라인 함수: 람다의 부가 비용 없애기
  * 5장 코틀린이 보통 람다를 무명 클래스로 컴파일하지만 매번 람다 식을 사용할 때마다
  새로운 클래스가 만들어지지 않음
  * 람다가 변수를 포획하면 람다가 생성되는 시점마다 무명 클래스 객체가 생김
  * 실행 시점에 무명 클래스 생성데 따른 비용이 들기에 똑같은 작업을 수행하는 일반 함수를 사용한 구현보다 효율적이지 않음
  * 라이브러리 함수로 빼서 inline 변경자를 붙이면 그 함수를 호출하는 몯느 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해줌
  * 인라이닝이 작동하는 방식
    - 함수를 인라인으로 선언하면 함수의 본문이 인라인(inline) 됨. 함수를 호출하는 코드를
      함수를 호출하는 바이트코드가 아니라 함수 본문을 번역한 바이트 코드로 컴파일
    ```kotlin
    @kotlin.internal.InlineOnly
    public inline fun <R> synchronized(lock: Any, block: () -> R): R {
      contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
      }
    
      @Suppress("NON_PUBLIC_CALL_FROM_PUBLIC_INLINE", "INVISIBLE_MEMBER")
      monitorEnter(lock)
      try {
          return block()
      }
      finally {
          @Suppress("NON_PUBLIC_CALL_FROM_PUBLIC_INLINE", "INVISIBLE_MEMBER")
          monitorExit(lock)
      }
    }

    // 예제
    fun foo(l: Lock) {
      println("Before sync")
      
      synchronized(l) {
        println("Action")
      }
      
      println("After sync")
    }
    
    // 바이트코드
    fun __foo__(l: Lock) {
      println("Before sync")
      l.lock()
    
      try {
        println("Action") 
      } finally {
        l.unlock()
      }
    
      println("After sync")
    }
    ```
    - 함수의 본문뿐만 아니라 synchronized 에 전달된 람다의 본문도 함께 인라이닝 됨
    - 람다의 본문에 의해 만들어지는 바이트코드는 그 람다를 호출하는 코드 정의의 일부분으로 간주되기에 코틀린 컴파일러는
  그 람다를 함수 인터페이스를 구현하는 무명 클래스로 감싸지 않음
    - 인라인 함수를 호출하면서 함수타입의 변수를 넘길 수 있음. 인라인 코드 위치에서
  변수에 저장된 람다의 코드를 알 수 없기에 람다 본문은 인라인 되지 않고 synchronized 함수의 본문만 인라인 됨
  * 인라인 함수의 한계
    - 람다를 사용하는 모든 함수를 인라이닝할 수 없음
  * 컬렉션 연산 인라이닝
    - filter 는 인라인 함수인데 이를 사용하지 않고 직접 구현한다면?
    - filter, map 은 원소가 많을 때 중간연산에 비용이 발생해서 시퀀스를 쓰는게 좋지만,
  시퀀스는 람다를 저장해야 하므로 인라인하지 않기에 지연 계산을 통해 성능 향상을 이유로 모든 콜렉션
      연산에 시퀀스 사용을 하는 것은 좋지 않다.
  * 함수를 인라인으로 선언해야 하는 경우
    - 일반 함수 호출의 경우 JVM 은 이미 강력하게 인라이닝 지원
    - 람다를 인자로 받는 함수를 인라이닝하면 이익이 많음
      + 1. 인라인을 통해 없앨 수 있는 부가 비용이 큼 - 표현 클래스 및 람다 인스턴스 객체 생성
      + 2. JVM 이 함수 호출과 람다를 인라이닝해 줄 정도로 똑똑하지 않음
      + 3. 인라이닝 사용 시 몇가지 부가기능 사용 가능 - ex)넌로컬 반환 사용  
    - 인라이닝은 바이트 코드를 가져오므로 크기가 전체적으로 아주 커질 수 있음
  * 자원 관리를 위해 인라인된 람다 사용
    - try-with-resource : use 함수가 코틀린 표준 라이브러리 안에 있음
    - 파일에 대한 연산을 실행할 람다를 넘김
- 8.3 고차 함수 안에서 흐름 제어
  * 람다 안의 return 문: 람다를 둘러싼 함수로부터 반환
    - 람다 안에서 return 을 사용하면 람다로부터만 반환되는게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환 됨
    - 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 만드는 return 문을 논로컬 return 이라고 함
  * 람다로부터 반환: 레이블을 사용한 리턴
    - @라벨 사용을 통해 지정위치로 리턴이 가능함
    ```kotlin
    fun main() {
        add()
        add2()
    }
    
    fun add() {
      (1..5).forEach label@{
          if (it % 2== 0) return@label
      }

      println("add")
    }

    fun add2() {
      (1..5).forEach {
        if (it % 2== 0) return
      }
    
        println("add2")
    }
    ```
    - this 를 사용시 수신객체로 자기자신임
  * 무명함수: 기본적으로 로컬 return
    - qwe

<br></br>
### 9장 제네릭스
- 9.1 제네릭 타입 파라미터
  * Map<K, V> 일 때 Map<String, Person> 을 넘겨서 타입을 인스턴스화 할 수 있음
  * 제네릭 함수와 프로퍼티
    - <T>를 통해 함수 타입 파라미터, 수신 객체, 반환 타입에 쓰임
    ```kotlin
    fun <T> List<T>.slice(indices: IntRange): List<T>
    ```
    - slice() 를 호출할때 타입을 명시하지 않는 경우 컴파일러가 추론함
  * 제네릭 클래스 선언
    - 자바처럼 클래스/인터페이스에 <>를 통해 제네릭하게 만들 수 있음
  * 타입 파라미터 제약
    - 클래스/함수에 사용할 수 있는 타입 인자를 제한하는 기능
    - sum() 함수를 고려할 때 double, int 는 되지만 String 은 안되도록..
    ```kotlin
    fun <T: Number> List<T>.sum() : T
    listOf(1,2,3).sum()
    listOf(1.0, 2.0, 3.0).sum()
    listOf("1.0")
    ```
    - 제약인자로 Comparable<T>도 가능
    - where T:CharSequence, T: Appendable T에 대한 제약사항으로 두가지의 인터페이스를 구현하도록
  * 타입 파라미터를 널이 될 수 없는 타입으로 한정
    - T: Any 로 지정해서 null 이 발생할 수 없다는 것을 보장
- 9.2 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터
  * JVM 제네릭스는 타입 소거를 사용하여 구현 됨
  * 코틀린에서 함수 inline 을 통해 타입 인자가 안지워지는 "실체화 reify"
  * 실행 시점의 제네릭: 타입 검사와 캐스트
    - List<String> 객체를 만들고 문자열을 넣더라도 실행 시점에 그 객체를 오직 List 로만 볼 수 있음
    - if (list is List<String>) { } 를 사용하면 컴파일 오류가 발생함 (cannot check for instance of erased type)
    - 저장해야 하는 타입 정보가 줄어드므로 메모리 사용량이 줄어들기에 제네릭 타입 소거 나름의 장점이 존재
    - 스타 프로젝션 (*) 을 사용해서 확인이 가능
    - Collection<*> type 으로 받은 후에 List<Int> 타입캐스팅 시 컴파일 에러가 발생하므로 throw exception 으로 unchecked 를 해결하면 됨
  * 실체화한 타입 파라미터를 사용한 함수 선언
    - inline & reified 를 통해 실체화하여 사용 시
    ```kotlin
    fun <T> isA(value: Any) = value is T //compile Error
    inline fun <reified T> isA(value: Any) = value is T
    
    public inline fun <reified R> Iterable<*>.filterIsInstance(): List<@kotlin.internal.NoInfer R> {
        return filterIsInstanceTo(ArrayList<R>())
    }
    ```
    - list 에 문자열, 숫자형이 섞여있을 때 list.filterIsInstance<String>() 을 통해 특정 타입만 가져올 수 있음
  * 실체화한 타입 파라미터로 클래스 참조 대신
    - val serviceImpl = ServiceLoader.load(Service::class.java)
    - 안드로이드 코드의 startActivity() 를 편하게 쓸 수 있음
    ```kotlin
    inline fun <reified T: Activity> Context.startActivity(){
        val intent = Intent(this, T::class.java)
        startActivity(intent)
    }
    
    startActivity<DetailActivity>()
    ```
  * 실체화한 타입 파라미터의 제약
    - 다음과 같은 경우
      * 타입 검새와 캐스팅(is, !is, as, as?)
      * 리플렉션 API(::class),
      * 코틀린 타입에 대응하는 java.lang.Class 얻기
      * 다른 함수를 호출할 때 타입 인자로 사용
    - 불가능한 경우
      * 타입 파라미터의 클래스의 인스턴스 생성
      * 타입 파라미터 클래스의 동반 객체 메서드 호출
      * 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
      * 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 reified 로 지정
- 9.3 변성: 제너릭과 하위 타입
  * 변성이 있는 이유: 인자를 함수에 넘기기
    - Any 타입으로 받는 함수에서 원소의 수정/삭제가 나는 경우 특정 타입으로 받게 되면 실행도중 에러가 발생할 수 있음
    - String 타입을 던졌는데 내부에서 정수형의 원소를 수정/삭제하게 된 경우 컴파일은 문제가 없지만 런타임 에러가 발생
  * 클래스, 타입, 하위 타입
    - Int 는 Number 의 하위 타입이지만 String 은 아님
    - 상위 타입은 하위 타입의 반대 개념
    - Int? 의 하위 타입은 Int 가 맞다
    ```kotlin
    val s: String = "abc"
    val t: String? = s
    
    val s2: String? = "abc"
    val t2: String = s2 // compile Error
    ```
    - Any-String 은 상위-하위 관계가 맞미나 List<Any> - List<String> 이들은 아니다
    - 제네릭 타입을 인스턴스화 할 때 타입 인자로 서로 다른 타입이 들어가고 인스턴스 타입 사이의
  하위 타입 관계가 성립하지 않으면 그 제네릭 타입을 "무공변"(invariant) 라고 함
  * 공변성: 하위 타입 관계를 유지
    - Animal-Cat 관계를 Producer<Animal>-Producer<Cat> 도 유지하기 위해서 out 키워드 사용하여
    공변적이라고 선언
    ```kotlin
    interface Producer<out T> { fun produce(): T }
    ```
    - 타입 캐스팅이 줄어들어서 코드 장황성 및 실수를 줄임
    - Producer<Animal> 로 파라미터를 받는 곳에 Producer<Cat> 을 전달할 수 있음
    - in 을 쓰면 파라미터 안에 쓸수있고 out 을 쓰면 리턴타입 위치
    ```kotlin
    interface MutalbeList<T> : List<T>, MutableCollection<T> {
        override fun add(element: T): boolean // T 위치가 파라미터 안(in 의 위치)에 해당하므로 공변적이지 않음
    }
    ```
    - out 은 읽기 전용으로 사용
  * 반공변성: 뒤집힌 하위 타입 관계
    - 타입을 소비... 쓰기가 가능해짐
    ```kotlin
    interface Comparator<in: T> {
        fun compare(e1: T, e2: T): Int { }
    }
    ```
    - A가 B의 하위타입이면 Consumer<B>가 Consumer<A> 의 하위타입이 됨
    ```kotlin
    interface Function1<in: P, out: R> {
        operator fun invoke(p: P): R // P는 in 의 위치, R은 out 의 위치
    }
    ```
  * 사용 지점 변성: 타입이 언급되는 지정에서 변성 지정
    - in/out 의 위치에 따라 타입 프로젝션이 일어남. MutableList 가 아니라 MutableList 프로젝션을 한 제약 타입
  * 스타 프로젝션: 타입 인자 대신 * 사용
    - 위험할 수 있음 String 으로 받은 후에 Number 연산을 하면 에러가 나기에 타입체크 필수
    - * -> out Any? 처럼 동작함


<br></br>
### 10장 어노테이션과 리플렉션
- 10.1 어노테이션 선언과 적용
  * 어노테이션 적용
    - @ + 이름으로 구성
      * @Test, @Deprecated
    - 클래스를 인자로 지정할 시 @Annotation(MyClass:class) 클래스이름 필요
    - 배열을 인자로 지정하려면 arrayOf 사용하나 annotation 에 ArrayValue 와 같은 경우는
  자동으로 변환 됨
    - const 가 붙은 경우 파일 맨 위 또는 object 안 선언이 필요하고 원시타입/String 만 가능
  * 어노테이션 대상
    - 사용 지점 대상
      + @get: Rule 처럼 사용하면 프로퍼티에 붙임
      + property 전체, field 생성되는 필드, get/set 게터/세터, receiver 확장함수/수신객체,
  param 생성자 파라미터, setparam 세터 파라미터, file 파일안 최상위 함수/프로퍼티스 담는 클래
    - 자바 API를 어노테이션으로 제어하기
      + @JvmName - 코틀린 선언이 만들어내는 자바 필드/메서드 이름을 변경
      + @JvmStatic - 자바 정적 메서드 노출
      + @JvmOverloads - 디폴트 파라미터 값이 있는 함수에 의해 컴파일러가 자동으로 오버로딩한 함수 생성
      + @JvmField - public 자바 필드 형식으로 제공 프로퍼티 세터/게터 없음 
  * 어노테이션으로 활용한 JSON 직렬화 제어
    - 직렬화 : 객체를 저장장치에 저장하거나 네트워크를 통해 전송하기 위해 텍스트나 이진 형식으로 변환하는 것
    - 자바 <-> JSON 을 이용시 Jackson, GSON 라이브러리 사용
    - Person("JS", 31) <-> "name":"JS", "age": 31
  * 어노테이션 선언
  * 메타 어노테이션: 어노테이션 처리하는 방법 제어
    - 어노테이션 클래스에 적용할 수 있는 어노테이션을 지칭함
      + @Target - 지정 가능
      + @Retention - 클래스를 소스 수준에서 유지할지, .class 파일에 저장할지, 실행 시점에 리플렉션을
    사용해 접근할 수 있게 할지 정함
  * 어노테이션 파라미터로 클래스 사용
  * 어노테이션 파라미터로 제네릭 클래스 받기
- 10.2 리플렉션: 실행 시점에 코틀린 객체 내부 관찰
  * 실행 시점에 동적으로 객체의 프로퍼티와 메서드에 접근할 수 있게 해주는 방법
  * java.lang.reflect or kotlin.reflect 가 있는데 코틀린은 null 타입 및 고유 개념이 추가되어 있음
  * 안드로이드의 경우 kotlin-reflect.jar 파일의 의존관계를 추가해야 함
  * 코틀린 리플렉션 API: KClass, KCallable, KFunction, KProperty
    - MyClass:class 라는 식을 쓰면 KClass 의 인스턴스를 얻을 수 있음
    ```kotlin
    val p = Person("abc", 15)
    val kClass = p.javaClass.kotlin
    println(kClass.simpleName)
    ```
    - 함수의 경우 ::functionName 을 통해 확인 가능
    - 인자의 경우 ::인자명
  * 리플렉션을 사용한 객체 직렬화 구현
  * 어노테이션을 활용한 직렬화 제어
  * JSON 파싱과 객체 역직렬화
  * 최종 역직렬화 단계: callBy(), 리플렉션을 사용해 객체 만들기


<br></br>
### 11장 DSL 만들기
- 11.1 API 에서 DSL 로
  * 코틀린 DSL은 간결한 구문을 제공하는 기능과 그 구문을 확자앻서 여러 메서드 호출을 조함으로써
  구조를 만들어 내는 기능에 의존 함
  ```kotlin
  fun createSimpleTable() = createHTML().
    table {
        tr {
            td { +"cell" }
        }
    }
  ```
  * 영역 특화 언어라는 개념
    - 익숙한 DSL로 SQL 과 정규식이 있음
    - 프로그래밍 언어와 달리 선언적이며 명령적 특성을 가짐
    - DSL 만으로 애플리케이션을 만들기는 어려움
  * 내부 DSL
    - SQL 문을 kotlin 에서 사용하는 메서드를 통해 만들 수가 있음 by 코틀린 익스포즈드 프레임워크
  * DSL의 구조
    - gradle : 람다를 통한 중첩구조 사용
    ```gradle
    dependencies {
      compile("junit:junit:4.11")
    }
    ```
  * 내부 DSL로 HTML 만들기
    - 테이블 예시
    ```kotlin
    fun createSimpleTable() = createHTML().
      val numbers = mapOf(1 to "one", 2 to "two")
     
      for ( (num, string) in numbers) {
        
        tr {
          td { +"$num" }
          td { +string }
        }
      }
    ```

- 11.2 구조화된 API 구축: DSL에서 수신 객체 지정 DSL 사용
  * 수신 객체 지정 람다와 확장 함수 타입
    - fun <T> T.apply(block: T.() -> Unit): T { block() return this }
  * 수신 객체 지정 람다를 HTML 빌더 안에서 사용
    - 예시
    ```kotlin
    open class Tag
    class TABLE: Tag { fun tr(init: TR.() -> Unit) }
    class TR: Tag { fun td(init: TR.() -> Unit) }
    class TD: Tag
    
    fun createSimpleTable() = createHTML().
        table {
            (this@table).tr { //this@table type 은 TABLE
                (this@tr).td { //this@tr type 은 TR
                    +"cell"
                }
            }
        }
    ```
  * 코틀린 빌더: 추상화와 재사용을 가능하게 하는 도구
    - kotlinx.html 을 사용하면 div, button, ul, li 같은 함수 사용 가능
  * invoke 관례와 함수형 타입
    - 람다를 함수처럼 호출하면 이 관례에 따라 invoke 메서드 호출로 변환 됨
  * DSL의 invoke 관례: gradle 에서 의존관계 정의
- 11.4 실전 코틀린 DSL
  * 중위 호출 연쇄: 테스트 프레임워크의 should
    - s should startWith("kot") s값이 kot 으로 시작하는지 체크
  * 원시 타입에 대한 확장 함수 정의: 날짜 처리
    - val Int.days: Period = get() Period.ofDays(this)
  * 멤버 확장 함수: SQL 을 위한 내부 DSL
  * 안코: 안드로이드 UI를 동적으로 생성하기
    - AlertDialogBuilder 패턴


<br></br>
### 부록
- 에코 시스템
  * 테스팅 https://github.com/kotlintest/kotlintest
  * 의존관계 주입 https://github.com/kotlintest/kotlintest
  * JSON 직렬화 잭슨-모듈-코틀린, 콧슨, 클락슨
  * HTTP 클라이언트 retrofit 를 우선적 사용, okhttp or Feul 이 있
  * 웹 어플리케이션 스프링, 버텍스
  * 데이터베이스 접근 - 익스포즈드
  * 유틸리티와 데이터 구조 - RxJava, RxKotlin, funKTionale, 커버넌트
  * 데스크탑 프로그래밍 - 토네이도FX

### 코틀린 언어 버전 변경 이력 정리
- D.1 코틀린 1.1
  * 타입 멸병
    - typealias 타입 별명
  * 봉인 클래스와 데이터 클래스
    - sealed & data class
    - toString() 함수를 쉽게 얻을 수 있는 장점
  * 바운드 멤버 참조
  * 람다 파라미터에서 구조 분해 사용
    - person.forEach { (name, age) -> println("$name $age") } 
  * 밑줄로 파라미터 무시
  * 식이 본문인 게터만 있는 일기 전용 프로퍼티의 타입 생략
  * 프로퍼티 접근자 인라이닝
    - inline 을 접근자에 선언 가능
  * 제네릭 타입으로 enum 값 저근
    - inline fun <refified T: Enum<T>> enumValues(): Array<T>
  * DSL 수신 객체 제한
  * 로컬 변수 등을 위임
    - by 키워드 사용
  * 위임 객체 프로바이터
  * mod & rem
    - mod 대신 rem 이 % 연산자로 해석
    - BigInteger 구현과 다른 정수형 타입의 % 연산 결과를 맞추기 위함
  * 표준 라이브러리 변화
    - 문자열 <-> 숫자
    - onEach() collection 이나 sequence 를 다시 반환
    - also() ,takeIf(), takeUnless()
    - groupingBy()
    - toMap(), toMutableMap() 맵 복사
    - minOf(), maxOf() 둘 또는 세 값 중 최소 최대
    - 람다를 사용한 리스트 초기화
    - Map.getValue()
    - 추상 컬렉션 클래스
    - 배열 연산 추가 (비교 연산, 해시 코드환 계산, 문자열 변)
  * 기타
    - jvm-target 1.8 옵션 추가
    - const val 값을 바이트코드에 인라이닝
<br></br>
- D.2 코틀린 1.2
  * 애노테이션의 배열 리터럴
    - arrayOf 로 전달 안하고 [] 로 가능해짐
  * 지연 초기화 개선
    - lateinit 최상위 프로퍼티 및 지역 변수에 사용 가능
  * 인라인 함수의 디폴트 함수 타입 파라미터 지원
  * 함수/메서드 참조 개선 ::foo this 없이 맥락 해석 가능
  * 타입 추론 개선 - 컴파일러가 제네릭 타입도 잘 감지해줌
  * 경고를 오류로 처리
    - -Werror 지정시 모든 경고를 오류로 처리 gradle 설정으로 가능함
  * 스마트 캐스트 개선
  * 이넘 원소 안의 클래스는 내부 클래스로
    - inner 사용 가능
  * (기존 코드를 깨는 변경) try 블록의 스마트 캐스트 안정성 향상
  * 사용 금지 예고된 기능
    - 다른 클래스를 상속한 하위 데이터 클래스의 자동 생성 copy로 인한 문제
    - 이넘 원소 안에서 중첩 타입 정의
    - 가변 인자에게 이름 붙인 인자로 원소 하나만 넘기기
    - Throwable 확장하는 제네릭 클래스의 내부 클래스
    - 읽기 전용 프로퍼티를 뒷받침하는 필드 엎어쓰기
  * 표준 라이브러리
    - 호환 패키지 분리
    - 컬렉션 (chunked/chunkedSequence, windowed/windowedSequence, ZipWithNext)
    - 리스트 원소 처리 (fill, replaceAll, shuffle, shuffled)
    - kotlin.math (상수, 삼각함수, 하이퍼볼릭 함수, 지수 함수, 로그 함수, 올림/내림, 부호와 절댓값, 2 값의 최대값/최솟값, 부동소수점의 2진 표현)
  * JVM 백엔드 변경
    - 생성자 호출 정규화
    - 자바 디폴트 메서드 호출
    - (기존 코드를 깨는 변경) 플랫폼 코드의 x.equals(null) 동작 일관성
  * 1.2에 추가된 실헝 기능: 다중 플랫폼 프로젝트

<br></br>
- D.3 코틀린 1.3
  * 코틀린 네이티브 개선: 멀티 플랫폼
  * 다중 플랫폼 프로젝트
  * 컨트랙트(Contract, 계약)
    - 널 가능성이 없으면 자동으로 널이 될 수 없는 타입으로 캐스팅
  * When 대상을 변수에 포획
  * 인터페이스의 동반 객체에 있는 멤버를 @JvmStatic or @JvmField 어노테이
  * 파라미터 없는 메인
  * 함수 파라미터 수 제한 완화 (255개 까지 처리 가능)
  * 프로그래시브 모드
    - 인라인 클래스 (실험적 기능)
  * 부호 없는 정수 (실험적 기능)
  * @JvmDefault (실험적 기능)
    - 인터페이스에 사용
  * 표준 라이브러리 변화
    - Random()
    - isNullOrEmpty, orEmpty
    - 배열 원사 복사 확장 함수 copyInto()
    - 맵 연관 쌍 추가 함수 associateWith()
    - ifEmpty & ifblank
    - 봉인된 클래스에 접근할 수 있는 리플렉션 추가
    - 기타 소소한 변경
  * 코틀린 1.3.10에 도입된 변화
  * 코틀린 1.3.20에 도입된 변화
    - gradle & dsl 개선 
  * 코틀린 1.3.30에 도입된 변화
    - 부호 없는 정수 타입 정확하게..
  * 코틀린 1.3.40에 도입된 변화
    - 코틀린/js 에서 NPM gradle, 얀, 웹팩 지원
    - 다중 플랫폼 프로젝트 테스트 러너 개선
    - 코틀린/네이티브에서 성능과 상호 운용성 개
  * 코틀린 1.3.50에 도입된 변화
    - 새로운 시간 측정과 기간
    - 자바 -> 코틀린 변환기 개선
    - 두캇을 사용해 그레이들 코틀린/js 프로젝트의 npm 의존관계를 외부에 정의
    - 코틀린/네이티브 디버깅 플로그인
    - 다중 플랫폼 프로젝트의 자바 컴파일 지
  * 코틀린 1.3.60에 도입된 변화
    - 인라인 클래스간의 비교 최적화
    - 디버깅 도구, J2K 변환기, 코틀린으로 작성한 그레이들 스크립트 개선
    - 더 많은 코틀린/네이티브 플랫폼/타겟 지원
    - 코틀린/MPP IDE 개선
    - 코틀린/js 소스맵 지원 추가
    - 코틀린/js 테스트 러너 통합 개선
  * 코틀린 1.3.70에 도입된 변화
    - StringBuilder 확장 기능
    - JVM KClass 사용시 kotlin-reflect 의존하지 않음
    - Clock & ClockMark 시간 측정 API
    - 데크 구현 ArrayDeque
    - 컬렉션 빌더 builderList, buildSet, buildMap 추가
    - randomOrNull(), replaceOrNull() 추가
    - fold(), scan()
  * 코틀린 1.3.70에 도입된 변화
  
  

<br></br>
- D.4 코틀린 1.4
  * 1.4의 언어 변화
    - SAM 인터페이스와 람다 변환
    - 위치 기반 인자
    - 배열 원소 끝에 , 가능
    - 호출 가능 참조가 개선
    - 디폴트 인자가 있는 경우 함수 참조를 사용하면 디폴트 인자 값이 자동으로 적용
    - 루프 안에 있는 when 내부의 절에서 break or continue 사용시 when 을 둘러싸는 가장 가까운 루프로 제어가 이동
  * 라이브러리 변화
    - ArrayDeque 데큐 추가
    - 배열고 ㅏ컬렉션의 새 함수 추가
  * 언어 변화
    - JVM 레코드 지원이 프리뷰로 추가
    - 봉인된 인터페이스 제공 (실험)

<br></br>
- D.5 코틀린 1.5
  * 1.5의 언어 변화
    - 봉인된 인터페이스
    - JVM 레코드 지원
    - 인라인 클래스 도입
  * 라이브러리 변환
    - 부호 없는 정수 타입 안정화
    - 로케일 무시하는 대소문자 변환 함수가 추가 to~~ 로 시작하는 것들
    - 정수 연산 함수 floorDiv(), mod()

<br></br>
- D.6 코틀린 1.6
  * 1.6의 언어 변화
    - Enum & 봉인된 클래스 ,boolean 타입의 값에 대한 when 문에서 모든 경우 처리하지 않으면 경고
    - 일시 중단 함수 타입을 상위 클래스로 하는 클래스 선언


<br></br>
### 코루틴과 Async/Await
- E.1 코루틴이란
  * 컴퓨터 프로그램 구성 요소 중 하나로 비선점형 멀티태스킹을 수행하는 일반화한 서브루틴
  * 일시 중단(suspend) 하고 재개(resume) 할 수 있는 여러 진입 지점(entry point)을 허용함
  * 메서도도 서브루틴이고 호출할 때 마다 활성 레코드라는 것이 스택에 할당되면서 서브루틴 내부의 로컬 변수 등이 초기화
  * 코루틴 : 서로 협력해서 실행을 주고 받으면서 작동하는 여러 서브루틴
  ```kotlin
  generator countdown(n) {
      while (n > 0) {
          yield n
          n -= 1
      }
  }
  
  for i in countdown(10) {
      println(i)
  }
  ```
  * 서부루틴은 호출시 처음부터 끝까지 실행되고 다시 불러도 동일함. 반면에 코루틴은 yiled 로 실행을 양보했던 지점부터 실행을 계속할 수 있음
- E.2 코틀린의 코루틴 지원: 일반적인 코루틴
  * 메이븐 * gradle
    - kotlinx-coroutines-core
  * E.2.1 여러 가지 코루틴
    - 코루틴을 만들어주느 코루틴 빌더
      + kotlinx.coroutines.CoroutineScope.launch
        * 잡(job) 을 반환하고 만들어지면 즉시 실행하고 cancel() 을 통해 중단 가능
        * suspend 함수 내부라면 CoroutineScope 가 있겠지만 그렇지 않은 경우 GlobalScope
        * runblocking 을 통해 종료될 때까지 현재 쓰레드를 블록 (코루틴스코프의 확장함수가 아님)
      + kotlinx.coroutines.CoroutineScope.async
        * async Deffered 를 반환하고 이는 Job 을 상속한 클래스
        * Deffered 에는 await() 함수가 정의되어 있는데 코루틴이 계산하고 돌려주는 값의 타입 (Job 은 Unit)
  * E.2.2 코루 컨텍스트와 디스패처
    - CoroutineContext 는 실행한 여러 작업과 디스패처를 저장하는 일종의 맵 역할을 
  * E.2.3
    - produce: 정해진 채널로 데이터를 스트림으로 보내는 코루틴을 만듬. ReceiveChannel<> 반환
    - actor : 정해진 채널로 메시지를 받아 처리하는 액터를 코루틴으로 만듬 SendChannel<> 채널의 send() 사용
    - withContext : 다른 컨텍스트로 코루틴 전환
    - withTimeout: 정해진 시간안에 실행되지 않으면 예외 발생
    - withTimeoutOrNull: 정해진 시간안에 실행되지 않으면 null 결과로
    - awaitAll: 모든 작업의 성공을 기다린다. 작업 중 어느 하나가 예외로 실패하면 awaitAll도 그 예외로 실패
    - joinAll: 모든 작업이 끌날 때까지 현재 작업을 일시 중단

- E.3 suspend 키워드와 코틀린의 일시 중단 함수 컴파일 방법
  * suspend() 안세머나 yield 사용
  * 작동원리
    - 코루틴에 진입할 때 나갈 때 실행 중이던 상태를 저장하고 복구하는 작업 진행
    - 현재 실행 중이던 위치 저장하고 재개될 때 해당위치 부터 실행
    - 다음에 어떤 코루틴을 실행할지 결정
    - 컨티뉴에이션 패싱 스타일(CPS) 변환과 상태 기계를 활용하여 코드를 생

- E.4 코루틴 빌더 만들기
  * E.4.1 제네레이터 빌더 사용법
    - sample
    ```kotlin
    fun idMaker() = generate<Int, Unit> {
      var index = 0
      while (index < 3) yield(index++)
    }
    
    fun main() {
      val gen = idMaker()
      println(get.next(Unit))
      println(get.next(Unit))
    }
    ```