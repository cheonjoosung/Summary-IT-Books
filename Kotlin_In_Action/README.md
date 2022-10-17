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


<br></br>
### 5장 람다로 프로그래밍
- 5.1 람다 식과 멤버 참조
  * 람다 소개: 코드 블록을 함수 인자로 넘기기
    - android 클릭리스너를 onClick() 메소드를 구현해야할 때 불필요한 코드를 추가해야 했지만 람다식은 간편
    ```kotlin
    btn.setOnClickListener { /* 동작수행 */    }
    ```
  * 람다와 컬렉션
    - .maxBy{ it.age } or .mayBy(Person::age)
  * 람다 식의 문법
    - val sum = {x: Int, y: Int -> x + y }
  * 현재 영역에 있는 변수에 접근
  * 멤버 참조
    - 클래스::멤버 멤버 참조
- 5.2 컬렉션 함수형 API
  * 필수적인 함수: filter, map
    - filter 이터레이션하면서 주어진 람다에 각 원소를 넘겨서 true 만 반환하는 원소만 모음 (원소를 반환하지 않음)
    - map 함수는 람다에 컬렉션의 각 원소에 적용한 결과를 모아서 새 컬렉션을 만듬
  * all, any, count, find: 컬렉션에 술어 적용
    - count() 가 size 보다 효율적임 (조건을 만족하는 원소를 따로 저장하지 않기에...)
  * groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경
    - key 와 객체로 그룹핑해서 결과를 반환함
  * flatMap & flatten: 중첩된 컬렉션 안의 원소 처리
    - "abc", "def" -> "abcdef" 를 flatten
- 5.3 지연 계산(lazy) 컬렉션 연산
  * 시퀀스 연산 실행: 중간 연산과 최종 연산
  * 시퀀스 만들기
- 5.4 자바 함수형 인터페이스 활용
  * SAM(Single Abstract Method) 추상 메소드가 하나인 것
  * 자바 메서드에 람다를 인자로 전달 
  * SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경
    - btn1.setOnClickListener(listener), btn2.setOnClickListener(listener) listener 를 반복사용
- 5.5 수신 객체 지정 람다: with 와 apply
  * with 함수
    - 메서드를 호출하려는 수신 객체를 전달하고 내부적으로 this 를 사용해도 되고 생략도 가능
    - 일반 함수는 일반 람다, 확장 함수는 수신 객체 지정 람다
  * apply 함수
    - 인스턴스를 만들고 apply에 이 값을 전달함
    - 블럭 안에서 해당 수신 객체를 프로퍼티를 설정할 수 있음


<br></br>
### 6장 코틀린 타입 시스템
- 6.1 널 가능성
  * Null 가능성 NPE(NullPointerException 오류)를 피할 수 있도록 도와주는 코틀린 타입 시스템 특징
  * Null 이 될 수 있는 타입
    - Type? 를 통해 해당 인스터스가 널이 될 수 있는 값을 알려줘야 함
  * 타입의 의미
    - 어떤 값들이 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류
    - 코틀린은 컴파일 타입에 모든 검사를 수행하기에 실행 시점 부가 비용이 들지 않음
  * 안전한 호출 연산자: ?.
    - 해당 값이 null 이 아닐 때 뒤의 연산을 수행한다
  * 엘비스 연산자: ?:
    - 이항 연산자로 좌항을 계산한 값이 널인지 체크하고 널인 경우 우항 값을 결과라 할당함
  * 안전한 캐스트: as?
    - as? 를 통해 타입을 원하는 타입인지 쉽게 검사하고 casting 까지 가능
  * 널 아님 단언: !!
    - 널이 될 수 없다고 단정하고 강제로 NotNull 형태로 타입 강제
    - 잘못 생각해도 예외를 감수하겠다는 의미로 위험한 코드다
  * let 함수
    - Object?.let { } null이 아닐 경우 블록 안의 코드를 실행
  * 나중에 초기화할 프로퍼티
    - lateinit null 로 할당하는 대신에 사용 가능
  * 널이 될 수 있는 타입 확장
    - String 의 isNullOrBlank(), isNullOrEmpty() 널을 명시적으로 검사함
  * 타입 파라미터의 널 가능성
    - t: T 인 경우 널이 도리 수 없는 타입 상한을 지정해야 함
  * 널 가능성과 자바
    - 플랫폼 타입
      + 코틀린이 널 관련 정보를 알 수 없는 타입
      + 자바에서는 널을 체크를 따로 하지 않기에 메소드에 어노테이션을 통해 알려줘야 함
    - 상속
      + 자바 클래스 또는 인터페이스를 코틀린에서 구현할 경우널 가능성을 제대로 처리하는 것이 중요
- 6.2 코틀린의 원시 타입
  * 원시 타입: Int, Boolean 등
  * 널이 될 수 있는 원시 타입: Int?, Boolean? -> Wrapper로
  * 숫자 변환
    - 다른 타입의 숫자로 자동 변환하지 않고 toLong, toInt 등 직접 호출 필요
    - 1.1 이상부터 숫자 사이에 _ 넣을 수 있음
  * Any, Any?: 최상위 타입
    - Any 는 자바에서 Object 클래스처럼 조상 타입임
  * Unit 타입: 코틀린의 패ㅑㅇ
    - 반환 타입을 명시하지 않거나 : Unit 으로 쓴 경우
    - void 와 달리 타입 인자로 사용ㅇ 가능
  * Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다
    - 정상적으로 끝나지 않는다는 표현 제공
    - 함수의 반환 타입이나 타입 파라미터로 쓰임
