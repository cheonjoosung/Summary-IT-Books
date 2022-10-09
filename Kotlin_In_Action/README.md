# Kotlin In Action

## 1부 코틀린 소개
### 1장 코틀린이랑 무엇이며, 왜 필요한가?
- 1.1 코틀린 맛보기
- 1.2 코틀린 주요특성
  * 멀티 플랫폼 가능
    + 서버, 안드로이드 등
  * 정적 타입 지정 언어
    + 컴파일러가 문맥을 확인하여 타입 추론(성능, 신뢰성, 유지 보수성, 도구지원)
  * 함수형 프로그래밍 & 객체지향 프로그래밍
    + 일급 시민인 함수 - 함수를 변수에 저장 또는 인자로 전달
    + 불변성 - 내부 상태가 바뀌지 않는 불변 객체를 사용
    + 부수 효과 없음 - 입력이 같으면 출력이 같음 (순수함수 - 테스트편이성)
    + 이러한 특성으로 인해 Thread-Safe
  * 무료 오픈소스
- 1.3 코틀린 응용
  * 서버 프로그래밍
  * 코틀린 안드로이드 프로그래밍
    + Compose 구성 
    + NPE 크게 줄어듬
    + 코틀린 표준 라이브러리 함수는 인자로 받은 람다 함수를 인라이닝 한다.
      람다를 사용해도 새로운 객체가 만들어지지 않으므로 가비지 컬렉션이 늘어나지 않음
- 1.4 코틀린 철학
  * 실용성
  * 간결성 - data class 묵시적으로 제공하는 함수들이 많음 (get, set, toString, hashCode 등)
  * 안정성 - NPE 가능성을 크게 줄여줌
  * 상호운용성 - 자바 콜렉션을 사용하고 이를 추가하여 확장하는 방식을 사용
- 1.5 코틀린 도구 사용
  * 코틀린 코드 컴파일
    + .kt -> .class 컴파일러에 의해 변환하고 실행시 코틀린 런타임을 통해 실행
  * 인텔리J & 안드로이드 스튜디오
  * 대화형 셀
  * 이클립스 플러그인
  * 온라인 놀이터 - 플레이그라운드
  * 자바-코틀린 변환기
    
<br></br>
### 2장 코틀린 기초
- 2.1 기본 요소: 함수와 변수
  * Hello World
    + 함수 fun 키워드 사용
  * 함수
    + 식이 본문인 함수
    ```kotlin
    fun add(a: Int, b: Int) = a + b
    fun max(a: Int, b: Int) = if (a > b) a else b
    ```
  * 변수
    + val 변경 불가능한 참조를 저장하는 변수
    + var 변경 가능한 참조
  * 문자열 템플릿
    ```kotlin
    val hello = "Hello $name"
    ```
- 2.2 클래스와 프로퍼티
  * 프로퍼티
    + val 멤버변수는 get 제공, var 멤버변수는 get, set 제공
  * 커스텀 접근자
    ```kotlin
    val isGood: Boolean
      get() {
        return message == "Good"
      }
    ```
  * 코틀린 소스코드 구조: 디렉터리와 패키지
- 2.3 선택 표현과 처리: enum & when
  * enum
    ```kotlin
    enum class Color { RED, BLUE }
    ```
  * when 으로 enum 사용
    ```kotlin
    enum class Color { RED, BLUE }
    
    fun getColor(color: Color) = 
       when (color) {  
         Color.RED -> "RED"
         Color.BLUE -> "BLUE"
       }
    ```
  * when & 임의의 객체를 함께 사용
  * 인자없는 when 가능
  * 스마트 캐스트 : 타입 검사와 타입 캐스트를 조합
  * 리팩토링: if -> when 으로 변경
    + when 은 해당 구문만 실행하지만 if-else는 전체를 실행할 수 잇음
  * if - when 분기에서 블록 사용
- 2.4 대상을 이터레이션: while & for 루프
  * while 루프
  * 수에 대한 이터레이션 범위와 수열
  * 맵에 대한 이터레이션
  * in으로 컬렉션이나 범위의 원소 검사
    ```kotlin
    fun isAlphabet(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
    ```
- 2.5 코틀린의 예외처리
  * try, catch, finally
    + throws 절이 없음
  * try 를 식으로 사용

<br></br>
### 3장 함수 정의와 호출
- 3.1 코틀린에서 컬렉션 만들기
  * 코틀린이 자체 컬렉션을 제공하지 않는 이유는
    + 자바 컬렉션 활용하면 상호작용이 쉬움
    + 서로 변환할 필요가 없음
- 3.2 함수를 호출하기 쉽게 만들기
  * 이름 붙인 인자
    ```kotlin
    println(joinToString(list, separtor = ";", prefix = "(", postfix = ")"))
    ```
  * 디폴트 파라미터 값
    + 자바 일부 클래스에서 오버로딩 메서드가 너무 많음
    + 함수 인자 받을 때 없는 경우 = "" 식으로 값을 세팅할 수 있음
  * 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티
- 3.3 메서드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
  * 확장함수
    + 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스 밖에 선언된 함수
  * 수식 객체 & 타입
    ```kotlin
    fun String.lastChar(): Char = get(length - 1)
    // 수신객체타입은 는 String이 되고 수신객체는 넘어오는 값 
    ```
  * 임포트와 확장 함수
  * 자바에서 확장 함수 호출
  * 확장 함수로 유틸리티 함수 정의
  * 확장 함수는 오버라이드할 수 없음
  * 확장 프로퍼티
