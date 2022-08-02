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
  + out - 공변성(covariant) A가 B의 서브타입일때 Cup<A>가 Cup<B>의 서브타입
  + in - 반변성(contravariant) A가 B의 서브타입일때 Cup<A>가 Cup<B>의 슈퍼타입
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
  
### 2-2. 추상화 설계
### 2-3. 객체 생성
### 2-4. 클래스 설계

<br></br>
## 3. 효율성
### 3-1. 비용 줄이기
### 3-2. 효율적인 컬렉션 처리