- 6.3 컬렉션과 배열
  * 널 가능성과 컬렉션
    - List<Int?> List<Int>? 는 전혀 다름. 원소가 null이 될수있는 것과 전체 리스트가 null 이 될수 있는 것의 차이
    - filterNotNull 를 통해 List 에서 Null 이 아닌것을 필터할 수 있음
  * 읽기 전용과 변경 가능한 컬렉션
    - Collection 을 이용하여 만든 것이 MutableCollection 이고 add,remove, clear 등의 원소 추가,삭제,클리어 등 기능
    - 읽기 전용 컬렉션이 항상 스레드 안전하지 않다는 점 명심
  * 코틀린 컬렉션과 자바
    - Iterable 를 구현한 MutableIterable, Collection 를 구현한 MutableCollection 자바 컬렉션 인터페이스의 구조를 그대로 옮겨 놓음
  * 컬렉션을 플랫폼 타입으로 다루기
    - 컬렉션이 널이 될 수 있는가? 원소가 널이 될 수 있는가? 오버라이드하는 메서드가 컬렉션을 변경할 수 있는가 등
  의 사항을 반영해서 코드를 작성해야 함
  * 객체의 배열과 원시 타입의 배열
    - Array...


<br></br>
### 7장 연산자 오버로딩과 기타 관례
- 7.1 산술 연산자 오버로딩
  * 이항 산술 연산 오버로딩
    - operator 키워드를 붙여서 관례 표시
    - plus, minus, mod(1.1 부터 rem), div, times 가 함수이름
    - 코틀린 연산자가 자동으로 교환법칙을 지원하지 않음
  * 복합 대입 연산자 오버로딩
    - +=, -=, *=, /= 와 같은 연산자
    - plus, plusAssign 을 동시에 정의 no
  * 단항 연산자 오버로딩
    - unaryMinus, unaryMinus, ! - not, ++a or a++ - inc, --a or a-- - dec
- 7.2 비교 연산자 오버로딩
  * 동등성 연산자: equals
    - 자바에서는 객체 비교를 위해 equals or compareTo 를 사용하지만 코틀린은 필요없음
    - a == b => a?.equals(b) ?: (b == null)
    - === 식별자 비교 연산자로 두 객체가 같은 값을 가르키는지 == 식별자는 값이 같은지
  * 순서연산자: compareTo
    - a >= b 연산은 a.compareTo(b) >= 0 으로 변겨오디어 계산 됨
    - compareValuesBy(this, obj2, Class::property1, Class:property2) 순차적으로 비교
- 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례
  * 인덱스로 원소에 접근: get & set
    - 인덱스 연산자 [] 를 활용 함
    - x[a, b] -> x.get(a, b) 로 치환
  * in 관례
    - in 은 컬렉션에 들어가있는지 검사로 contains() 와 대응한다.
  * rangeTo 관례
    - start..end start 이상 end 이하의 범위 start.rangeTo(end)
  * for 루프를 위한 iterator 관례
    - for (x in list) -> list.iterator() 를 호출
- 7.4 구조 분해 선언과 component 함수
  * val (a, b) = p 는 a = p.component1, b = p.component2 순차적 할당으로 구조 분해 선언을 의미한다.
  * 구조 분해 선언과 루프
    - map 의 엔트리에서 component1, component2 를 꺼내 key,value 대입 가능
- 7.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티
  * 위임 - 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴
  * 위임 프로퍼티 소개
    - var p: Type by Delegate()
    - 컴파일러에 의해 생성된 get/set Value 를 통해 p의 값을 설정함
    - 프로퍼티 위임을 사용해 초기화 지연이 가능
  * 위임 프로퍼티 사용: by lazy() 사용한 프로퍼티 초기화 지연
    - 객체의 일부분을 초기화하지 않고 필요한 경우에 초기화하여 사용하는 패턴
    ```kotlin
    class Email {}

    class Person(val name: String) {
    private var _emails: List<Email>? = null
    
        val emails: List<Email>
            get() {
                if (_emails == null) {
                    _emails = emails
                }
                return _emails!!
            }
    }
    ```
    - 뒷받침하는 프로퍼티 기법 객체의 인스터스를 만들때는 emails 이 없고 emails 접근할 때 초기화를 진행
  다만 쓰레드 안전성이 떨어지고 프러포티가 많아질 수록 코드가 기하급수적으로 커진다
    ```kotlin
    val emails by lazy { loadEmails(this) }
    ```
    - lazy 함수는 기본적으로 쓰레드 안전함. 동기화에 사용할 락을 lazy 함수에 전달할 수도 있고 다중 스레드 환경에서
  사용하지 않을 프로퍼티를 위해 lazy 함수가 동기화를 하지 못하게 막을 수 있음
  * 위임 프로퍼티 구현
    - 프로퍼티가 바뀔 때마다 리스너에게 변경 통지할 경우 (UI 변경)
  * 위임 프로퍼티 컴파일 규칙
  * 프레임워크에서 위임 프로퍼티 활용

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
    - 
      