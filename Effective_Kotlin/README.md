# Summary-Effective-Kotlin

## 1. 좋은코드
### 1-1. 안정성
- #### Item 1. 가변성을 제한하라
  
  + 가변성이 생기면 프로그램 이해&디버깅&테스트이 어려워지며 코드 실행의 추론이 어려워 진다. 또한, 
    멀티스레드 상황에서 적절한 동기화가 필요하다.
    ```kotlin
    var num = 0
    for (i in 1..1000) {
        thread {
            Thread.sleep(10)
            num += 1
        }
    }
    
    num = 0
    val lock = Any()
    for (i in 1..1000) {
        thread {
            Thread.sleep(10)
            synchronized(lock) {
                num += 1
            }
        }
    }
    ```
  + synchronized 을 통한 동기화가 필요하고 가변이 되는 곳마다 동기화 처리가 필요해지므로 프로그램 난이도가 올라간다.
  <br></br>
  + **코틀린에서 가변성 제한하기**
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
      * immutable 객체 사용 이유
        - 한 번 정의된 상태가 유지
        - immutable 객체 공유 시 충돌이 발생하지 않음 병렬 처리 가능
        - immutable 객체에 대한 참조가 변경되지 않으므로 쉽게 캐시 가능
        - immutable 객체는 방어적 복사본을 만들 필요가 없음
        - mutable or immutable 객체로 만들 때 활용하기 좋음  
      ```kotlin
      data class Person(val name: String, val age: Int)
      
      val user1 = User("first", 1)
      val user2 = user.copy(name = "second")
      ```
  <br></br>

    + 다른 종류의 변경 가능 지점
      ```kotlin
      val list = mutalbleListOf()
      var list2 = listOf()
      ```
      * list & list2 의 차이점은 변경가능한 것이 내부냐 외부냐에 있다 -> 멀티쓰레드 상황에서의 처리가 필요
    + 변경 가능 지점 노출하지 말기
      * class 내 copy를 사용하여 방어적 복제를 전달하여 돌발적인 수정을 막는다
      * 클래스 내 컬렉션 객체를 읽기 전용으로 만들어 가변성을 제한
    <br></br>

