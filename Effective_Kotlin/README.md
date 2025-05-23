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

- #### Item 3 최대한 플랫폼 타입을 사용하지 마라
  + nullable 여부를 알 수 없는 타입 -> 어노테이션 활용(@NotNull, @Nullable)
  + NPE를 발생할 수 있는 요소 중 하나
  ```kotlin
  //java 에서는 null 이 올 수 있기에 annotation 을 사용해서 코틀린에서 nullable 확인 가능
  public class JavaTest {
    public String giveName() {
        return null;
    }
  }
  ```
  + !! 키워드를 사용해서 이 메소드가 전혀 null이 아니다라는 것을 사용가능하긴하다
  ```kotlin
  //List 및 User 의 nullable 체크가 필요하다
  public class UserRepo {
    public List<User> getUser() { ... }
  
    public @NotNull List<User> getUserNotNull() { ... }
  }
  
  val Users: List<User> = UserRepo.getUser()!!.filterNotNull()
  val Users2: List<User> = UserRepo.getUserNotNull()
  ```
  + 자바에서 넘어온 값들을 사용할 때 null 여부 체크를 무조건 해줘야 한다
  + 자바에서 넘어온 타입을 '플랫폼 타입'이라고 부른다
  + 안드로이드가 java -> kotlin 으로 넘어오면서 혼용할 때 이런 문제들이 자주 발생한다
    * 해결책으로 java 의 annotation 을 추가
    * java class -> kotlin class 로 변환
  * 아무런 의식없이 쓰게되면 nullable 값을 notNullable 로 받아들여서 디버깅에 많은 문제를 야기할 수 있음
  <br></br>
 
- #### Item 4 inferred 타입으로 리턴하지 마라
  + 추론타입은 피연산자에 맞게 설정 됨
  ```kotlin
  open class Animal 
  class Zebra : Animal()
  
  var animal = Zebra()
  animal = Animal() // type mismatch
  var animal2 : Animal = Zebra()
  animal2 = Animal() // OK
  ```
  + 인터페이스 Fiat16P 만 생산한다고 타입을 제외하는 순간 Car 가 아닌 Fiat126P 로 제한이 되어 문제가 발생한다(다형성)
  ```kotlin
  interface CarFactory {
    fun produce(): Car
  }
  
  val DEFAULT_CAR : Car = Fiat126P()
  val DEFAULT_CAR2 = Fiat126P()
  ```
  + API를 만들어서 외부에 공개할 떄 명확한 리턴 타입 명시가 필요
  <br></br>
  
- #### Item 5 예외를 활용해 코드에 제한을 걸어라
  + 제한 방법
    * require : argument 제한
    * check : 상태와 관련된 동작 제한
    * assert : 어떤 것이 true 인지 확인
    * return or throw 와 함께 활용하는 Elvis 연산자 활용
    ```kotlin
    fun pop(num: Int = 1): List<T> {
        require(num <= size) {
            "Cannot remove more elements than current size"
        }
        check(isOpen) { "Cannot pop from closed stack" }
        val ret = collection.take(num)
        collection = collection.drop(num)
        assert(ret.size == num)
        return ret    
    }
    ```
  + 아규먼트
    * require 키워드 사용 (조건 미 충족시 IllegalArgument Exception 발생)
    * ex) 팩토리얼 계산시 양의 정수만, 좌표를 받을 때 비어있지 않은 좌표, 이메일일때 규격
  + 상태
    * check 키워드 사용 (조건 미 충족시 IllegalState Exception 발생)
    * require 이후에 배치
    * ex) 어떤 객체 미리 초기화 후 진행, 로그인한 경우만 처리, 객체 사용할 있는 시점에만 처리
  + Assert 계열 함수
    * assertXXXX() 키워드 사용
    * 테스트에 주로 사용하고 함수가 올바르게 구현되었다면 참을 낼수 있는 코드
    * 예외를 throw 하지 않음
  + nullability & 스마트 캐스팅
    * requireNotNull, checkNotNull 메소드를 통해 변수를 'unpack' 도 가능. 
    * Null 체크를 하면 기 이후부터는 notNullable 이 됨
    * val email : String = person.email ?: return 의 형태로 많이 사용하기도 한다
    * val email : String = person.email ?: run { null 인 경우 하고 싶은 코드 } 의 형태로 많이 사용하기도 한다
<br></br>

- #### Item 6 사용자 정의 오류보다는 표준 오류를 사용
  + 재정의 한 오류보다는 표준 오류가 많이 알려져있기에 개발자가 이해하기 쉬움
  + CustomException 을 사용하는 경우 모르는 사람이 이해하기 어려움
  <br></br>
  
- #### Item 7 결과 부족이 생길떄 null & Failure 사용
  + 코틀린의 모든 예외는 unchecked 예외 (처리의 강제성이 없음)
  + try-catch 사용 시 명확하고 오류를 놓칠 확률이 줄어들지만 컴파일러가 할 수 있는 최적화가 줄어듬
  + null or Failure 사용
    ```kotlin
    inline fun <reified T> String.readObjectOrNull(): T? {
        if (incorrestSign) return null
        
        return result
    }
    
    val age = when UserText.readObjectOrNull<Person>?.age ?: -1
    
    inline fun <reified T> String.readObject(): Result<T> {
        if (incorrestSign) return Failure(JsonParsingException())
    
        return Success(result)    
    }
    
    sealed class Result<out T>
    class Success<out T>(val result: T): Result<T>()
    class Failure(val throwable: Throwable): Result<Nothing>()
  
    val person = userText.readObjectOrNull<Person>()
    val age = when (person) {
        is Success -> person.age
        is Failure -> -1  
    }
    ```
    * 안전 호출(safe call) or Elvis 연산자로 널 안정성 기능 활용 가능
    * Result 와 같은 공용체(union type)를 사용하여 when 처리도 가능
    * try-catch 보다 명확하며 애플리케이션 흐름도 중지시키지 않고 효율적임
  <br></br>

