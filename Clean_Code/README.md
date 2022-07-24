# Summary-Clean-Code

## 1.깨끗한코드
- 나쁜코드의 대가
  + 나쁜코드가 쌓일수록 코드의 생산성은 낮아진다.
    * 재설계를 위한 팀을 따로 두워서 진행을 하지만 쉽지 않다.
  + 관리자는 일정을 개발자는 좋은 코드를 만들어야 하는 현실적인 문제가 존재
- 깨끗한 코드
  + 한 가지의 목적만을 가지도록 소프트웨어 설계 원칙
  + 의도를 숨기지 않는 코드
  + 테스트케이스가 없으면 깨끗하지 않다는 말을 살짝 동의하기 어렵다. 
- 보이스카우트 원칙
  + 처음 왔을 때 보다 더 깨끗하게 해놓고 가라 -> 유지보수가 진행되어도 더러운 코드를
  코드로 만들지 마라

## 2.의미있는 이름
- 의도를 밝혀라
  + 의미를 전달할 수 있게 변수/함수 명 만들기
    - 리스트에 0번쨰를 4와 비교하는 코드를 의미를 전달할 수 있는 코드로..
  ```kotlin
  val d = 4
  val daySinceCreation = 4   
   
  fun getThem(): List<Int> {
    // 코드
    if (list[0] == 4) list.add(x)  
    return list
  }
  
  fun getFlaggedCells(): List<Int> {
    // 코드
    if (cell[status_value] == FLAGGED) 
        flaggedCellslist.add(cell)
    return flaggedCellslist
  }
  ```
  + 그릇된 정보를 피하라
    - o, 0, O & 1, l, i, I 구별이 힘듬
  + 의미 있게 구분
    - NameString vs String, Customer vs CustomerObject
    - getActiveAccount(), getActiveAccounts(), getActiveAccountInfo()
    - 매개변수에 의미없는 인자명보다는 사용처를 명확하게 쓰기
  + 발음하기 쉬운 이름 사용
    - 서버기반의 용어들은 축약어로 사용하고 이를 네트워크 통신으로 받아서 쓰면 곤란한 경우가 많다
    - DtaRcrd102 로 내려오지만 그대로 쓰면 발음도 어렵고 이해도 어렵다 DataRecord로 변경이 필요하다
  + 검색하기 쉬운 이름 사용
    - val PIE = 3.14 가 val p = 3.14 보다는 찾기가 편함
  + 인코딩 피하라
    - 옛날방식 mDest, mObject 
    - 인터페이스 ShapeFactoryImp IShapeFactory
  + 자신의 기억력을 자랑하지 마라
  + 클래스 이름 - 명사/명사구
  + 메소드 이름 - 동사/동사구
  + 기발한 이름은 피하라
  + 한 개념에 한 단어를 사용
    - 네트워크 통신 get, call, request
    - controller, manager, driver
  + 말장난을 하지 마라
    - 한 단어를 두 가지 목적으로 사용하면 안 됨
  + 맥락을 분명하게 하기
    - if-else 문 안에 다른 로직이 들어간다면 각각을 의미있는 함수로 만들고 호출하기
      
  

<br></br>