- 3.4 컬렉션 처리: 가변 길이 인자, 중위함수 호출, 라이브러리 지원
  * vararg 키워드, 중위 함수 호출 구문, 구조 분해 선언
  * 자바 컬렉션 API 확장
    + last() list의 확장함수
    ```kotlin
    public fun <T> List<T>.last(): T {
      if (isEmpty()) throw NoSuchElementException("List is empty.")
      return this[lastIndex]
    }
    ```
  * 가변 인자 함수: 인자의 개수가 달라질 있는 함수 정의 - vararg
  * 값의 쌍 다루기: 중위 호출과 구조 분해 선언 - 1 to "one"
- 3.5 문자열과 정규식 다루기
  * 문자열 나누기 - split()
  * 정규식과 3중 따옴표로 묶은 문자열
    ```kotlin
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    matchResult?.let {
        val (dir, fileName, ext) = it.destructured
    }
    ```
- 3.6 코드 다듬기: 로컬 함수와 확장

<br></br>
### 4장 클래스 객체 인터페이스
- 4.1 클래스 계층 정의
  * 코틀린 인터페이스
    - : , override 변경자 키워드 사용
    - 자바8 에서는 default 를 이용해서 구현했지만 코틀린에서는 그냥 함수 구현이 가능
    - 코틀린은 자바 6에 맞게 호환되어있고 코틀린 1.5부터 default 메서드 생성
  * open, final, abstract 변경자 : 기본 final
    - 취약한 기반 클래스(fragile base class) 자신을 상속하는 방법에 대해 정확한 규칙을 제공하지 않는 경우
    - 코틀린의 클래스는 기본적으로 final 이기에 상속을 하고 싶다면 open 변경자가 필요함
    - 클래스 내 메소드는 final 이 기본이고 open 이 붙어야 오버라이드 가능, override 키워드 붙은 메소드도 열려있음(final 키워드로 제한 가능)
    - abstract 가 붙은 메소드는 반드시 오버라이드 해야 함
  * 가시성 변경자: 기본적으로 공개(public)
    - 클래의 외부 접근을 제어
    - public, private, protected 는 기본적으로 자바와 같음
    - internal 같은 모듈안에서만 볼 수 있음
  * 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스
    - 자바 static class A - class A, 코틀린 class A - inner class A
  * 봉인된 클래스(sealed class):클래스 계층 정의 시 계층 확장 제한
    - 기본적으로 open 이고 else 식 없이 사용이 가능
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
    
- 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언
  * 클래스 초기화: 주 생성자와 초기화 블록
    - init 블록을 통해 갑ㅅ 세팅 가능 
    - 생성자 파라미터에 val 키워드 붙으면 이에 상응하는 프로퍼티가 생성 됨
  * 부 생성자: 상위 클래스를 다른 방식으로 초기화
    - constructor(ctx: Context), constructor(ctx: Context, attr:AttributeSet) 부 생성자들을
  안드로이드에서 많이 볼 수 있다. : super() 를 통해 상위 클래스 생성자 호출 가능
    - this() 를 통해 다른 생성자에게 위임 가능
  * 인터페이스에 선언된 프로퍼티 구현
  * 게터 & 세터에서 뒷받침하는 필드에 접근
    - field 라는 식별자 사용
  * 접근자의 가기성 변경
    - 접근자의 가시성은 프로퍼티의 가기성과 같지만 get/set 앞에 가시성 변경자를 붙여 변경 가능
  
- 4.3 컴파일러가 생성한 메서드: data class & class 위임
  * 모든 클래스가 정의해야 하는 메서드
    - toString(), equals(), hashCode()
  * data class: 메소드 자동 생성
    - 인스터간 비교, 해시 기반 컨터네이너, 문자열 표현을 위한 메서드 자동 생성
    - copy() - 일부 프로퍼티를 변경하여 새로운 객체를 만들수 있음
  * 클래스 위임: by 키워드
    - 데코레이터 패턴 : 상속을 허용하지 않는 클래스 대신 사용할 수 있는 새로운 클래스를 만들되 기존 클래스와 같은 인터페이스를
  데코레이터가 제공하게 만들고
    - : MutableCollection<T> by innerSet 뮤터블콜렉션의 구현을 innerSet에게 위임, 뮤터블 콜렉션에서
  사용하는 메소드를 직접구현

- 4.4 object 키워드: 클래스 선언과 인스터스 생성
  * 객체 선언(싱글턴), 동반객체(companion object), 무명 내부 클래스 대신 사용
  * 객체 선언: 싱글턴 만들기
    - class 대신 object 키워드를 사용
    - 생성과 동시에 인스턴스가 만들어지기에 생성자 정의가 필요 없음
  * 동반객체
    - static 키워드는 코틀린에 없기에 패키지 수준의 최상위 함수 & 객체 선언 시 사용
  * 동반객체를 일반 객체처럼 사용
    - 동반 객체에 이름을 붙임
    - 인퍼테이스를 구현하여 사용
    - 확장함수에 사용 가능
  * 객체 식: 무명 내부 클래스를 다른 방식으로 확장
    - 추상 클래스 또는 인터페이스를 구현한 클래스없이 확장하여 사용 가능 (android clickListener)
  