- #### Item 8 적절하게 null 을 사용하라
  + nullable 타입 3가지 처리 방법
    * ?. , 스마트 캐스이 , Elvis 연산자
    * 오류 throw
    * 함수 또는 프로퍼티를 리팩터링 해서 nullable 타입 제거
  + null 안전하게 처리
    ```kotlin
    printer?.print() //안전호출
    if (printer != null) printer.print() // 스마트캐스팅
    ```
    * Collection<T>?.orEmpty() 를 통해 빈 List<T> 를 받을 수 있음
    * 공격적 프로그래밍 : require, check, assert 미리 알려서 수정
    * 방어적 프로그래밍 : 발생할 수 있는 많은 것들로부터 안정성을 높이기위해 이것저것 다 하는 것
  + 오류 thorw 하기
    * 개발자가 어떤 코드를 보고 당연히 null이 아닐것으로 착각할 수 있다면 throw 를 날려 인지시키기
  + not-null assertion(!!)과 관련된 문제
    * !! 현 시점에 확실하다고 해도 미래의 변경될 수도 있기에 사용을 권하지 않음
    * lateinit(초기 호출이 명확할 때) 또는 Delegated.notNull 사용
    ```kotlin
    private lateinit var name: String
    private var id: Int by Delegates.notNull<Int>()
    ```
  + 의미없는 nullability 피하기
    * null -> !! 사용을 만들고 불필요한 의미없는 코드로 예외처리 하는 안좋은 영향이 발생 함 
    * 요소가 부족하다는 것을 나타낼때는 빈 컬렉션 사용 (List<Int>? 비추)
  + lateinit 프로퍼티와 notNull 델리게이트
    ```kotlin
    private lateinit var name: String
    private var id: Int by Delegates.notNull<Int>()
    ```
    * 초기화전에 값을 사용할 때 문제가 발생하지만 !!(unPack), nullable 로도 변경가능한 장점
    * JVM Int, Long, Double, Boolean 과 같은 기본타입과 연결된 경우 lateinit 사용 불가
    * 이럴 때 성능은 다소 떨어지지만 lateinit 대신 Delegates.notNull() 사용
    <br></br>

- #### Item 9 use를 사용하여 리소스를 닫아라
  + close 메소드 사용
    * InputStream/OutputStream, java.sql.Connection java.io.Reader, java.new.Socket & java.util.Scanner
    * close 한 리소스는 가비지 콜렉터가 처리하는데 레퍼런스가 없어질때까지 기다리기에 느리고 유지비용이 많이 듬
    * 반드시 닫아야 하는 리소스들 사용할 때 use(Closeable, AuthCloseable) 를 사용해라 
    ```kotlin
    BufferReader(FileReader(path)).use { reader -> 
      return reader.lineSequence().sumBy { it.lengh }
    }
    ```
  <br></br>

- #### Item 10 단위 테스트를 만들어라
  + 일반적인 UseCase(Happy Case), 오류케이스, 엣지케이스 ,
  + TDD 접근방식도 가능 
    * 테스트가 잘 되었기에 신뢰가고 리팩토링이 어렵지 않음
    * 레거시 코드를 수정하기가 겁남 (다른 개발자가 케이스를 파악하기 힘들므로)
    * 테스트를 만드는데 많이 시간도 걸림
    * 테스트를 유지하는데 아키텍처가 강제되는 경우도 발생
  + 복잡한 부분, 계속해서 수정이 발생해서 리팩토링이 필요한 부분, 비즈니스 로직, 공용 API, 문제가 자주 발생하는 부분, 운영 버그

### 1-2. 가독성
- #### Item 11 가독성을 목표로 삼아라
  + 인식 부하 감소
  ```kotlin 
    if (person != null && person.isAdult) {
      // code
    } else {
      // error
    }
  
    person?.takeIf { it.isAdult }
    ?.let {
        //code
    }
    ?.run {
        //error 엘비스연산자 활
    }
  ```
    * if-else 구문이 익숙하기에 눈에 잘 들어옴 (kotlin 의 경우 takeIf, elvis 연산자가 익숙하긴하나 복잡함)
    * 가독성은 뇌가 프로그램의 작동 방식을 이해하는 과정을 더 짧게 만드는 것
  + 극단적이 되지 않기
    * let, run 을 무조건 쓰지 말라는 것이 아님
    * 연산을 아규먼트 처리 후 이동시킬 때, 데코레이터를 사용해서 객체를 랩할 때 등 필요
  + 컨벤션
    * 연산자는 의미에 맞게 사용, 기존에 있는 기능을 새로 만들지 마라 등
  <br></br>
  
- #### Item 12 연산자 오버로드를 할 때는 의미에 맞게 사용
  + 함수에 맞는 이름을 사용(예외적으로 도메인 특화 언어는 가능)
  ```kotlin
  fun Int.factorial(): Int = (1..this).product()
  fun Iterable<Int>product(): Int = fold(1) { acc, i -> acc * i }
  
  print(10 * 6.factorial()) // 7200
  print(10 * 6!) // ? 
  ```
    * factorial 을 ! 표현하기는 하지만 코드에서는 not 을 의미함
    * +, -, ++, --, == 등 정해진 오버로딩 이외 사용 금지
  + 분명하지 않은 경우
  ```kotlin
    val tripleHello = 3 * { print("Hello") } // 의미가 분명하지 않음
    
    val tripleHello = 3 timesRepeated  { print("Hello") } 
    infix fun Int.timesRepeated(operation: ()-> Unit) = {
        repeat(this) { operation() } // infix 활용
    }
    repeat(3) { print("Hello") } // 톱레벨 함수를 사용    
  ```
  + 규칙을 무시해도 되는 경우
    * 도메인 특화 언어(DSL : Domain Specific Language) 설계 시
    ```kotlin
    body {
        div {
            +"Some Text" // String.unaryPlus 가 사용
        }   
    }
    ```
  <br></br>
  
- #### Item 13 Unit? 을 리턴하지 말라
  + if (!isSuccess(key)) return 와 success(key) ?: return 을 
  오해를 하기 쉽고 눈에 잘 들어오지 않는다.
  + boolean 을 사용하는 것이 좋음
  ```kotlin
  if (!ksyIsCorrect(key)) return
  verifyKey(key) ?: return // 키가 없는 건지 실패한건지 한눈에 이해하기 어려움
  ```
  <br></br>

- #### Item 14 변수 타입이 명확하지 않는 경우 확실하게 지정
  + 코틀린은 수준 높은 타입 추론 시스템을 가지고 있음
  ```kotlin
  val data = getSomeData() 
  val data: UserData = getSomeData() 
  ```
  + 타입을 명시하면 프로그래머가 개발자가 코드를 더 쉽게 파악할 수 있음
  + 상속 또는 구현을 받은 클래스의 default 로 사용하다가 원치않는 결과를 만들 수도 있음
  <br></br>
  