- #### Item 2 변수의 스코프를 최소화하라
  + 프로퍼티보다는 지역 변수를 사용
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
      
      // 변수를 선언할 때 초기화를 한 번에
      val user2: User
      if (hasValue) user = getValue() else User()
      
      val user2 = if (hasValue) getValue() else User()
      ```
    + 여러 프로퍼티 한꺼번의 설정할 경우 (if 보다는 when 사용)
      ```kotlin
      fun update(degree: Int) {
          val (description, color) = when {
            degree < 5 -> "cold" to Color.BLUE
            degree <23 -> "mild" to Color.YELLOW
            else -> "hot" to Color.RED 
          }     
      }
      ```
  <br></br>
  + 캡쳐링
    - sequence 지연으로 인해 변수를 캡쳐해서 발생됨
    - 변수는 사용하는 범위에 맞춰 최대한 작게 만들어야 함
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
- 리시버를 명시적으로 참조하
    + let, run, apply, also 등을 사용하면 return 으로 this, it 등의 확장 리시버를 받는다
    + 중첩으로 된 구조에서 명시적인 참조를 안하면 의도치 않게 다른 값을 참조하는 실수를 범할 수 있다.
    + DslMarker annotation 사용 (DSL 사용 )

<br></br>
- 프로퍼티는 동작이 아니라 상태를 표시
    + 자바의 필드와 코틀린의 프로퍼티는 엄연히 다름
    ```kotlin
      String name = null
    
      var name:String? = null
        get() = field?.toUpperCase()
        set(value) {
            if (!value.isNullOrBlank()) field = value
        }
    ```
    + 코틀린의 프로퍼티는 캡슐화 되어 있음
    + 확장 프로퍼티 (안드로이드) - preferences, inflater 만으로 접근이 가능
    ```kotlin
      val Context.preferences:SharedPreferences
            get() = PreferenceManager.getDefaultShagedPreferences(this)
      val Context.inflater: LayoutInfalte
            get() = getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflator 
    ```
    

<br></br>
- 이름있는 아규먼트 사용
    + 디폴트 아규먼트
    + 같은 타입의 파라미터 많은 경우
    + 함수 타입의 파라미터가 있는 경우 
    ```kotlin
    //자바 스타일
     observable.getUsers()
            .subscribe((List<User> users) -> { // onNext
                
            }, (Throwable throwable) -> { // onError
                
            }, () -> { // onCompleted
                
            }) 
  
    //코틀린 스타일 subscribe -> subscribeBy(자바 함수를 호출할 때 이름있는 
    // 파라미터 사용할 수 없어서 별도 코틀린 함수 만들어야 함)
     observable.getUsers()
            .subscribe(
                onNext = { users: List<User> -> },
                onError = { throwable: Throawable ->  },
                onCompledted = { _ -> }  
            )
    ```

<br></br>
- 코딩 컨센션 지키기
    + InteliJ 포매터 사용
    + 밑줄 그어지면 변경하고 자동정렬하면 됨
      
      
<br></br>
## 2. 코드설계
### 2-1. 재사용성
- knowledge를 반복하여 사용하지 말라
  + "이미 있던 코드를 복사해서 붙여넣고 있다면 잘못된 것"
  + 로직 & 공통 알고리즘
    - 비즈니스 로직은 시간에 따라 변하지만 공통 알고리즘은 덜한 편
    - 모든 것(디자인 표준, 가이드, 라이브러리 등)이 변하기에 ctrl c+v 
      가 잘못된 확률이 높음
    - 버튼을 복붙에서 그대로 사용할 때 디자인이 바뀌면 다 고쳐야 함
      -> 즉 추상화를 통해 구현하면 됨
  + 단일 책임 원칙
    - 두 액터가 동일한 클래스를 수정하면 안 됨
    - 서로 다른 곳에서 사용하는 knowledge는 독립적으로 변경될 가능이 높기에
      서로 다른곳에 두는것이 좋음

<br></br>
- 일반적인 알고리즘을 반복해서 구현하지 말라
  + 이미 있는 알고리즘은 구현하지 말고 그대로 써라
    * 작성 속도도 빨라지고 함수 이름만 보고 이해가 가능함
  ```kotlin
    val number = 20
  
    val percent = when { 
        number > 100 -> 100
        number < 0 -> 0
        else -> number
    } 
  
    val percent = number.coereIn(0, 100)
  ```
  + 표준 라이브러리 
    * forEach를 통해 값을 넣지 말고 map을 활용하여 사용 
  ```kotlin
    val entries = item.sources.map(::sourceToEntry)
  
    fun sourceToEntry(source: Source) = SourceEntry()
        .apply {
            id = source.id
            // 코드
        }
  ```
  
<br></br>
- 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라
  + lazy or Delegate(observable, vetoable, notnull)
    * 다양한 패턴을 만들 수 잇음 (리소스 바인딩, 의존성 주입, 데이터 바인딩 등) -> 어노테이션 필요
    * 코틀린에서는 type-safe 하게 만들 수 있음
  ```kotlin
  var observed = false
  var max: Int by Delegates.observable(0) { property, oldValue, newValue ->
    println("Changing max to $newValue")
    observed = true
  }

  var token: String? by LoggingProperty(null)

  private class LoggingProperty<T>(var value: T) {

    operator fun getValue(
      thisRef: Any?,
      prop: KProperty<*>
    ): T {
      println("${prop.name} returned value $value")
      return value
    }
  
    operator fun setValue(
      thisRef: Any?,
      prop: KProperty<*>,
      newValue: T
    ) {
      val name = prop.name
      println("$name changed from $value to $newValue")
      value = newValue
    }
  }
  ```

<br></br>
- 일반적인 알고리즘을 구현할 때 제네릭을 사용하라
  + 프로그램의 안정성이 높아짐. 개발이 편해짐
  + 제네릭 제한
    * 구체적인 타입의 서브타입만 사용하게 타입을 제한
    * Any 타입을 활용하여 nullable이 아닌 타입 표시
    ```kotlin
    fun <T: Comparable<T>> Iterable<T>.sorted(): List { }
    
    inline fun <T, R: Any> Iterable<T>mapNotNull()
    ```

<br></br>
- 타입 파라미터의 섀도잉을 피하라
  + 지역 변수가 외부 스코프의 프로퍼티를 가리는 경우
  + 클래스 타입 파라미터에서도 발생함
  ```kotlin
  class Forest(val name: String) {
     fun addTree(name: String) { 
     }
  }

  interface Tree
  class Birch: Tree
  class Spruce: Tree
  
  class Forest<T: Tree> {
    fun <T: Tree> addTree(tree: T) //독립적으로 동작
  }
  
  val forest = Forest<Birch>()
  forest.addTree(Birch())
  forest.addTree(Spruce())
  ```

<br></br>
- 제너릭 타입과 variance 한정자를 활용하라
  + out 또는 in 으로 관련성을 주고자 할 때
  + out - 공변성(covariant) A가 B의 서브타입일때 Cup[A]가 Cup[B]의 서브타입
  + in - 반변성(contravariant) A가 B의 서브타입일때 Cup[A]가 Cup[B]의 슈퍼타입
  + 함수타입
    * (Int) -> Any 는 (Int) -> Number, (Number) -> Any 등으로 다양하게 작동
    * 파라미터 타입은 contravariant 이고 리턴타입은 covariant 
  + variance 한정자와 안정성능
    * 자바의 배열은 covariant 코틀린은 invariant 묵시적으로 업캐스팅을 한다.
    ```kotlin
    Integer [] numbers = {1, 4, 2, 1}
    Object[] objects = numbers
    objects[2] = "B" //자바에서 에러 아직 Integer이기에.
    ```
    * Response - 네트워크 응답 데이 
    ```kotlin
    Response<T> 인 경우 Any가 예상되면 Int, String 가
    ```

<br></br>
- 공통 모듈을 추출해서 여러 플랫포에서 사용하라
  + 풀스택 개발(JS를 활용) -> 하이브리드 개

<br></br>
### 2-2. 추상화 설계
- 개요
  + 함수형 프로그래밍 커뮤니티에서는 추상화&컴포지션으로 프로그래밍을 구성한다?
  + 자동차는 엔진, 서스펜션, 알터네이터 등 다양한 요소가 있지만 운전자는 핸들만 조작하면 된다.
  -> 추상화가 잘 되어있음
    * 이는 복잡성을 숨기고
    * 코드를 체계화등
    * 만드는 사람에게 변화의 자유를 준다.
  
<br></b>
- 함수 내부의 추상화 레벨을 통일하라
  + 추상화가 잘 되어있는 것은 그 아랫단계는 신경 안써도 되고 내것만 잘 처리하면 

<br></b>
- 변화로부터 코드를 보호하려면 추상화를 사용하라
  + 상수 
    * const val min = 7 보다는 const val MIN_PASSWORD_LENGTH가 좋음
  + 함수 
    * 안드로이드에서 toast, snackbar, showMessage 같은 것을 확장함수로 만들면 편히 사용 가능
  + 클래스
    * 기본적으로 final이고 open은 통해 서브클래스 제공으로 자유도 조금 높아짐. 추상화를 통해 더 자유로움
  을 제공할 수 있음
  + 인터페이스
    * 인터페이스 뒤에 객체를 숨겨 실질적인 구현을 추상화하고 사용자가 추상화된 것에만 의존하게 만든다
    이러면 결함을 줄일 수 있음
    * listOf or Iterable or Collection 은 인터페이스를 리턴하는데 각 플랫폼마다 다른 리스틀 리턴
    최적화 때문에..? 하지만 모두 인터페이스에 맞춰져 있으므로 차이없게 사용이 가능하다?
  + ID 만들기
    * id 값을 숨기고 get을 통해 제공
  + 균형이 필요
    * 팀의 크기, 경험, 프로젝트 규모, 도메인 지식 

<br></b>
- API 안정성을 확인하라
  + 표준 API를 사용해야 함
    * 변경 시 수동 업데이트를 해야 함
    * 개발자는 새로운 API를 배워야 함 - MAJOR.MINOR.PATCH 순으로..

<br></b>
- 외부 API 랩(wrap) 해서 사용
  + 외부 API가 불안정한 경우 랩해서 사용 -> 자유&안정성 높임
  + 래퍼만 변경하면 되므로 대응이 쉬운 편. 라이브러리 교체도 쉬움

<br></b>
- 요소의 가시성을 최소화하라
  + 작은 인퍼테이스가 배우기 쉽고 유지도 쉽다.
  + 가시성 한정자
    * public(디폴트), private, protected, internal(모듈 내부)
    * 모듈은 그레이들, 메이븐, 모듈 단위로 패키지와 다름

<br></b>
- 문서로 규약을 정의하라
  + 확장함수로 토스트 메시지를 만들었다면 다른 사용자는 모를 수도 있음 
    그렇기에, KDoc 주석을 붙여주는 것이 좋음.
  + KDoc 은 /** */ 요약 설명 -> 상세 설명 -> 으로 시작
  + 타입 시스템과 예측
    * 리스코프 치환 원칙

<br></b>
- 추상화 규약을 지켜라
  + 규약은 개발자들의 단순한 합의

<br></b>
### 2-3. 객체 생성
<br></br>
- 생성자 대신 팩토리 함수를 사용하라
  + 생성자의 역할을 대신 해 주는 함수가 팩토리 함수
  ```kotlin
  class MyLinkedList<T> (
    val hear: T,
    val tail: MyLinkedList<T>?        
  )
  
  fun <T> myLinkedListOf(vararg elements: T) :MyLinkedList<T>? {
    if (elements.isEmpty()) return null
    val head = elements.first()
    val elementsTail = elements.copyOfRagne(1, elemetns.size)
    val tail = myLinkedListOf(*elementsTail)
    return MyLinkedList(head, tail)
  }
  
  val list = myLinkedListOf(1, 2)
  ```
  + 생성자와 다르게 이름을 붙여서 사용 ArrayList(3) 보다는 ArrayList.withSize(3)이
    이해하기 더 쉬워보인다.
  + companion, 확장, 톱레벨, 가짜 생성자, 팩토리 클래스의 메소드 등이 있음
  + 이름 가진 - Date.from, EnumSet.of, BigInteger.valueOf, StackWalker.getInstance 등
  + 확장 팩토리 함수
    * 재정의
    ```kotlin
    interface Tool { companion object {} }
    
    Tool.Companion.createBigTool(): BigTool { }
    ```
  + 톱레벨 팩토리 함수
    * listOf, setOf, mapOf
  + 가짜 생성자
    * invoke 함수를 갖는...
  + 팩토리 클래스의 메서드
    * 

<br></b>
- 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라
  + 생성자시 그냥 입력보다는 parameter="" 로 하면 좋음
- 빌더 패턴
  + 자바에서는 이렇게 사용하지만... 그냥 길다.. 명확하지도 않고.. 사용하기도 어렵다, 
  thread-safe 구현도 어려움

<br></b>
- 복잡한 객체를 생성하기 위한 DSL
  + DSL(Domain Specific Language)
  + 사용자 정의 DSL 만들기
    * () -> Unit 아규먼트를 갖지 않고 & 리턴 Unit
    * (Int) -> Unit 아큐먼트 Int & 리턴 Unit
    * (Int) -> () -> Unit Int 아규먼트 & 리턴 함수
    * (()->Unit) -> Unit 아규먼트 함수 & 리턴 Unit
    * 람다 표현식, 익명함수, 함수 레퍼런스
    * 리시버를 가진 람다 표현식으로 정의하면 this 키워드가 이를 참조
      + 일반적인 객체처럼 invoke 메서드 사용
      + 확장 함수가 아닌 함수처럼 사용
    * DSL Builder 를 만들고 파라미터를 활용해서 값들은 처리

<br></br>
### 2-4. 클래스 설계
- 상속보다는 컴포지션을 사용하라
  + 간단한 행위 재사용
    * 로딩 프로그레스바를 추상클래스로 만들고 사용처에서 상속받아 쓸 때 문제점
      + 상속은 클래스의 모든 것을 가져오기에.. 불필요한 것도 가져옴. 이는 객체지향설계중 
        인터페이스 분리 원칙인 ISP를 위반
      + 메소드의 작동방식을 이해하기 위해 슈퍼클래스를 여러 번 확인해야 함
      + 단일 상속이기에 두 기능을 하나의 슈퍼클래스로 두어야 하고 복잡한 계층 구조가 만들어
      ```kotlin
      class Progress {
        fun showProgress() { }
        fun hideProgress() { }
      }
      
      val progress = Progress()
      progress.showProgress()
      ```
  + 모든 것을 가져올 수밖에 없는 상속
    * 일부분을 재사용하기 위한 목적으로는 적합하지 않음
    * 
  + 캡슐화를 깨는 상속
    * 포워딩 메소드 - 인터페이스에서 정의한 메서드를 구현하는 패턴
      + 컴파일 시점에 자동으로 만들어지기도 한다?
  + 오버라이딩 제한하기
    * final 키워드를 사용하면 상속 못함
    * open 클래스는 open 메서드만 오버라이드 할 수 있음
  + 결론
    * 명확한 IS-A 인 경우는 상속이 맞음
    * 상속이 안되도록 설계되지 않은 메소드는 final

<br></br>
- 데이터 집합 표현에 data 한정자를 사용하라
  + toString(), equals & hashCode, copy, componentN(component1, component2) 자동생성
  + copy 는 immutable 를 만들 때 유용, shallow-copy
  + componentN은 순서대로 맵핑을 시킬 수 있음
  + 최소 2개 이상을 가지도록...
  + 튜플 대신 데이터 클래스 사용하기
    * 튜플은 Serializable 기반으로 만들어지고 toString을 사용할 수 있는 제네릭 데이터 클래스
    * Pair, Triple 이 있는데 코틀린에 남아있는 마지막 튜플
    * 집합용도로 표현시는 좋음
      


<br></br>
## 3. 효율성
- 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용
  + SAM(Single Abstract Method) - 메소드가 하나만 있는 인터페이스
  ```kotlin
  interface OnClick {
    fun clicked(view: View)
  }
  
  var onClicked: ((Int) -> Unit)? = null
  ```
    * 인터페이스보다 함수타입을 사용하면 이벤트의 수정이 일어날 때 자유롭게 가능함. 하지만 인터페이스는 구현하는 모든
  클래스를 수정해야하는 문제가 발생함.
  + 언제 SAM 을 사용할까?
    * 코틀린이 아닌 다른 언어세ㅓ 사용할 클래스를 설계할 

<br></br>
- 태그 클래스보다는 클래스 계층 사용
  ```kotlin
  fun match(value: T?) = when (matcher) {
  
  }
  
  enum class Matcher {
    EQUAL, NOT_EQUAL, LIST_EMPTY, LIST_NOT_EMPTY
  }
  ```
  + sealed 클래스를 + abstract 사용하여 구현 
  + 태그 클래스와 상태 패턴의 차이

<br></br>
- equals 규악을 지켜라
  + Any 에는 equals, hashCode, toString 메서드들이 잘 설정된 규약을 가지고 있음
  + 동등성
    * 구조적 동등성 ==, != 값
    * 레퍼런스적 동등성 ===, !== 객체
  + equals 가 필요한 이유
    * 객체를 비교할 때 class 를 사용하면 equals 를 따로 구현해야함 -> data 한정자를 사용(final 임)
  + equals 의 규약
    * 반사적, 대칭정, 연속적, 일관적, null 관련된 동작
  + URL equals
    * https://en.wiki 와 https://wiki 는 사실상 같은 주소인데 인터넷의 연결여부에 따라 달라짐 ->
    설계가 잘못되어 있음.

<br></br>
- hashCode 규악을 지켜라
  + 해시 테이블
    * 해쉬 테이블은 각 요소에 숫자를 할당하는 해시 함수가 필요
    * 같은 요소라면 같은 숫자 리턴
    * 특성 - 해쉬함수는 빠르고, 충돌이 적음
  + 가변성과 관련된 문제
    * LinkedHashSet, LinkedHashMap 의 키는 한번 추가한 요소를 변경할 수 없음
    -> immutable 객체를 많이 사용
  + hashCode 규약
    * 어떤 객체를 변경하지 않는 이상 여러번 호출해도 같은 결과 - 일관성
    * equals 의 결과로 같다면 hashCode 결과도 같다고 나와야 함 - 개발자들이 많이 잊어먹음
    * eqauls 를 오버라이드 하면 hashCode 도 반드시 오버라이드 해야 함
  + hashCode 구현하기
    * data 한정자를 쓰면 알아서 해줌
    * 그게 아니라면 오버라이드 필요, 관례적으로 31을 쓰고 Kotlin/JVM Objects 를 사용
    ```kotlin
    class A {
      val name = ""
      val age: Int = 0

      override fun hashCode(): Int {
          return name.hashCode().let {
              it * 31 + age.hashCode()
          }
      }
    } 
    
    class B {
      val name = ""
      val age: Int = 0

      override fun hashCode(): Int = Objects.hash(name, age)
    } 
    ```
    * 다른 플랫폼 (코틀린 stdlib에는 이러한 함수가 따로 없음)
    ```kotlin
    override fun hashCode(): Int = hashCodeOf(name, age)
    
    inline fun hashCodeOf(vararg values: Any?) = 
       values.fold(0) { acc, value -> 
            (acc * 31) + value.hashCode()
    }
    ```
    
<br></br>
- compareTo 규악을 지켜라
  + o1.compareTo(o2)
  + 비대칭적 동작, 연속적 동작, 코넥스적 동작
  + compareTo 따로 정의해야 할까?
    * 원하는 키로 정렬 시
    ```kotlin
    class User(val name: String, val surname: String)
    val sorted = names.sortBy { it.surname }
    val sorted2 = names.sortedWith(compareBy({it.surname}, {it.name}))
    ```
    * 비교를 자주 사용한다면 class 안에 companion object 안에 넣어서 사용
  + compareTo 구현
    * 리시버 = other 0, 리시비 > other 양수, 리시버 < other 음수

<br></br>
- API의 필수적이지 않는 부분을 확장 함수로 추출하라
  + 함수 내에 멤버 메소드를 -> 외부로 꺼내어 사용
    * 확장 함수는 다른 패키지에 있을 확률이 높음 (데이터 & 행위를 분리하도록 설계한 프로젝트)
    * 같은 이름이 있는 메소드가 있다면 위험할 수 있음
    * 가상이 아님, 상속을 목적으로 설계된 클래스라면 학장 함수 노노
  
<br></br>
- 멤버 확장 함수의 사용을 피하라
  + 어떤 클래스에 대한 확장 함수를 정의할 때 멤버로 추가하는 것은 좋지 않음
    * 레퍼런스를 지원하지 않음 ex)String::isPhoneNumber
    * 암묵접 접근을 할 때, 두 리시버 중에 혼동함

### 3-1. 비용 줄이기
<br></br>
- 불필요한 객체 생성을 피하라
  + JVM 에서 동일한 문자열은 그대로 사용 -> 상수는 String, 문자열 처리는 StringBuilder/Buffer
  + Integer, Long 박스화한 기본 자료형도 작은 경우 캐쉬 (-128 ~ 127) nullable 일 경우 Integer로
  컴파일 되고 그 외의 경우 Int 로 컴파일
  + 객체 생성을 비용은 항상 클까?
    * 객체는 더 많은 용량 차지 (헤더 + 데이터)
    * 요소가 캡슐화되어 있다면 접근에 추가적인 함수 호출
    * 객체는 생성되고 메모리 영역에 할당되고 레퍼런스 만드는 등의 작업
  + 객체 선언
    * 싱글톤 - 객체를 재사용하는 방법
  + 캐시를 활용하는 팩토리 함수
    * fun <T> List<T> emptyList() { return EMPTY_LIST }
    * 모든 순수 함수는 캐싱을 활용할 수 있음 - 메모이제이션
    ```kotlin
    private val FIB_CACHE = mutableMapOf<Int, BigInteger>()

    fun fib(n: Int): BigInteger = FIB_CACHE.getOrPut(n) {
    if (n <= 1) BigInteger.ONE else fib(n-1) + fib(n-2)
    }
    ```
    * WeakReference - 가비지 컬렉터가 값을 정리하는 것을 막지 않음 (미사용시 바로 제거)
    * SoftReference - 가비지가 정리할 수도 안 할수도 있음. JVM이 메모리가 부족할 때만 필요로 의해서
    정리 함. 캐시를 만들 때 SoftReference 를 사용해야 함
  + 무거운 객체를 외부 스코프로 보내기
    * 성능을 위한 유용한 트릭
    * 함수 내에서 정규식을 매번 만드는 것보다 톱 레벨로 빼서 사용하는게 더 좋음. 더나아가 by lazy 를 사용하여
    사용 시 생성되도록 ..
  + 지연 초기화
    * 무거운 클래스를 만들 때 지연되게... 메소드의 호출을 빨리할 때
  + 기본 자료형 사용하기
    * nullable, 제너릭 사용 시 JVM에 의해서 wrap 한 자료형이 사용 됨

<br></br>
- 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라
  + 고차함수를 살펴보면 inline 키워드가 많이 붙어있다 ?
  ```kotlin
  inline fun repeat(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) action(index)
  }
  ```
  + 컴파일 시점에 '함수를 호출하는 부분' -> '함수의 본문'으로 대체
  + repeat(10) -> for (index in 0 until 10) println(index) 가 되는 것
  + 타입 아규먼트 reified 한정자를 붙여서 사용
  + 함수 타입 파라미터를 가진 함수가 훨씬 바르게 동작
  + 비지역(non-local) 리턴을 사용
  + 타입 아규먼트를 reified 사용할 수 있다
    * 자바 구버전에는 제네릭이 없음
    * 타입 파라미터를 사용한 부분이 타입 아규먼트로 대체. List<Int> 로 사용할 시 List<*> 는 가능한데 List<Int> 는 불가?
  + 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작
    * 함수 호출 & 리턴을 하는 점프&백스텝 과정이 빠지기에.. 그래서 표준 라이브러리에 간단한 함수둘은 대부분
    inline 한정자가 붙음 
    * Function(N)<Types> () 파라미터 갯수, 뒤에는 파라미터 타입, 리턴 타입 모두
  + 비지역적인 리턴(non-local return) 사용가능 
  + inline 한정자의 비용
    * 재귀적으로 동작할 수 없음.
    * private & internal 가시성을 가진 함수와 프로퍼티 사용 불가
  + crossinline & noinline
    * crossinline - 아규먼트로 인라인 함수를 받지만 비지역적 리턴을 하는 함수를 받을 수 없음
    * noinline - 아규먼트로 인라인 함수를 받을 수 없음

<br></br>
- 인라인 클래스의 사용을 고려하라
  + 타입만 맞는다면 곧바로 값을 집어넣을 수도 있음
  + 모든 메서드는 정적 메서드로 만들어짐
  + 측정 단위를 표현할 때
  + typealias 로 긴 타입을 바꾸어 사용

<br></br>
- 더 이상 사용하지 않는 레퍼런스를 제거하라
  + companion 으로 유지해 버리면 GC 가 메모리 해제를 할 수 없음. 특히나 액티비티는 큰 객체
  + 스택 구현시 요소가 1000 -> 1개가 될 때 나머지 공간을 제거해줘야 함, 객체 = null 로 표시해서..
  + 지역스코프로 사용하는게 좋음. 해당 스코프를 지나가면 GC에 의해서 제거될 수 있으므로
  + WeakReference 를 사용하여 대화상자 같은 경우 사용완료 후 제거될 수 있도록...가

### 3-2. 효율적인 컬렉션 처리
- 하나 이상의 처리 단계를 가진 경우에는 시퀀스를 사용하라
  + Iterable 과 Sequence 의 차이
    * 다른 목적으로 설계되어 있음, Sequence 지연(lazy) 처리, 시퀀스 처리
  함수들을 사용하면, 데코레이터 패턴으로 꾸며진 새로운 시퀀스가 리턴하고 최종에만 계산
    * 반면 Iterable 은 함수를 사용할 떄 마다 연산이 이루어져 List 만들어짐
    * 자연스로운 처리 순서 유지, 최소한의 연산,무한 시퀀스 형태, 각 단계에서 컬렉션을 만들어 내지 않음
    * 순서의 중요성
      - element-by-element order 또는 lazy order 로 요소 하나하나에 저정한 연산
      - filter -> map -> forEach 출력을 할 때 1,2,3을 다 처리하고 넘기는게 아니라 1에 대한 연산을
  다하고 2에 대한 연산을 하고 3에 대한 연산을 함
    * 최소 연산
      - filter -> map -> find 일 때 
      - 중간 처리 단계를 모든 요소에 적용할 필요가 없는 경우 시퀀스 사용
    * 무한 시퀀스
      - generateSequence or sequence 사용
      - take(10) 10개만 추출
    * 각각의 단게에서 컬렉션을 만들어 내지 않음
      - 콜렉션이 무거울 때 성능 및 메모리 낭비가 커짐 (ex readLine List<String>) outOfMemory 발생할 수 있음
      - readLines 보다는 useLines 사용이 필요
    * 시퀀스가 빠르지 않은 경우
      - sorted() 는 Sequence 를 List 로 변환한 뒤 자바의 sort 를 사용해 처리
      - 시퀀스에서 sorted() 를 빼는게 좋음
    * 자바 스트림의 경우
      - lazy 하게 동작하며 마지막 처리 단계에서 연산이 일어남
    * 코틀린 시퀀스 디버깅
      - 플러그인 Kotlin Sequence Debugger
- 컬렉션 처리 단계 수를 제한하라
  + filter -> map 보다는 filterNotNull() 이 더 좋음
  + 성능이 중요한 부분에는 기본 자료형 배열 사용
    * Int, IntArray 보다 List<Int> 가 5배 메모리 더 할당함
- mutable 컬렉션 사용을 고려하라
  + immutable 은 연산 시 복제가 발생하는데 비용이 많이 발생 함
  + 안정성은 immutable 콜렉션이 좋지만 성능적인 부분은 mutable 임
  + utils 또는 지역스코프 등 요소 삽입 자주 발생할 때?

- 용어
  + 함수
    * 톱레벨, 클래스 멤버 함수, 함수 내부의 지역 함수
  + 메서드
    * 클래스와 연결된 함수
    * 이를 호출하기 위해서는 인스턴스가 필요
  + 멤버
    * 클래스 내부에 정의된 요소
  + 확장
    * 이미 존재하는 클래스에 추가하는 가짜 멤버
  + 파라미터
    * 함수 선언에 정의되어 있는 변수
  + 아규먼트
    * 함수로 전달되는 실질적인 값