- #### Item 15 리시버를 명시적으로 참조하
  ```kotlin
  class Node(val name: String) {
    fun makeChild(childName: String) =
      create("$name.$childName").apply {
          print("Created ${this?.name} in ${this@Node.name}")
      } 
    }
  fun create(name: Stirng) : Node? = Node(name)
  ```
  + let, run, apply, also 등을 사용하면 return 으로 this, it 등의 확장 리시버를 받는다
  + 중첩으로 된 구조에서 명시적인 참조를 안하면 의도치 않게 다른 값을 참조하는 실수를 범할 수 있다.
  + DslMarker annotation 사용 (DSL 사용 )
  ```kotlin
  @DslMarker
  annotation class HtmlDsl
  
  fun table(f: TableDsl.() -> Unit) { /**/ }
  
  @HtmlDsl
  class TableDsl { /**/ }
  
  table {
    tr {
        td { +"Column 1" }
        td { +"Column 2" }
        this@table.tr {
            td { +"value 1"}
            td { +"value 2"}
        } 
    }
  }  
  ```
  + DSL 마커
    * 가장 가까운 리시버만을 사용하게 하거나
    * 명시적으로 외부 리시버를 사용하지 못하게 할 때 활용할 수 있는 중요한 메커니즘
    * tr 이 상위의 tr 인지 하위의 tr의 tr 인지 명확한 구분을 통해 오류 방지
  <br></br>
    
- #### Item 16 프로퍼티는 동작이 아니라 상태를 표시
  + 자바의 필드와 코틀린의 프로퍼티는 엄연히 다름
  ```kotlin
  String name = null
  
  var name:String? = null
    get() = field?.uppsercase
    set(value) {
        if (!value.isNullOrBlank()) field = value
    }
  ```
  + get(), set() 필드를 가짐
    * field 는 데이터를 저장해 두는 백킹 필드에 대한 레퍼런스
    * val 읽기전용 변수에서는 field 생성되지않고 var 에서 생성되고 이를 파생 프로퍼티(derived property) 라고 부름
  + 프로퍼티 위임도 가능
  ```kotlin
  val db: Database by lazy { connectToDb() }
  ```
  + 확장 프로퍼티 (안드로이드) - preferences, inflater 만으로 접근이 가능
  ```kotlin
  val Context.preferences:SharedPreferences
    get() = PreferenceManager.getDefaultShagedPreferences(this)
  val Context.inflater: LayoutInfalte
    get() = getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflator 
  ```
  + 반복적이나 계산 요소가 들어가는 것은 프로퍼티가 아니라 함수로 구현
    * 연산 비용 높거나 복잡도 O(1) 넘는 경구
    * 비즈니스 로직 포함 하는 경우
    * 매 연산이 같은 결과를 반환하지 않는 경우
    * 변환의 경우 (ex: Int.toDouble())
    * 상태 변경이 일어나는 경우
  <br></br>
    
- #### Item 17 이름있는 아규먼트 사용
  ```kotlin
  val text = (1..10).joinToString("|")
  val text = (1..10).joinToString(separator = "|")
  
  sleep(100)
  sleep(timeMillis = 100)
  ```
  + joinToString() 함수를 알면 상관없지만 모르는 경우 아래의 코드가 명확한 정보를 전달해줌
  + sleep() 또 마찬가지로 아규먼트 명시하면 시간을 헷갈리는 경우를 줄일 수 있음
  + 디폴트 아규먼트
    * 같은 타입의 파라미터 많은 경우 (이름있는 아규먼트 사용)
    * 함수 타입의 파라미터가 있는 경우 (이름있는 아규먼트 사용)
    ```kotlin
    button({/** 1 **/}, {/** 2 **/})
    button(onClick = {/** 1 **/}) {/** 2 **/}
    ```
    
    ```kotlin
    //자바 스타일 주석을 통해 정보 전달
    observable.getUsers()
    .subscribe((List<User> users) -> {
    // onNext
    }, (Throwable throwable) -> {
    // onError
    }, () -> {
    // onCompleted
    })
    
    // 코틀린 스타일 subscribe -> subscribeBy
    // (자바 함수를 호출할 때 이름있는 파라미터 사용할 수 없어서 별도 코틀린 함수 만들어야 함)
    observable.getUsers()
      .subscribe(
        onNext = { users: List<User> -> },
        onError = { throwable: Throawable ->  },
        onCompledted = { _ -> }  
      )
    ```
    <br></br>

- Item 18 코딩 컨센션 지키기
  + 지키는 이유
    * 어떤 프로젝트 접해도 쉽게 이해
    * 다른 외부 개발자도 프로젝트의 코드 쉽게 이해
    * 코드의 작동방식 쉽게 유추
    * 코드 병합 & 일부 코드를 다른 곳으로 이동시키기 편해짐
  + InteliJ 포매터 사용
  + 밑줄 그어지면 변경하고 자동정렬하면 됨
  ```kotlin
  class Student(val id: String, val name: String, val age: Int, val school: String) : Person(id, age ,name) {
  }
  
  class Student(
    val id: String,
    val name: String,
    val age: Int,
    val scholl: String
  ) : Person(id, name, age) {
  }
  ```
  + 각각의 파라미터를 다음줄에 작성을 하는 것을 따름(자바와 다름)     
  <br></br>
  
## 2. 코드설계
### 2-1. 재사용성
- Item 19. knowledge를 반복하여 사용하지 말라
  + "이미 있던 코드를 복사해서 붙여넣고 있다면 잘못된 것"
    * DRY(Don't Repeat Yourself)
    * WET 안티패턴
  + Knowledge
    * 의도적인 정보
    * 로직 : 프로그램이 어떠한 식으로 동작하는지 & 프로그램이 어떻게 보이는지
    * 공통 알고리즘 : 원하는 동작을 하기 위한 알고리즘
    * 둘의 차이는 시간에 따른 변화
  + 모든 것은 변화한다
    * 회사가 사용자의 요구 또는 습관을 파악
    * 디자인 표준이 변경
    * 플랫폼, 라이브러리, 도구 등 의 변화
    * ex) 슬랙의 전신은 글리치라는 온라임 게임이었음 -> 커뮤니케이션이 마음에 들어 게임만 사라지고 커뮤니티 기능만 남음
    * 공용 버튼을 여러군데서 사용할 때 디자인이 변경되는 경우 하나씩 고쳐야한다면?
    * 데이터베이스 접근을 한 군데서 하고 있는데 이름을 바꿨을떄 SQL 을 수정해야한다면?
    * 추상화를 통해 변화로부터 코드를 보호해야 함
  + 언제 코드를 반복해도 될까?
    * 함께 변견될 가능성이 높은가? 따로 변경될 가능성이 높은가?
  + 단일 책임 원칙
    * 클래스를 변경하는 이유는 단 한가지여야 한다
    * 두 액터가 동일한 클래스를 수정하면 안 됨
    * 서로 다른 곳에서 사용하는 knowledge는 독립적으로 변경될 가능이 높기에 서로 다른곳에 두는것이 좋음
    ```kotlin
    class Student {
        // isPassing 인증관련부서
        // qualifiesForScholarship 장학금관련부서 
        fun isPassing() : Boolean = calculatePointsFromPassedCourses() > 15
        fun qualifiesForScholarship(): Boolean = calculatePointsFromPassedCourses() > 30
    
        private fun calculatePointsFromPassedCourses() : Int {
            // ...
        }
    }
    ```
    * 덜 중요한 과목의 비중을 줄여라(calculatePointsFromPassedCourses) 모두 영향이 감
    * 즉, 서로 다른 곳에서 사용하는 knowledge 는 독립적으로 변경될 가능성이 높기에 분리해서 취급하는 것이 좋음
    <br></br>
  
- #### Item 20. 일반적인 알고리즘을 반복해서 구현하지 말라
  + 이미 있는 알고리즘은 구현하지 말고 그대로 써라
    ```kotlin
    val number = 20
  
    val percent = when { 
        number > 100 -> 100
        number < 0 -> 0
        else -> number
    } 
  
    val percent = number.coereIn(0, 100)
    ```
    * 작성 속도도 빨라지고 함수 이름만 보고 이해가 가능함
  
  + 표준 라이브러리 
    * forEach를 통해 값을 넣지 말고 map & apply 를 통해 프로퍼티 설정 
  ```kotlin
    val entries = item.sources.map(::sourceToEntry)
  
    fun sourceToEntry(source: Source) = SourceEntry()
        .apply {
            id = source.id
            // 코드
        }
  ```
  + 정리
    * 확장 함수는 구체적인 타입이 있는 개체에만 사용 제한이 좋음
    * 수정할 객체를 아규먼트로 전달받아 사용하는 것보다 확장 리시버로 사용하는 것이 가독성이 더 좋음
    * TextUtils.isEmpty(str) 보다는 str.isEmpty() 가 사용하기 더 쉬움
  
<br></br>
- #### Item 21. 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라
  + lazy or Delegate(observable, vetoable, notnull)
    * 다양한 패턴을 만들 수 잇음 (리소스 바인딩, 의존성 주입, 데이터 바인딩 등) -> 어노테이션 필요
    * 코틀린에서는 type-safe 하게 만들 수 있음
  ```kotlin
  var observed = false
  var max: Int by Delegates.observable(0) { property, oldValue, newValue ->
    println("Changing max to $newValue")
    observed = true
  }

  // Android
  private val button: Button by bindView(R.id.button)
  
  // 위임을 공통화
  var token: String? by LoggingProperty(null)
  var attemps: Int by LogginProperty(0)
  
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

- #### Item 22. 일반적인 알고리즘을 구현할 때 제네릭을 사용하라
  + 프로그램의 안정성이 높아짐. 개발이 편해짐
  + 대표적인 예로 stlib 에 filter 가 있음
  + 제네릭 제한
    * 구체적인 타입의 서브타입만 사용하게 타입을 제한
    * Any 타입을 활용하여 nullable이 아닌 타입 표시
    ```kotlin
    fun <T: Comparable<T>> Iterable<T>.sorted(): List { }
    
    fun <T: Animal> pet(animal: T) where T : GoodTempered { }
    
    inline fun <T, R: Any> Iterable<T>.mapNotNull(
        transform: (T) -> R?
    ) : List<R> {
        return mapNotNullTo(ArrayList<R>(), transform)
    }
    ```
  <br></br>

- #### Item 23. 타입 파라미터의 섀도잉을 피하라
  + 지역 변수가 외부 스코프의 프로퍼티를 가리는 경우 이를 섀도잉이라 함
  + 클래스 타입 파라미터에서도 발생함
  + 의도와 다르게 동작할 수 있음
  ```kotlin
  class Forest(val name: String) {
     fun addTree(name: String) { 
     }
  }

  interface Tree
  class Birch: Tree
  class Spruce: Tree
  
  class Forest<T: Tree> {
    fun addTree(tree: T) //독립적으로 동작
  }
  
  val forest = Forest<Birch>()
  forest.addTree(Birch())
  forest.addTree(Spruce()) // Error type mismatch
  
  // type 제한을 두어서 사용해야 함
  class Forest<T: Tree> {
    fun <ST: Tree> addTree(tree: ST)
  }
  ```
<br></br>

- #### Item 24. 제너릭 타입과 variance 한정자를 활용하라
  + variance out or in 키워드가 없는 경우 불공변성(invariant)라 함
  ```kotlin
  class Cup<T>
  Cup<Int>
  Cup<Number>
  Cup<Any>
  ```
    * 제너릭 타입으로 만들어지는 타입들이 전혀 관련이 없다는 뜻임
  + out 또는 in 으로 관련성을 주고자 할 때
    * out - 공변성(covariant) A가 B의 서브타입일때 Cup[A]가 Cup[B]의 서브타입
    ```kotlin
    class Cup<out T>
    open class Dog
    class Puppy: Dog()
       
    fun main() {
        val b: Cup<Dog> = Cup<Puppy>() // ok
        val a: Cup<Puppy> = Cup<Dog>() // 오류
    }
    ```
    * in - 반변성(contravariant) A가 B의 서브타입일때 Cup[A]가 Cup[B]의 슈퍼타입
    ```kotlin
    class Cup<in T>
    open class Dog
    class Puppy: Dog()
       
    fun main() {
        val b: Cup<Dog> = Cup<Puppy>() // 오류
        val a: Cup<Puppy> = Cup<Dog>() // OK
    }
    ```
  + 함수타입
    * (Int) -> Any 는 (Int) -> Number, (Number) -> Any 등으로 다양하게 작동
    * 파라미터 타입은 상위 계층 타입으로 이동이 가능하고 리턴 타입은 하위 계층 타입으로 이동 가능
    ```kotlin
    val intToDouble: (Int) -> Number = { it.toDouble() }
    val numberAsText: (Number) -> Any = { it.toShort() }
    val identity: (Number) -> Number = { it }
    val numberToInt: (Number) -> Int = { it.toInt() }
    val numberHash: (Any) -> Number = { it.hashCode() }
    ```
    * 코틀린 함수 타입의 모든 파라미터 타입은 contrvariant 
    * 코틀린 함수 타입의 모든 리턴 타입은 covariant
    * List 에는 out 한정자가 붙어있지만 MutalbleList 에는 붙지 않음
    ```kotlin
    (T1 in, T2 in) -> T out 
    ```
  + variance 한정자의 안정성능
    * 자바의 배열은 covariant 코틀린은 invariant 묵시적으로 업캐스팅을 한다.
    ```kotlin
    Integer [] numbers = {1, 4, 2, 1}
    Object[] objects = numbers
    objects[2] = "B" //자바에서 에러 아직 Integer임
    // 코틀린에서는 Array<Int> -> Array<Any> 변환이 가능 invariant
    ```
    * 파라미터 타입을 예상이 가능하다?
    * Response<T> 라면 T의 모든 서브타입 허용 ex) Response<Any> 라면 Response<Int>, Response<String> 가능
    * Response<T1, T2> 라면 T1과 T2 의 모든 서브타입 허용
    * Failure<T> 라면 T의 모든 서브타입 Failure 허용 
    ```kotlin
    sealed class Response<out R, out E>
    class Failure<out E>(val error: E) : Response<Nothing, E>()
    class Success<out R>(val value: R) : Response<R, Nohting>()
    ```
    * contravariant 타입 파라미터(in 한정자)를 public out 한정자 위치에 사용하는 것을 금지하고있다
  + variance 한정자 위치
    * 선언 부분과 클래스/인터페이스를 활용한 위치
    ```kotlin
    class Box<out T>(val value: T) //선언
    val boxStr: Box<String> = Box("str")
    val boxAny: Box<out Any> = boxStr
    ```
    * MutableList 는 in 한정자를 포함하면 요소를 리턴활 수 없으므로 in 한정자 안 붙음
    * 단일타입에 in 한정자를 붙여 contravariant 를 가지게하여 여러 가지 타입을 받아들이게 할 수 있음
  + 정리
    * 리턴만 되는 타입에는 covariant(out 한정자) 사용
    * 허용만 되는 타입에는 contracovariant(in 한정자) 사용
    <br></br>
    
- #### Item 25. 공통 모듈을 추출해서 여러 플랫폼에서 사용하라
  + 코틀린 백엔드 플랫폼 ktor
  + 코틀린J/S or 서버 코틀린으로 공통 코드 생산이 가능
  + 코틀린JS(브라우저), 코틀린 네이티브(iOS)
<br></br>
  
### 2-2. 추상화 설계
- 개요
  + 함수형 프로그래밍 커뮤니티에서는 추상화&컴포지션으로 프로그래밍을 구성한다?
  + 추상활 설계? 함수를 정의할 때 그 구현을 함수 시그니처 뒤에 숨기는 것
  + 자동차는 엔진, 서스펜션, 알터네이터 등 다양한 요소가 있지만 운전자는 핸들만 조작하면 된다.
  -> 추상화가 잘 되어있음
    * 이는 복잡성을 숨기고
    * 코드를 체계화등
    * 만드는 사람에게 변화의 자유를 준다.
<br></br>
    
- #### Item 26. 함수 내부의 추상화 레벨을 통일하라
  + 추상화가 잘 되어있는 것은 그 하위의 단계는 신경 안써도 되고 내것만 잘 처리하면 됨
    * 애플리케이션 > 프로그래밍 언어 > 어셈블리 > 하드웨어 > 물리장치
  + 추상화 레벨 통일
    * SLA 원칙(single level of abstraction)
    ```kotlin
    fun makeCoffee() { 
        /** 모든 로직 처리 **/   
    }
    
    fun makeCoffee() {
        // 함수를 나누어 사용 (낮은레벨을 다른곳에서 커피 > 물끓이기,커피브루잉,커피붓기,우유붓기 등)
        boilWater()
        brewCoffee()
        pourCoffe()
        pourMilk()
    }
    
    fun makeEspressoCoffee() {
        boilWater()
        brewCoffee()
        pourCoffe()
    }
    ```
    * 추상화를 해놓으면 에스프레소는 우유붓기만 빼고 조합을 하면 되니 간편히 개발 가능하고 테스트도 쉬워짐
<br></br>
    
- #### Item 27. 변화로부터 코드를 보호하려면 추상화를 사용하라
  + 상수 
    * const val min = 7 보다는 const val MIN_PASSWORD_LENGTH가 좋음
    ```kotlin
    const val MIN_PASSWORD_LENGTH = 7
   
    fun isPasswordValid(text: String) {
       if (text.length < 7) return
       if (text.length < MIN_PASSWORD_LENGTH) return 
    }
    ```
    * 상수를 사용하면 일일히 수정하지 않고 한번에 모든것을 변경이 가능해짐
  + 함수 
    * 안드로이드에서 toast, snackbar, showMessage 같은 것을 확장함수로 만들면 편히 사용 가능
  + 클래스
    * 클래스는 함수와 달리 상태를 가지기에 더 강력하다
    * 기본적으로 final이고 open은 통해 서브클래스 제공으로 자유도 조금 높아짐. 추상화를 통해 더 자유로움
  을 제공할 수 있음
  + 인터페이스
    * 인터페이스 뒤에 객체를 숨겨 실질적인 구현을 추상화하고 사용자가 추상화된 것에만 의존하게 만든다
    * listOf 팩토리 메소드로 List 를 리턴함
    * 컬렉션 처리 함수는 Iterable 또는 Collection 의 확장함수로 List, Map 등을 반환
    * 프로퍼티 위임은 모두 인터페이스로
    이러면 결함을 줄일 수 있음
    * listOf or Iterable or Collection 은 인터페이스를 리턴하는데 각 플랫폼마다 다른 리스틀 리턴
    최적화 때문에..? 하지만 모두 인터페이스에 맞춰져 있으므로 차이없게 사용이 가능하다?
  + ID 만들기
    * id 값을 숨기고 get을 통해 제공
  + 균형이 필요
    * 팀의 크기, 경험, 프로젝트 규모, 도메인 지식
<br></br>
    
- #### Item 28. API 안정성을 확인하라
  + 표준 API를 사용해야 함
    * 변경 시 수동 업데이트를 해야 함
    * 개발자는 새로운 API를 배워야 함 - MAJOR.MINOR.PATCH 순으로..
    * @Experimental 또는 @Deprecated 또는 @ReplaceWith 등을 통해 고지가 필요
<br></br>

- #### Item 29. 외부 API 랩(wrap) 해서 사용
  + 외부 API가 불안정한 경우 랩해서 사용
    * 문제가 생기는 경우 wrapper 만 변경하기에 대응이 쉬움
    * 프로젝트 스타일에 맞춰서 조정 가능
    * 라이브러리 교체도 가능해짐
    * 그러나 개발자들이 따로 확인해야하며 문제가 생기는 경우 직접 해결해야함
<br></br>

- #### Item 30. 요소의 가시성을 최소화하라
  + 작은 인퍼테이스가 배우기 쉽고 유지도 쉽다.
  + 클래스의 프로퍼티를 외부에 노출시키면 상태를 보장할 수 없으므로 불변성이 무너짐
  + 가시성 한정자
    * 클래스 - public(디폴트), private, protected, internal(모듈 내부)
    * 톱레벨 - public(디폴트), private, internal
    * 모듈은 그레이들, 메이븐, 모듈 단위로 패키지와 다름
  + API를 상송할 때 오버라이드해서 가시성을 제한할 수는 없다
    * 서브클래스가 슈퍼클래스로도 사용될 수 있기 떄문이다
    * 상속보다 컴포지션을 선호하는 대표적인 이유
    <br></br>
    
- #### Item 31. 문서로 규약을 정의하라
  + 확장함수로 토스트 메시지를 만들었다면 다른 사용자는 모를 수도 있음 
    * 그렇기에, KDoc 주석을 붙여주는 것이 좋음.
    * 추상화 함수에 의존하게 만들지 않도록 문서로 설명하자
  + 규약 정의하기
    * 이름, 주석과 문서, 타입 
  + 주석을 써야할까?
    * sum, product, minus 등 누구나 아는 요소의 주석은 노이즈일 뿐이다
  + KDoc 형식
    * KDoc 은 /** */
    * 첫 번째 부분 요약 설명, 두 번째 부분 상세 설명, 세 번째 부분 태그로 시작
    * 태그 - param, return ,constructor ,receiver, property, throws, sample, see, author, since, supress
  + 타입 시스템과 예측
    * 클래스가 어떤 동작을 할 것이라 예측되면 그 서브클래스도 이를 보장
    * 리스코프 치환 원칙
<br></br>
    
- #### Item 32. 추상화 규약을 지켜라
  + 규약은 개발자들의 단순한 합의
  + hashCode() 가 구현되지 않으면 HashSet 과 함께 사용할 때 제대로 동작하지 않음
  + equals() 구현하지 않아 중복을 허용할 수도 있음
<br></br>
  
## 5장 객체 생성

- #### Item 33. 생성자 대신 팩토리 함수를 사용하라
  + 생성자 말고도 함수, 패턴 등으로 객체 생성이 가능함
  + 생성자 역할을 대신 해 주는 함수가 팩토리 함수
    * 함수의 이름을 붙여 방법과 아규먼트가 무엇이 필요한지 명확히 전달
      * ArrayList(3) 보다는 ArrayList.withSize(3) 이해하기 더 쉬워보인다.
    * 함수가 원하는 타입을 리턴 가능 : 인터페이스 뒤에 실제 객체의 구현을 숨길 때 유용 listOf
      * 코틀린 자바/JS/Native 에 따라 맞는 컬렉션 형태로 제공함
    * 매번 생성하는 것이 아니라 이미 생성되어있으면 반환 (싱글턴)
    * inline 형태로 구현 가능하며 복잡한 형태로 구현도 가능
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
  + companion 객체 팩토리 함수
    * 가장 쉬운 팩토리 함수 정의하는 일반적인 방법
    ```kotlin
    class Test {
        companion object {
            fun <T> of(vararg elements: T): MyLinkedList<T>? {
                /**/
            }   
        }    
    }
    ```
    * 자바에서 static method 와 동일
    ```kotlin
    val date: Date = Date.from(instance)
    val faceCards: Set<Rank> = EnumSet.of(JACK, QUEEN, KING)
    val prime: BigInteger = BigInteger.valueOf(Integer.MAX_VALUE)
    val luke: StackWalker = StackWalker.getInstacne(options)
    val newArray = Array.newInstance(classObject, arrayLen)
    val fs: FileStore = Files.getFileStore(path)
    val br: BufferedReader = Files.newBufferedReader(path)
    ```
  + 확장 팩토리 함수
    * 재정의
    ```kotlin
    interface Tool { companion object {} }
    
    Tool.Companion.createBigTool(): BigTool { }
    ```
  + 톱레벨 팩토리 함수
    * listOf, setOf, mapOf
  + 가짜 생성자
    * 인터페이스를 위한 생성자를 만들고 싶을 때
    * reified 아비 아규먼트를 갖게 하고 싶을 때
    * invoke 함수를 갖는... 톱레벨 함수를 사용
  + 팩토리 클래스의 메서드
  <br></br>
  
- #### Item 34. 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라
  + 점층적 생성자 패턴
    * 여러 가지 종류의 생성자를 사용하는 간단한 패턴
    * 코틀린에서는 디폴트 아규먼트를 설정해서 인자가 들어오지 않는 경우 세팅 함
    * 생성자시 그냥 입력보다는 parameter="" 로 하면 좋음
  + 빌더 패턴
    * 자바에서는 이렇게 사용하지만... 그냥 길다.. 명확하지도 않고.. 사용하기도 어렵다, 
    thread-safe 구현도 어려움
    * .setXX() 를 이어서 붙이게끔 하는 방식인데 코틀린에서 굳이 쓸 필요는 없다
    * AlertDialog 를 보면 값의 의미를 묶어서 전달이 가능하기에 필요하긴 함
    * 코틀린에서는 DSL 빌더를 활용하고 있음
    ```kotlin
    context.alert(R.string.fire_msg) {
        positiveButton(R.string.fire) {
    
        }
    
        negativeButton {
    
        }
    }
    ```
    * 사용하는 경우
      * 다른 언어로 작성된 라이브러리 옮길때
      * DSL 지원하지 않는 다른 언어에서 쉽게 사용하는 API 만들 때
    <br></br>
      
- #### Item 35. 복잡한 객체를 생성하기 위한 DSL
  + DSL(Domain Specific Language)
    ```kotlin
    body {
        div {
            a("https://kotlinlang.org") {
                target = ATarget.blank
                +"Main site"
            }   
        }   
        +"Some content"
    }
    ```
    * 복잡하고 계층적인 자료 구조를 쉽게 만들 수 있음
    * type-safe 하며 그루비와 다르게 여러 유용한 힌트 활용 가능
  + 사용자 정의 DSL 만들기
    * () -> Unit 아규먼트를 갖지 않고 & 리턴 Unit
    * (Int) -> Unit 아큐먼트 Int & 리턴 Unit
    * (Int) -> () -> Unit Int 아규먼트 & 리턴 함수
    * (()->Unit) -> Unit 아규먼트 함수 & 리턴 Unit
    * 람다 표현식, 익명함수, 함수 레퍼런스
    * 리시버를 가진 람다 표현식으로 정의하면 this 키워드가 이를 참조
      ```kotlin
      val myPlus: Int.(Int) -> Int = {this + it}
      
      myPlus.invoke(1,2)
      myPlus(1,2)
      1.myPlus(2)
      ```
      + 일반적인 객체처럼 invoke 메서드 사용
      + 확장 함수가 아닌 함수처럼 사용
    * DSL Builder 를 만들고 파라미터를 활용해서 값들은 처리
      ```kotlin
      fun createTable(): TableDsl = table {
        tr {
            for (i in 1..2) {
                td {
                    +"This is column $i"
                }     
            }
        }
      }
      ```
  + 언제 사용할까?
    * 복잡한 자료 구조
    * 계층적인 구조
    * 거대한 양의 데이터
    <br></br>


### 2-4. 클래스 설계
- #### Item 36. 상속보다는 컴포지션을 사용하라
  + 간단한 행위 재사용
    * 로딩 프로그레스바를 추상클래스로 만들고 사용처에서 상속받아 쓸 때 문제점
      + 상속은 클래스의 모든 것을 가져오기에.. 불필요한 것도 가져옴. 이는 객체지향설계 중 인터페이스 분리 원칙인 ISP를 위반
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
    ```kotlin
    abstact class Dog {
        open fun bark()
        open fun sniff() 
    }
    ```
    * 로봇 강아지의 경우 sniff 를 할 수 없고 bark 만 가능할 때 사용하면 좋음
  + 캡슐화를 깨는 상속
    * 위임패턴은 클래스가 인터페이스를 상속받게 하고 포함한 객체의 메소드를 활용해서 인터페이스에서 정의한 메서드를 구현하는 패턴
    * 포워딩 메소드 - 인터페이스에서 정의한 메서드를 구현하는 패턴
      + 컴파일 시점에 자동으로 만들어지기도 한다?
  + 오버라이딩 제한하기
    * final 키워드를 사용하면 상속 못함
    * open 클래스는 open 메서드만 오버라이드 할 수 있음
  + 결론
    * 명확한 IS-A 인 경우는 상속이 맞음
    * 상속이 안되도록 설계되지 않은 메소드는 final 키워드 사용하고 반대의 경우 open 키워드
    <br></br>
    

- #### Item 37. 데이터 집합 표현에 data 한정자를 사용하라
  + data 한정자
    * toString(), equals & hashCode, copy, componentN(component1, component2) 자동생성
    * copy() 의 경우 Immutable 데이터 클래스 만들 떄 편리 (shallow-copy : 얕은 복사 사용)
    * componentN()
      + val (first, second, third) = list 이런식으로 사용
  + 튜플 대신 데이터 클래스 사용하기
    * 튜플은 Serializable 기반으로 만들어지고 toString을 사용할 수 있는 제네릭 데이터 클래스
    * Pair, Triple 이 있는데 코틀린에 남아있는 마지막 튜플
    * 집합용도로 표현시는 좋음
    ```kotlin
    val (description, color) = when {
        degrees < 5 -> "color" to Color.BLUE
        degrees < 23 -> "mild" to Color.YELLOW
        else -> "hot" to Color.RED
    }
    
    val (odd, even) = numbers.filter { it % 2 == 1 }
    val map = mapOf(1 to "San Francisco", 2 to "LA")
    ```
    <br></br>


- #### Item 38. 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용
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
    * 코틀린이 아닌 다른 언어에서 사용할 클래스를 설계할 때
  ```kotlin
  class CalendarView {
    // java 스타일
    var listener: Listener? = null
    interface Listener {
        fun onDateClicked(date: Date)
        fun onPageChanged(date: Date)
    }
  
    // kotlin
    var onDateClicked: ((date: Date) -> Unit)? = null
    var onPageChanged: ((date: Date) -> Unit)? = null
  }
  ```
  <br></br>


- #### Item 39. 태그 클래스보다는 클래스 계층 사용
  ```kotlin
  fun match(value: T?) = when (matcher) {
  
  }
  
  enum class Matcher {
    EQUAL, NOT_EQUAL, LIST_EMPTY, LIST_NOT_EMPTY
  }
  ```
  + sealed 클래스를 사용하여 서브클래스 정의를 제한 함
    * abstract 사용하여 구현
  + sealed 한정자
    * 외부에서 추가적인 서브 클래스를 만들 수 없으므로 else 구문이 필요없음
  + 태그 클래스와 상태 패턴의 차이
  <br></br>


- #### Item 40. equals 규악을 지켜라
  + Any 에는 equals, hashCode, toString 메서드들이 잘 설정된 규약을 가지고 있음
  + 동등성
    * 구조적 동등성 ==, != 값의 비교
    * 레퍼런스적 동등성 ===, !== 객체가 가리키는 주소
  + equals 가 필요한 이유
    * Any 클래스에 구현되어 있는 equals 메서드느 디폴르톨 ==== 같은 객체인지 비교함
    * 직접구현
      + data 한정자가 없거나 비교해야 할 프로퍼티가 클래스에 없는 경우
      + 일부 프로퍼티만 비교해야 하는 경우
      + 기본적으로 제공하는 동작과 다른 동작을 하는 경우
  + equals 의 규약
    * 반사적 x.equals(x) = true
    * 대칭정 x.equals(y) 와 y.equals(x) 는 값은 결과 값
    * 연속적 x.equals(y) true 이고 y.equals(z) true -> x.equals(z) true 
    * 일관적 x.equals(y) 는 여러번 하더라도 같은 값 리턴
    * null 관련된 동작 x가 널이 아니라면 x.equals(null) 은 항상 false
    * 내부적으로 빠르게 동작해야 함
  + URL equals
    * https://en.wiki 와 https://wiki 는 사실상 같은 주소인데 인터넷의 연결여부에 따라 달라짐 ->
    설계가 잘못되어 있음.
  + equals 구현하기
    * 특별한 이유가 없는 이상 직접 equals 를 구현하는 것은 좋지 않습니다
    <br></br>
    

- #### Item 41. hashCode 규악을 지켜라
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


- #### Item 42. compareTo 규악을 지켜라
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
    * 톱 레벨 함수로 compareValues() 단순하게 값 비교시, compareValuesBy() 더 많은 값을 비교하거나 선택기 활용 시
  <br></br>
  

- #### Item 43. API의 필수적이지 않는 부분을 확장 함수로 추출하라
  + 함수 내에 멤버 메소드를 -> 외부로 꺼내어 사용
    * 확장 함수는 다른 패키지에 있을 확률이 높음 (데이터 & 행위를 분리하도록 설계한 프로젝트)
    * 같은 이름이 있는 메소드가 있다면 위험할 수 있음
    * 가상이 아님(컴파일 시점에 정적으로 선택 됨), 상속을 목적으로 설계된 클래스라면 학장 함수 노노
    <br></br>
    

- #### Item 44. 멤버 확장 함수의 사용을 피하라
  + 어떤 클래스에 대한 확장 함수를 정의할 때 멤버로 추가하는 것은 좋지 않음
    * 레퍼런스를 지원하지 않음 ex)String::isPhoneNumber
    * 암묵접 접근을 할 때, 두 리시버 중에 혼동함
  * 확장 함수의 가시성을 사용하는 것이 좋다
  <br></br>
  

### 3-1. 비용 줄이기
- #### Item 45. 불필요한 객체 생성을 피하라
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
    * 함수 내에서 정규식을 매번 만드는 것보다 톱 레벨로 빼서 사용하는게 더 좋음. 
      + 더나아가 by lazy 를 사용하여사용 시 생성되도록 ..
      + 톱 레벨 선언 시 한번만 생성하지만 함수일 때는 매번 생성하는 문제가 있음
  + 지연 초기화
    * 무거운 클래스를 만들 때 지연되게... 메소드의 호출을 빨리할 때
  + 기본 자료형 사용하기
    * nullable, 제너릭 사용 시 JVM에 의해서 wrap 한 자료형이 사용 됨
    * 되도록 기본 자료형을 쓰는게 좋음
    <br></br>
    

- #### Item 46. 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라
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
    ```kotlin
    inlinfe fun <reified T> printTypeName() {
        print(T::class.simpleName)
    }
    
    // 사용 
    printTypeName<Int>() // Int
    printType mName<String>() // String
    ```
  + 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작
    * 함수 호출 & 리턴을 하는 점프&백스텝 과정이 빠지기에.. 그래서 표준 라이브러리에 간단한 함수둘은 대부분
    inline 한정자가 붙음 
    * Function(N)<Types> () 파라미터 갯수, 뒤에는 파라미터 타입, 리턴 타입 모두
    * filter, sumBy 의 메소드들이 inline 으로 되어있기에 빠름
  + 비지역적인 리턴(non-local return) 사용가능 
  + inline 한정자의 비용
    * 재귀적으로 동작할 수 없음.
    * private & internal 가시성을 가진 함수와 프로퍼티 사용 불가하기에 클래스에서 사용되지 않음
  + crossinline & noinline
    * crossinline - 아규먼트로 인라인 함수를 받지만 비지역적 리턴을 하는 함수를 받을 수 없음
    * noinline - 아규먼트로 인라인 함수를 받을 수 없음
    <br></br>
  

- #### Item47. 인라인 클래스의 사용을 고려하라
  + 타입만 맞는다면 곧바로 값을 집어넣을 수도 있음
  + inline class 모든 메서드는 정적 메서드로 만들어짐
    ```kotlin
    inline class Name(private val value: String) {
    
        fun greet() {
            print("Helo, I am $value")
        }
    }
    val name = Name("JS")
    name.greet()
    
    // compile 시
    val name: String = "JS"
    Name.'greet-impl'(name)
    ```
    * 측정 단위 표시 or 타입 오용으로 발생하는 문제 막을 때 사용
  + 측정 단위를 표현할 때
  + 타입 오용으로 발생하는 문제를 막을 때
    * DB 에서 ID 가 Int 형일 때 클래스를 사용해 wrapping 해서 전달
    * 오버헤드도 적은 편
  + 인라인 클래스와 인터페이스
  + typealias 로 긴 타입을 바꾸어 사용
    * 단위 등을 표시할 때는 안전하지 않음
  <br></br>
  

- #### Item 48. 더 이상 사용하지 않는 레퍼런스를 제거하라
  + companion 으로 유지해 버리면 GC 가 메모리 해제를 할 수 없음. 특히나 액티비티는 큰 객체
  + 스택 구현시 요소가 1000 -> 1개가 될 때 나머지 공간을 제거해줘야 함, 객체 = null 로 표시해서..
  + soft reference
    * 메모리가 필요한 경우 GC 에서 알아서 해제함
  + WeakReference
    * 대화상자 같은 경우 사용완료 후 제거될 수 있도록...가
  + 수동으로 객체를 해제하는 것이 어려움
    * 지역스코프를 활용하는 것을 고려
    * 톱레벨 프로퍼티 나 객체 선언으로 큰 데이터 저장하지 않는 것이 좋음
  <br></br>
    

### 3-2. 효율적인 컬렉션 처리
- #### Item 49. 하나 이상의 처리 단계를 가진 경우에는 시퀀스를 사용하라
  + Iterable 과 Sequence 의 차이
    * 다른 목적으로 설계되어 있음, Sequence 지연(lazy) 처리, 시퀀스 처리
    * 함수들을 사용하면, 데코레이터 패턴으로 꾸며진 새로운 시퀀스가 리턴하고 최종에만 계산
    * 반면 Iterable 은 함수를 사용할 떄 마다 연산이 이루어져 List 만들어짐
    * 자연스로운 처리 순서 유지, 최소한의 연산,무한 시퀀스 형태, 각 단계에서 컬렉션을 만들어 내지 않음
  + 순서의 중요성
    * element-by-element order 또는 lazy order 로 요소 하나하나에 저정한 연산
    * filter -> map -> forEach 출력을 할 때 1,2,3을 다 처리하고 넘기는게 아니라 1에 대한 연산을
  다하고 2에 대한 연산을 하고 3에 대한 연산을 함
  + 최소 연산
    * filter -> map -> find 일 때 
    * 중간 처리 단계를 모든 요소에 적용할 필요가 없는 경우 시퀀스 사용
  + 무한 시퀀스
    * generateSequence or sequence 사용
    * take(10) 10개만 추출
  + 각각의 단게에서 컬렉션을 만들어 내지 않음
    * 콜렉션이 무거울 때 성능 및 메모리 낭비가 커짐 (ex readLine List<String>) outOfMemory 발생할 수 있음
    * readLines 보다는 useLines 사용이 필요
  + 시퀀스가 빠르지 않은 경우
    * sorted() 는 Sequence 를 List 로 변환한 뒤 자바의 sort 를 사용해 처리
    * 시퀀스에서 sorted() 를 빼는게 좋음
  + 자바 스트림의 경우
    * lazy 하게 동작하며 마지막 처리 단계에서 연산이 일어남
  + 코틀린 시퀀스 디버깅
    * 플러그인 Kotlin Sequence Debugger
  <br></br>


- #### Item 50. 컬렉션 처리 단계 수를 제한하라
  + filter -> map 보다는 filterNotNull() 이 더 좋음
  ```kotlin
  list.map {it.name}
        .filter {it != null}
        .map { it!! }
  
  list.map {it.name}
        .filterNotNull()
  
  list.mapNotNull {it.name}
  ```
    * 각 단계마다 컬렉션 만들기에 성능이 떨어짐
  <br></br>
  
  
- #### Item 51. 성능이 중요한 부분에는 기본 자료형 배열 사용
  * Int, IntArray 보다 List<Int> 가 5배 메모리 더 할당함
    * 가볍고 값 접근 시 추가 비용이 들어가지 않기에 빠름
  <br></br>
  

- #### Item 52. mutable 컬렉션 사용을 고려하라
  + immutable 은 연산 시 복제가 발생하는데 비용이 많이 발생 함
  + 안정성은 immutable 콜렉션이 좋지만 성능적인 부분은 mutable 임
  + utils 또는 지역스코프 등 요소 삽입 자주 발생할 때?
  <br></br>