# 코틀린 스터디

> Oreilly / 코틀린 쿡북
>
> 2022.01.13 ~ 

## 2. 코틀린 Basic

### 2.1 코틀린에서 널 허용 타입 사용하기

- 변수가 절대 null 값을 갖지 못하게 하고 싶다면?

  - 물음표를 사용하지 않고 변수의 타입을 정의한다. 또한 nullable 타입은 안전 호출 연산자(?.)나 엘비스 연산자(?:)와 결합해서 사용한다.

- 물음표로 정의되지 않은 변수에 null을 할당하면 컴파일되지 않는다.

- null을 허용하려면 String? 와 같이 선언해야 한다.

- 널 아님 단언 연산자(!!)가 있지만, 이는 코틀린에서 NPE를 만날 수 있는 몇가지 상황중 하나이므로 가능하면 사용하지 말자.

  ```kotlin
  var p = Person(first = "North", moddle = null, last = "West")
  if (p.middle != null) {
    val middleNameLength = p.middle!!.length
  }
  ```

  - 이런 상황에서는 안전 호출 연산자(?.)를 사용하는 것이 좋다.

    ```kotlin
    var p = Person(first = "North", moddle = null, last = "West")
    val middleNameLength = p.middle?.length
    ```

  - 안전 호출 연산자를 사용한 결과 추론 타입도 결과적으로 nullable한 타입니다. 안전호출 연산자와 엘비스 연산자(?:)를 병행해서 사용하는 것이 NPE를 대응하는데 유리하다.

    ```kotlin
    var p = Person(first = "North", moddle = null, last = "West")
    val middleNameLength = p.middle?.length ?: 0
    ```

  - middle이 null일 경우 0을 리턴한다.

  - 엘비스 연산자의 오른쪽은 식이므로 함수의 인자를 확인할 때 return이나 throw를 사용할 수 있다.

- 코틀린은 안전 타입 변환 연산자(as?)를 제공한다.(safe cast 연산자)

  -  사용 목적은 ClassCastException 방지

    ```kotlin
    val p1 = p as? Person
    ```

  - 변수 p1의 타입은 Person? 이다.

  - 타입 변환이 성공하여 Person 타입이 되거나, 실패하여 null 값을 갖거나

### 2.2 자바에 널 허용성 지시자 추가하기

- 코틀린 코드가 자바 코드와 상호 작용이 필요하고 널 허용성 애노테이션을 강제하고 싶다면?
  - 코틀린 코드에 JSR-305 널 허용성 애노테이션을 강제하려면 컴파일 타임 파라미터 Xjsr305=strict를 사용한다.

- 많은 라이브러리가 JSR-305 호환 애노테이션을 사용중이며 코틀린은 호환 라이브러리를 지원한다. 아래의 코드로 호환성을 강제할 수 있다.

  ```kotlin
  //코틀린 DSL
  tasks.withType<KotlinCompile> {
    kotlinOptions {
      jvmTarget = "1.8"
      freeCompilerArgs = listOf("-Xjsr305=strict")
    }
  }
  ```

### 2.3 자바를 위한 메소드 중복

- 기본 파라미터를 가진 코틀린 함수가 있는데, 자바에서 각 파라미터의 값을 직접적으로 명시하지 않고 해당 코틀린 함수를 사용하고 싶다면?
  - @JvmOverloads 애노테이션을 해당 함수에 추가한다.
- 필요한 상황에서 추가 학습

### 2.4 명시적으로 타입 변환하기

- 코틀린은 자동으로 기본 타입을 더 넓은 타입으로, 예를 들어 Int를 Long으로 승격하지 않는다.

  - 다 작은 타입을 명시적으로 변환하려면 toInt, toLong 등 구체적인 변환 함수를 사용한다.
  - toByte(), toChar(), toShort(), toInt, toLong(), toFloat(), toDouble()

- 다행이도 코틀린은 타입 변환을 투명하게 수행하는 연산자 중복 장점이 있기 때문에 다음 코드는 명시적 타입 변환이 필요하지 않다.

  - 알아서 intVar의 값을 long으로 변환하고 long 리터럴에 그 값을 더한다.

  ```kotlin
  val longSum = 3L + intVal
  ```

### 2.5 다른 기수로 출력하기

- 십진법이 아닌 다른 기수를 사용하는 숫자를 출력하고 싶다면?

  - 올바른 기수를 위해 확장함수 toString(radix: Int)를 사용하자

- 42를 이진법으로 출력하기

  ```kotlin
  42.toString(2) == "101010"
  ```

### 2.6 숫자를 거듭제곱하기

- 숫자를 거듭제곱하고 싶지만 코틀린에는 미리 정의된 거듭제곱 연산자가 없다는 사실을 알게 됐다면?
  - Int와 Long에 정의되어 있는 코틀린 확장 함수 pow에 위임하는 중위(infix) 함수를 정의한다.
- 필요할 때 학습해서 쓰자!

### 2.7 비트 시프트 연산자 사용하기

- 비트 시프트 연산자를 사용하고 싶다면?
  - 비트 시프트를 위한 shr, shl, ushr 같은 비트 중위 연산자가 있다.
- 필요할 때 학습해서 쓰자!

### 2.8 비트 불리언 연산자 사용하기

- 비트 값에 마스트를 적용하고 싶다.
  - 비트 불리언 연산을 위해 코틀린이 제공하는 and, or, xor, inv 비트 연산자를 사용한다.
- 필요할 때 학습해서 쓰자!

### 2.9 to로 Pair 인스턴스 생성하기

- (보통 Map 항목으로) Pair 클래스의 인스턴스를 생성하고 싶다.

  - 직접 Pair 클래스 인스턴스를 생성하는 것 보다 중위 함수(infix function) to 함수를 사용한다.

- 코틀린은 Pair 인스턴스의 리스트로부터 Map을 생성하는 mapOf 와 같은 맵 생성을 위한 최상위 함수 몇 가지를 제공한다

  ```kotlin
  val map = mapOf(
    "a" to 1,
    "b" to 2,
    "c" to 3
  )
  ```

  ```kotlin
  val p1 = Pair("a", 1)
  val p2 = "a" to 1
  
  assertThat(p1.first, `is`("a"))
  assertThat(p1.second, `is`(1))
  assertThat(p2.first, `is`("a"))
  assertThat(p2.second, `is`(1))
  assertThat(p1, `is`(equalTo(p2)))
  ```

## 3. 코틀린 객체 지향 프로그래밍

- 코틀린의 OOP 특성 중 시간을 들여 살펴볼 가치가 있는 내용을 3장에서 살펴본다.
- 객체 초기화, getter와 setter, late initialization, lazy initialization, 싱글톤 생성, Nothing 클래스 이해하기

### 3.1 const와 val의 차이 이해하기

- 런타임 보다는 컴파일 타임에 변수가 상수임을 나타내야 한다.
  - 컴파일 타임에 상수에 const 변경자를 사용한다.
  - val 키워드는 변수에 한 번 할당되면 변경이 불가능함을 나타내지만 이러한 할당은 실행 시간에 일어난다.
- 코틀린 키워드 val은 값이 변경 불가능한 변수임을 나타낸다.
  - 자바에서 final 키뤄드가 같은 목적으로 사용된다.
  - 코틀린에서 const 변경자도 지원하는 이유는 무엇일까?
  - 컴파일 타임 상수는 반드시 객체나 동반 객체(companion object ) 선언의 최상위 속성 또는 멤버여야 한다.
  - 컴파일 타임 상수는 문자열 또는 기본 타입의 래퍼 클래스이며, getter를 가질 수 없다.
  - 컴파일 시점에 값을 사용할 수 있도록 main 함수를 퐇마한 모든 함수의 바깥쪽에서 할당돼야 한다.
- 코틀린에서 **val은 키워드**지만 **const는 private, inline 등과 같은 변경자**임에 유의하자
  - **그런 이유로 const가 val 키워드를 대체하는 것이 아니라 반드시 같이 쓰여야 한다.**

### 3.2 사용자 정의 획득자와 설정자 생성하기

- 값을 할당하거나 리턴하는 방법을 사용자 정의하고 싶다

  - 코틀린 클래스의 속성에 get과 set 함수를 추가한다.

- 코틀린은 특이하게도 모든 것이 기본적으로 public이다.

- **코틀린 클래스에서 필드는 직접 선언할 수 없다.**

  - 아래 Task 클래스에서는 필드처럼 보이는 속성을 정의한 것이다.

- 여러번 읽다 정리해본다 천천히 따라가보자.

- 아래의 Task 클래스는 name과 priority라는 두 가지 속성을 정의한다.

  ```kotlin
  class Task(val name: String) {
    val priority = 3
    // ...
  }
  ```

  - 속성 하나는 주 생성자 안에, 하나는 클래스의 최상위 멤버로 선언되었다

  - 이 방식으로 priority를 선언할 때의 단점은 apply 블록을 사용해서 priority에 값을 할당할 수 있지만, 클래스를 인스턴스화할 때 priority에 값을 할당할 수 없다는 것이다.

    ```kotlin
    var myTask = Tash().apply { priority = 4 }
    ```

  - 이 방식으로 속성을 정의하면 장점은 쉽게 사용자 정의 획득자와 설정자를 추가할 수 있다는 것이다.

  - 속성 초기화 블록, 획득자, 설정자는 선택사항이다.

  - 속성 타입이 초기값 또는 획득자의 리턴타입에서 추론 가능하다면 속성 타입 또한 선택사항이다.

  - 하지만, 생성자에 선언한 속성에서는 타입 선언이 필수다.

  - isLowPriority를 계산하는 사용자 정의 획득자를 보자

    ```kotlin
    val isLowPriority
      get() = priority < 3
    ```

  - 설명한대로 isLowPriority의 타입은 get() 함수의 리턴 타입으로부터 추론되며, 불리언 타입이다.

  - 사용자 정의 설정자는 속성에 값을 할당할 때마다 사용된다.

  - Priority 값이 반드시 1과 5사이의 값이 되게 하려면 아래와 같이 사용자 정의 획득자를 사용한다

    ```kotlin
    var priority = 3
      set(value) {
        field = value.coerceIn(1..5)
      }
    ```

  - 비로소 public 속성 / private 필드 딜레마의 해법을 살펴봤다.

  - **일반적으로 속성에는 지원필드(backing field)가 필요하지만 코틀린은 자동으로 지원 필드를 생성한다.**

  - 사용자 정의 획득자에서는 **field** 식별자가 코틀린이 생성한 지원 필드를 참조하는 데 사용됐다.

  - field 식별자는 오직 사용자 정의 획득자나 설정자에서만 사용할 수 있다.

  - 속성에서 코틀린이 생성하는 기본 획득자 또는 설정자를 사용하거나 사용자 정의 획득자 또는 설정자에서 field 속성을 통해 지원 필드를 참조하는 경우에는 코틀린이 지원 필드를 생성한다. 이는 위에 정의한 파생 속성 isLowPriority에는 지원 필드가 없다는 뜻이다.

  - 이 예제를 완성하기 위해 생성자를 통해 우선순위를 할당하고 싶다고 가정하자.

  - 이를 수행하는 한 가지 방법은 var 또는 val 키워드를 생략함으로써 속성이 아닌 생성자 파라미터를 사용하는 것이다.

    ```kotlin
    class Task(val name: String, _priority: Int = DEFAULT_PRIORITY) {
      companion object {
        const val MIN = 1
        const val MAX = 5
        const val DEFAULT = 3
      }
      
      var priority = validPriority(_priority)
        set(value) {
          field = validPriority(value)
        }
      
      private fun validPriority(p: Int) = p.coerceIn(Min, MAX)
    }
    ```

  - 파라미터 _priority는 속성이 아니라 생성자의 인자일 뿐이다.

  - 이 인자는 실제 priority 속성을 초기화하는데 사용되고 사용자 정의 설정자는 우선순위 값을 원하는 범위로 강제하기 위해서 그 값이 변경될 때마다 실행된다.

  - 사용자 정의 설정자에서 value는 임의의 명칭임을 유의하자.

  - 다른 함수 파라미터처럼 원하는 이름으로 바꿀 수 있다.

### 3.3 데이터 클래스 정의하기

- Equals, hashCode, toString 등이 완벽하게 갖춰진 엔티티를 나타내는 클래스를 생성하고 싶다.

  - 클래스를 정의할 때 data 키워드를 사용한다.

- 코틀린은 데이터를 담는 특정 클래스의 용도를 나타내기 위해서 data 키워드를 제공한다.

- 클래스 정의에 data를 추가하면 코틀린 컴파일러는 일관된 equals와 hashCode gkatn, 클래스와 속성 값을 보여주는 toString 함수, copy 함수와 구조분해를 위한 component 함수 등 일련의 함수를 생성한다.

- 코틀린 컴파일러는 주 생성자에 선언된 속성을 바탕으로 equals와 hashCode 함수를 생성한다. 

- data 클래스의 copy는 깊은 복사가 아니라 얕은 복사를 수행한다는 점에 유의하자

  - 두개의 OrderItem은 같은 Product를 공유하고 있다.

  ```kotlin
  //class
  data class OrderItem(val product: Product, val quantity: Int)
  
  //test
  @Test
  fun `data copy function is shallow`() {
    val item1 = OrderItem(Product("baseball", 10.0), 5)
    val item2 = item1.copy()
    
    assertAll(
      {assertTrue(item1 == item2)},
      {assertFalse(item1 === item2)},
      {assertTrue(item1.product == item2.product)},
      {assertTrue(item1.product === item2.product)}
    )
  }
  ```

### 3.4 지원 속성 기법

- 클래스의 속성을 클라이언트에게 노출하고 싶지만 해당 속성을 초기화하거나 읽는 방법을 제어해야 한다

  - 같은 타입의 속성을 하나 더 정의하고 사용자 정의 획득자와 설정자를 이용해 원하는 속성에 접근한다

- Customer 클래스가 있고 고객 메세지 목록 또는 고객에 관련된 메모를 보고싶다고 가정하자

  - 고객 인스턴스를 생성할 때마다 모든 메세지를 불러올 필요는 없지만 예제는 작성해보자

    ```kotlin
    class Customer(val name: String) {
      private val _messages: List<String>? = null // 널 허용 private 속성 초기화
      
      val messages: List<String>  // 불러올 속성
        get() {										// private 함수
          if (_messages == null) {
            _messages = loadMessage()
          }
          return _messages!!
        }
      
      private fun loadMessges(): MutableList<String> =
        mutableListOf(
          "Initial contact",
          "Convinced them to use Kotlin",
          "Sold training class. Sweet."
        ).also { println("Loaded messages") }
    }
    ```

- 고객 메시지에 접근하기

  ```kotlin
  @Test
  fun `void messages`() {
    val customer = Customer("Fred").apply { messages }  // messages 처음 로딩
    assertEquals(3, customer.messages.size)							// messages에 다시 접근, 로딩됨
  }
  ```

  - _messages는 private이기 때문에 생성자 속성을 사용해서 messages를 불러올 수 없다
  - 위와 같이 messages를 바로 불러오려면 apply 함수를 사용한다
  - apply 블록에서 messages 호출은 messages를 불러오고 로딩 완료 정보를 출력하는 getter를 호출한다
  - 두 번째로 messages 속성에 접근할 때 messages는 로딩됐기 때문에 로딩 완료 정보 메세지를 출력하지 않는다
  - 위의 예제는 설명에 유용하지만 지연 로딩(**lazy loading**)을 어렵게 구현한 것이다.

- 아래와 같이 코틀린 내장 lazy 대리자 함수를 사용하면 더 쉬운 방법으로 코드를 구현할 수 있다.

  ```kotlin
  class Customer(val name: String) {
    val messages: List<String> by lazy { loadMessages() }
    
    private fun loadMessages(): MutableList<String> =
      mutableListOf(
        "Initial contact",
        "Convinced them to use Kotlin",
        "Sold training class. Sweet."
      ).also { println("Loaded messages") }
  }
  ```

- 하지만, 속성 초기화를 강제하기 위한 private 지원 필드(backing field)의 사용은 유용한 방법이다.

  - _priority 속성은 실제 클래스 속성이 아니라 오직 생성자 인자임을 나타내는 val로 명시되지 않았음에 유의하자

  - 제어하고 싶은 priority 속성은 생성자 인자를 바탕으로 해당 속성 값을 할당하는 사용자 정의 설정자를 가진다

  - 지원 속성(Backing property) 기법은 코틀린 클래스에서 자주 볼 수 있으므로 동작 방식을 알아 두는 것이 좋다

    ```kotlin
    class Task(val name: String, _priority: Int = DEFAULT_PRIORITY) {
      companion object {
        const val MIN = 1
        const val MAX = 5
        const val DEFAULT = 3
      }
      
      var priority = validPriority(_priority)
        set(value) {
          field = validPriority(value)
        }
      
      private fun validPriority(p: Int) = p.coerceIn(Min, MAX)
    }
    ```

### 3.5 연산자 중복

- 라이브러리에 정의된 클래스와 더불어 +, * 와 같은 연산자를 사용할 수 있는 클라이언트를 만들고 싶다
  - 코틀린의 연산자 중복(overloading) 메커니즘을 사용해서 +, * 등의 연산자와 연관된 함수를 구현 한다.

### 3.6 나중 초기화를 위해 lateinit 사용하기

- 생성자에 속성 초기화를 위한 정보가 충분하지 않으면 해당 속성을 널 비허용 속성으로 만들고 싶다

  - 속성에 lateinit 변경자를 사용한다.
  - lateinit은 꼭 필요한 경우만 사용하자. 이 장에서 설명하는 의존성 주입의 경우 유용하지만 일반적으로 가능하다면 lazy 같은 대안을 먼저 고려하자

- 널 비허용으로 선언된 클래스 속성은 생성자에서 초기화되어야 한다

  - 스프링 프레임워크나 유닛테스트의 설정 메소드 안에서 속성에 할당할 값의 정보가 충분하지 않은 경우가 있다

  - 예를 들어 스프링은 애플리케이션 컨텍스트로 부터 의존성에 값을 할당하기 위해 @Autowired를 사용한다. 값은 인스턴스가 이미 생성된 후에 설정되므로 해당 값을 lateinit으로 표기해야 한다.

    ```kotlin
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    class OfficerControllerTests {
      @Autowired
      lateinit var client: WebTestClient // 오토와이어링에 의한 초기화
      
      @Autowired
      lateinit var repository: OfficerRepository
      
      @Before
      fun setUp() {
        repository.addTestData() // setUp 함수에서 repository 사용
      }
      
      @Test
      fun `GET to route returns all officers in db`() {
        client.get().uri("/route") // 테스트에서 client 사용
          ...
      } 
    }
    ```

- lateinit 변경자는 클래스 몸체에서맘 선언되고 사용자 정의 획득자와 설정자가 없는 var 속성에서만 사용할 수 있다

- 코틀린 1.2부터 최상위 속성과 지역 변수에서도 lateinit을 사용 가능하다

- lateinit을 사용할 수 있는 속성 타입은 널 할당이 불가능한 타입이어야 하며 기본 타입에는 lateinit을 사용할 수 없다

- lateinit을 추가하면 해당 변수가 처음 사용되기 전에 초기화 할 수 있다.

- 초기화 하기 전에 실패하면 UninitializedPropertyAccessException 을 던진다

- 속성 레퍼런스의 isInitialized를 사용해서 해당 속성의 초기화 여부를 확인할 수 있다.

- **lateinit과 lazy의 차이**

  - lateinit은 위의 제약사항과 함께 var 속성에 사용된다. Lazy 대리자는 속성에 처음 접근할 때 평가되는 람다를 받는다.
  - 초기화 비용이 높은데 lazy를 사용한다면 초기화는 반드시 실패한다
  - lazy는 val 속성에 사용할 수 있는 반면 lateinit은 var 속성에만 적용할 수 있다
  - lateinit 속성은 속성에 접근할 수 있는 모든 곳에서 초기화할 수 있기 때문에 앞의 예제에서처럼 객체 바깥에서도 초기화할 수 있다.

### 3.7 equals 재정의를 위해 안전 타입 변환, 레퍼런스 동등, 엘비스 사용하기

- 논리적으로 동등한 인스턴스인지를 확인하도록 클래스의 equals 메소드를 잘 구현하고 싶다.

  - 레퍼런스 동등 연산자(===), 안전 타입 변환 함수(as?), 엘비스 연산자(?:)를 다 같이 사용한다.

- 모든 객체자향 언어는 객체 동일(equivalence)과 객체 동등(equality) 개념이 있다.

  - 자바에서는 두개의 등호연산자(==)가 서로 다른 레퍼런스에 같은 객체가 할당 됐는지 확인하는데 사용된다.
  - 반면에 Object 클래스 일부인 equals 메소드는 재정의되어 두 객체의 동등 여부를 확인하기 위해 사용된다.
  - **코틀린에서 == 연산자는 자동으로 equals 함수를 호출한다.**

- 코틀린의 여러 가지 장점을 가지고 있는 간단하고 우아한 equals 구현에 주목하자

  - 먼저 ===으로 레퍼런스 동등성을 확인
  - 그다음 인자를 원하는 탑으로 변환하거나 널을 리턴하는 안전 타입 변환 연산자 as?를 사용
  - 안전 타입 변환 연산자가 널을 리턴하면 같은 클래스의 인스턴스가 아니브로 동등일 수 없기 때문에 엘비스 연산자 ?:는 false를 리턴
  - 마지막으로 equals 함수의 마지막 줄은 현재 인스턴스의 version 속성이 other 객체의 version 속성과 동등 여부를 검사해 결과를 리턴

- hasCode의 구현은 간단하다

  ```kotlin
  class Customer(val name: String) {
    override fun equals(other: Any?) Boolean {
      if (this === other) return true
      val otherCustomer = (other as? Customer) ?: return false
      return this.name == otherCustomer.name
    }
    
    override fun hasnCode() = name.hashCode()
  }
  ```

### 3.8 싱글톤 생성하기

- 클래스 하나당 인스턴스는 딱 하나만 존재하게 만들고 싶다

  - class 대신 object 키워드를 사용한다

- 코틀린에서 싱글톤 구현은 class 대신 object 키워드를 사용하기만 하면 된다

  - 이를 객체 선언(object declaration)이라고 한다.

  ```kotlin
  object MySingleton {
    val myProperty = 3
    
    fun myFunction() = "Hello"
  }
  ```

  - 코틀린에서 싱글톤 멤버에 접근하기

    ```kotlin
    MySingleton.myFunction()
    MySingleton.myProperty
    ```

  - 자바에서 코틀린 싱글톤 멤버에 접근하기

    ```kotlin
    MySingleton.INSTANCE.myFunction();
    MySingleton.INSTANCE.getMyProperty();
    ```

### 3.9 Nothing에 관한 야단법석

- Nothing 클래스를 사용법에 맞게 적절하게 사용하고 싶다.

  - 절대 리턴하지 않는 함수에 Nothing을 사용한다.

- Nothing은 클래스 안에서도 인스턴스화하지 않는다. 따라서 Nothing의 인스턴스는 존재하지 않는다.

  ```kotlin
  package kotlin
  
  public class Nothing private constructor()
  ```

- 결코 존재할 수 없는 값을 나타내기 위해 Nothing을 사용할 수 있다.

  - 리턴 없이 예외 던지는 함수의 리턴 타입에 Nothing을 사용한다.

    ```kotlin
    fun doNothing(): Nothing = throw Exception("Nothing at all")
    ```

  - 구체적인 타입 없이 변수에 널 할당하는 경우 컴파일러가 추론하는 변수의 타입은 Nothing?이다.

    ```kotlin
    val x = null
    ```

## 4. 함수형 프로그래밍

- 함수형 프로그래밍 이라는 용어는 불변성(immutability)을 선호하고
- 순수 함수를 사용하는 경우에 동시성을 쉽게 구현할 수 있으며
- 반복보다는 변형(transformation)을 사용하고
- 조건문 보다는 필터를 사용하는 코딩 스타일을 지칭한다
- 5,6,13장과 같은 레시피에 등장하는 map과 filter 같은 함수형 프로그래밍에 사용되는 여러 코틀린 함수를 설명한다.
- 자바와 대조적으로 독특한 꼬리 재귀나 다소 다르게 구현된 fold 또는 reduce 같은 코틀린 함수를 알아본다

### 4.1 알고리즘에서 fold 사용하기

- 반복 알고리즘을 함수형 방식으로 구현하고 싶다

  - fold 함수를 사용해 시퀀스나 컬렉션을 하나의 값으로 축약(reduce) 시킨다

- fold 함수의 문법

  - fold는 2개의 인자를 받는다
  - 첫번째는 누적자의 초기값
  - 두번째는 두개의 인자를 받아 누적자를 위해 새로운 값을 리턴하는 함수다.

  ```kotlin
  inline fun <R> Iterable<T>.fold(
    initlal: R,
    operation: (acc: R, T) -> R
  ): R
  ```

- fold를 사용해서 정수의 합 계산하기

  ```kotlin
  fun sum(vararg nums: Int) = nums.fold(0) { acc, n -> acc + n}
  ```

### 4.2 reduce 함수를 사용해 축약하기

- 비어있지 않은 컬렉션의 값을 축약하고 싶지만, 누적자의 초기값을 설정하고 싶지 않다

  - fold 대신 reduce 연산을 사용한다

- reduce 함수의 시그니처

  ```kotlin
  inline fun <S, T : S> Iterable<T>.reduce(
    operation: (acc: S, T) -> S
  )
  ```

- reduce 함수의 구현은

  - 비어있는 컬렉션은 예외를 발생시키고
  - 누적자는 컬렉션의 첫 번째 원소로 초기화 된다.

- reduce를 이용한 합의 구현

  ```kotlin
  fun sumReduce(vararg nums: Int) = nums.reduce { acc, i -> acc + i}
  ```

- 자바의 reduce -> 코틀린 fold or reduce. 사용할 때 잘 선택해야 한다.

### 4.3 꼬리 재귀 적용하기

- 재귀 프로세스를 실행하는 데 필요한 메모리를 최소화하고 싶다.
  - 꼬리 재귀(tail recursion)를 사용해 프로세스 알고리즘을 표현하고 해당 함수에 tailrec 키워드를 추가한다.
- 필요할 때 학습하자.

## 5. 컬렉션

- 코틀린은 자바처럼 다수의 객체를 담기 위해 타입을 명시한 컬렉션을 사용한다
- 하지만 코틀린은 자바와는 다르게 중개자의 역할을 하는 스트림을 거치지 않고 여러가지 흥미로운 메소드를 컬렉션 클래스에 직접 추가 한다.

### 5.1 배열 다루기

- arrayOf 함수를 이용해 배열을 만들고 Array 클래스에 들어 있는 속성과 메소드를 이용해 배열에 들어 있는 값을 다룬다

- arrayOf라는 이름의 간단한 팩토리 메소드를 제공한다.

- 배열에 접근할 때 사용하는 문법은 자바와 똑같지만, 코틀린에서 Array는 클래스다.

  ```kotlin
  val strings = arrayOf("this", "is", "an", "array", "of", "strings")
  ```

- 널로만 채워진 배열을 생성할 수 있다.

  - 하지만, 컴파일러는 개발자가 어떤 타입의 레퍼런스를 배열에 추가하려는지 알아야 하기 때문에 특정 타입을 선택해야 한다.

  - emptyArray 팩토리 메소드도 동일한 벙법으로 동작한다.

    ```kotlin
    val nullStringArray = arrayOfNulls<String>(5)
    ```

- Array 클래스에는 public 생성자가 하나만 있다. 생성자는 두 인자를 받는다

  - Int 타입의 size
  - init, 즉 (Int) -> T 타입의 람다

- Array 클래스 생성자의 두 번째 인자인 람다는 배열을 생설할 때 인덱스마다 호출된다.

  ```kotlin
  val squares = Array(5) { i -> (i * i).toString() }
  // 결과는 {"0", "1", "4", "9", "16"}
  ```

- 접근시 squares[1] 처럼 대괄호를 사용해 접근할 때 호출되는 public 연산자 메소드 get과 set이 정의 되어 있다.

- 코틀린에는 오토박싱과 언박싱 비용을 방지할 수 있는 기본 타입을 나타내는 클래스가 있다.

  - booleanArrayOf, byteArrayOf, shortArrayOf, charArrayOf, intArrayOf, longArrayOf, floatArrayOf, doubleArrayOf 함수는 예상하는 것 처럼 연관된 타입 배열을 생성한다.

- 코틀린에는 명시적인 기본 타입은 없지만, 값이 널 허용인 경우 Integer와 Double 같은 자바 래퍼 클래스를 사용하고,

- 널 비허용 값인 경우는 int와 double 같은 기본 타입을 사용한다.

- Array의 **indices 속성**을 사용하면 인덱스 값을 알 수 있다.

  ```kotlin
  val strings = arrayOf("this", "is", "an", "array", "of", "strings")
  val indices = strings.indices
  assertThat(indices, contains(0,1,2,3,4,5))
  ```

- 배열의 인덱스 값도 같이 사용해서 for-in 루프를 순회하고 싶다면 withIndex 함수를 사용하자

  ```kotlin
  fun <T> Array<out T>.withIndex(): Iterable<IndexedValue<T>>
  
  data class IndexedValue<out T>(public val index: Int, public val value: T)
  ```

  - withIndex함수에 선언된 IndexedValue는 index와 value 속성을 가진 데이터 클래스다.

  ```kotlin
  val strings = arrayOf("this", "is", "an", "array", "of", "strings")
  for ((index, value) in strings.withIndex()) {
    ...
  }
  ```

### 5.2 컬렉션 생성하기

- **listOf, setOf, mapOf** 처럼 **변경 불가능한 컬렉션**을 생성하기 위해 만들어진 함수나
- **mutableListOf, mutableSetOf, mutableMapOf** 처럼 **변경 가능한 컬렉션**을 생성하기 위해 고안된 함수 중 하나를 사용한다.
- 기본적으로 코틀린 컬렉션은 불변이다. 원소를 추가하거나 제거하는 메소드를 지원하지 않는다.
- 변경하는 메소드는 mutableXXX 인터페이스에 들어 있다.

### 5.3 컬렉션에서 읽기 전용 뷰 생성하기

- 변경 가능한 리스트, 셋, 맵이 있을 때 해당 컬렉션의 읽기 전용 버전을 생성하고 싶다.

- toList, toSet, toMap 메소드를 사용해 새로운 읽기 전용 컬렉션을 생성하자.

  - List, Set, Map 타입의 변수에 기존 컬렉션을 할당한다

- 변경이 가능한 읽기 전용 버전을 생성하는 방법은 두 가지다.

  ```kotlin
  // toList 호출
  val readOnlyNumList: List<Int> = mutableNums.toList()
  
  // List 타입 레퍼런스에 가변 리스트 할당
  val readOnlySameList: List<Int> = mutableNums
  ```

- 내용은 같지만 더 이상 같은 객체를 나타내지 않는다.

### 5.4 컬렉션에서 맵 만들기

- 키 리스트가 있을 때 각각의 키와 생성한 값을 연관시켜서 맵을 만들 때

  - associateWith 함수에 각 키에 대해 실행되는 람다를 제공해 사용한다

- associate 사용

  ```kotlin
  val keys = 'a'...'f'
  val map = keys.associate { it to it.toString().repeat(5).capitalize() }
  
  // {a=Aaaaa, b=Bbbbb, c=Ccccc, d=Ddddd, e=Eeeee}
  ```

- associateWith 사용

  ```kotlin
  val keys = 'a'...'f'
  val map = keys.associateWith { it.toString().repeat(5).capitalize() }
  ```

### 5.5 컬렉션이 빈 경우 기본값 리턴하기

- 컬렉션 처리시 컬렉션의 모든 원소가 선택에서 제외되지만 기본 응답을 리턴하고 싶다

  - ifEmpty와 ifBlank 함수를 사용해 기본값을 리턴한다

  ```kotlin
  // 빈 컬렉션에 기본 리스트를 제공
  products.filter {it.onSale}
    .map { it.name }
    .ifEmpty{ listOf("none") }
    .joinToString(seperator = ", ")
  
  // 빈 문자열에 기본 문자열을 제공
  products.filter {it.onSale}
    .map { it.name }
    .joinToString(seperator = ", ")
    .ifEmpty { "none" }
  ```

- 코틀린도 Optional을 지원하지만, 특정 값을 리턴하는 방법도 있다.

### 5.6 주어진 범위로 값 제한하기

- 주어진 값이 특정 범위 안에 들면 해당 값을 리턴하고 그렇지 않다면 범위의 최솟값 또는 최댓값을 리턴하고 싶다

  - Kotlin.ranges의 **coerceIn 함수**를 범위 인자 또는 구체적인 최소, 최댓값과 함께 사용한다.

- 이 함수는 min과 max 사이의 값은 해당 값을 리턴하고, 그렇지 않다면 경계 값을 리턴한다.

- 범위로 지정

  ```kotlin
  val range = 3..8
  
  assertThat(5, `is`(5.coreceIn(range)))
  assertThat(range.start, `is`(1.coreceIn(range)))        //3
  assertThat(range.endInclusive, `is`(9.coreceIn(range))) //8
  ```

- 최솟값, 최댓값으로 지정

  ```kotlin
  val min = 2
  val max = 6
  
  assertThat(5, `is`(5.coreceIn(min, max)))
  assertThat(min, `is`(1.coreceIn(min, max)))
  assertThat(max, `is`(9.coreceIn(min, max)))
  ```

### 5.7 컬렉션을 윈도우로 처리하기

- 값 컬렉션이 주어진 경우 컬렉션을 횡단하는 작은 윈도우를 이용해 컬렉션을 처리하고 싶다

- 컬렉션을 같은 크기로 나누고 싶다면 chunked 함수를 사용

- 정해진 간격으로 컬렉션을 따라 움직이는 블록을 원한다면 windowed 함수를 사용

- chunked

  - 컬렉션을 주어진 크기 또는 그보다 더 작게 리스트의 리스트로 분할한다.

    ```kotlin
    val range = 0..10
    
    val chunked = range.chunked(3)
    assertThat(chunked, contains(listOf(0,1,2), listOf(3,4,5), listOf(6,7,8), listOf(9, 10)))
    
    assertThat(range.chunked(3) { it.sum() }, `is`(listOf(3, 12, 21, 19)))
    assertThat(range.chunked(3) { it.average() }, `is`(listOf(1.0, 4.0, 7.0, 9.5)))
    ```

- windowed

  ```kotlin
  public fun <T> Iterable<T>.chunked(size: Int): List<List<T>> {
    return windowed(size, size, partialWindows = true)
  }
  ```

  - 세 개의 인자를 받고 그 중 2개의 인자는 선택사항

    - size : 각 윈도우에 포함될 원소의 개수
    - step : 각 단계마다 전진할 원소의 개수
    - partialWindows : 나뉘어 있는 마지막 윈도우 원소 개수가 모자랄 경우 유지할지 여부를 결정. 기본값 false

    ```kotlin
    val range = 0..10
    
    assertThat(range.windowed(3,3) { it.sum() }, `is`(listOf(3, 12, 21)))
    assertThat(range.windowed(3,3) { it.average() }, `is`(listOf(1.0, 4.0, 7.0)))
    
    // 1칸 씩 전진
    assertThat(
      range.windowed(3,1),
      contains(
        listOf(0,1,2), listOf(1,2,3), listOf(2,3,4),
        listOf(3,4,5), listOf(4,5,6), listOf(5,6,7),
        listOf(6,7,8), listOf(7,8,9), listOf(8,9,10)
      )
    )
    ```

### 5.8 리스트 구조 분해하기

- 리스트 원소에 접근할 수 있게 구조 분해

- 최대 5개의 원소를 가진 그룹에 리스트를 할당한다.

  - 현재 List 클래스는 처음 다섯 원소의 component 함수만 정의되어 있다. 추후 코틀린 버전에서 변경 될 수 있다.

  ```kotlin
  val list = listOf("a", "b", "c", "d", "e", "f", "g")
  
  val (a, b, c, d, e) = list
  ```

- 구조 분해는 componentN 함수의 존재에 의존한다. List 클래스에는 component1 ~ component5의 구현이 들어있기 때문에 위의 코드가 동작했다.

### 5.9 다수의 속성으로 정렬하기

- sortedWith와 comparedBy 함수를 사용한다.

- 연이은 속성으로 골프 선수 정렬하기

  ```kotlin
  val sortedGolfer = golfers.sortedWith(
    comparedBy({ it.score }, { it.last }, { it.first })
  )
  ```

- comparedBy는 Comparator를 생성하고 sortedWith 함수는 Comparator를 인자로 받는다.

  - comparedBy의 흥미로운 점을 Comparable 속성을 추출하는 선택자 목록을 제공한다 는 것과 차례차례 Comparator를 생성한다는 것이다.

    ```kotlin
    val comparator = compareBy<Golfer>(Golfer::score)
      .thenBy(Golfer::last)
      .thenBy(Golfer::first)
    
    golfer.sortedWith(comparator)
    ```

- **sortedWith, sortBy는 변경 가능 컬렉션을 요구한다.**

### 5.10 사용자 정의 이터레이터 정의하기

- 컬렉션을 감싼 클래스를 손쉽게 순회하고 싶다.

- next와 hasNext 함수를 모두 구현한 이터레이터를 리턴하는 연산자 함수를 정의한다.

- 팀에 속한 선수 순회하기

  ```kotlin
  val team = Team("member")
  team.addPlayer(Player("A"), Player("B"), Player("C"), Player("D"))
  
  for (player in team.players) {
    ...
  }
  ```

- 직접 팀을 순회하기

  ```kotlin
  operator fun Team.iterator() : Iterator<Player> = players.iterator()
  
  for (player in team) {
    ...
  }
  ```

- Iterable 인터페이스를 구현한 팀 클래스 내부에서 iterator를 오버라이드해서 함수를 확장할 수 있다.

### 5.11 타입으로 컬렉션을 필터링하기

- filterIsInstance 또는 filterIsInstanceTo 확장 함수를 사용해서 특정 타입의 원소로만 구성된 새 컬렉션을 생성하고 싶다.

- 아래의 코드에서 필터링 연산은 동작하지만 strings 변수의 추론 타입은 List\<Any\>이기 때문에, 개별 원소를 String으로 타입 변환하지 않는다.

  ```kotlin
  val strings = list.filter { it is String }
  
  for (s in strings) {
    // s.length -> 컴파일 되지 않는다. 타입이 삭제됨.
  }
  ```

- is확인을 추가하거나 filterIsInstance 함수를 대신 사용할 수 있다.

  - filterIsInstance 함수는 구체적인 타입을 사용하기 깨문에 필터링 결과 컬렉션의 타입을 안다.
  - 따라서 원소의 속성을 사용하기 전에 타입을 확인할 필요가 없다.

  ```kotlin
  val strings = list.filterIsInstance<String>()
  
  assertThat(strings, containsInAnyOrder("a", "b"))
  ```

- filterIsInstanceTo

  - 이 함수의 인자는 MutableCollection 이므로 원하는 컬렉션의 타입을 명시해 해당 타입의 인스턴스로 컬렉션을 채울 수 있다.

  ```kotlin
  val list = listOf("a", LocalDate.now(), 3, 4, "b")
  val strings = list.filterIsInstanceTo(mutableListOf<String>())
  
  assertThat(strings, containsInAnyOrder("a", "b"))
  ```

### 5.12 범위를 수열로 만들기

- 범위를 순회하고 싶지만 범위가 간단한 정수 또는 문자열로 구성되어 있지 않을때

- 사용자 정의 수열(progression)을 생성한다.

- 코틀린에서는 1..5  처럼 IntRange를 인스턴스화 하는 2개의 점 연산자를 통해 범위를 생선한다.

- 두 끝점은 모두 범위에 포함된다.

- LocalDate 으로 살펴보는 범위 연산자

  ```kotlin
  val startDate = LocalDate.now()
  val midDate = startDate.plusDays(3)
  val endDate = midDate.plusDays(5)
  
  val dateRange = startDate..endDate
  
  assertAll(
    { assertTrue(startDate in dateRange) },
    { assertTrue(midDate in dateRange) },
    { assertTrue(endDate in dateRange) },
    { assertTrue(endDate.plusDays(1) !in dateRange) }
  )
  ```

  - 위의 코드는 잘 동작하지만, 이 범위를 순회하려고하면 컴파일 에러가 난다.
  - 범위가 수열이 아니라는 문제가 있다.
  - 사용자 정의 수열은 코틀린 표준 라이버브리의 IntProgression, LongProgression, CharProgression처럼 Iterable 인터페이스를 구현해야 한다.

- 구현해서 쓰자.

  - LocalDate를 위한 수열 LocalDateProgression 
    - Iterable\<LocalDate\> 와 ClosedRange\<LocalDate\> 를 구현
  - LocalDateProgressionIterator 
    - iterator() 구현을 위해 클래스를 만들고 날짜를 순회하도록 내부적으로 next와 hasNext를 구현한다.
    - LocalDateProgression에서 LocalDateProgressionIterator 를 사용해 iterable을 구현한다.
  - LocalDate의 rangeTo 함수를 확장한다.
    - LocalDateProgression

- 두개의 클래스와 1개의 확장함수를 사용해서 원하는 클래스에 패턴을 복제해서 사용

## 6. 시퀀스

- 자바 스트림과 코틀린 시퀀스 사이의 유사점과 차이점을 살펴 본다
- 컬렉션에서는 처리는 즉시 발생한다.
- 컬렉션의 map이나 filter가 호출될 때 컬렉션의 모든 원소는 즉시 처리 된다.
- 반면에 **시퀀스는 지연처리 된다.**
- 데이터를 처리하기 위해 시퀀스를 사용하면 각각의 원소는 자신의 다음 원소가 처리되기 전에 전체 파이프라인을 완료 한다.
- 지연 처리 방식은 데이터가 많거나 first 같은 쇼트 서킷 연산의 경우에 도움이 되고, 원하는 값을 찾았을 때 시퀀스를 종료할 수 있게 도와준다.

### 6.1 지연 시퀀스 사용하기

- 특정 조건을 만족시키는 데 필요한 최소량의 데이터만 처리하고 싶다

  - 코틀린 시퀀스를 쇼트 서킷 함수와 함께 사용 한다.

- 예제. 3으로 나누어지는 첫 번째 배수 찾기

  - map, filter, first

    ```kotlin
    (100 until 200)
      .map { it * 2 }           // 1. 100번
      .filter { it % 3 == 0 }   // 2. 100번
      .first()
    ```

  - map, first

    ```kotlin
    (100 until 200)
      .map { it * 2 }           // 1. 100번
      .filter { it % 3 == 0 }   // 2. 3번
      .first()
    ```

  - asSequence, map, first(filter & first도 상관 없음)

    ```kotlin
    (100 until 200)
      .asSequence()
      .map { it * 2 }          // 1-1. 1번, 2-1. 1번, 3-1. 1번
      .first { it % 3 == 0 }   // 1-2. 1번, 2-2. 1번, 3-2. 1번
                               // 6번째 연산에서 조건 만족해서 결과 리턴
    ```

- 시퀀스가 비어 있다면 first는 예외를 던진다.

- 시퀀스가 Nullable이라면 first 대신 firstOfNull을 사용해야 한다.

- 최종 연산 없이는 시퀀스가 데이터를 처리하지 않는다.

  - 연쇄 함수로 구성된 파이프라인에서 최종 연산을 호출해야만 데이터를 처리한다.

### 6.2 시퀀스 생성하기

- 원소가 있다면 sequenceOf를

- Iterable이 있다면 asSequence를 사용한다.

- 그 외에는 시퀀스 생성기 generateSequence를 사용한다.

  - 주어진 정수 다음에 나오는 소수 찾기

    ```kotlin
    fun nextPrime(num: Int) = 
      generateSequence(num + 1) { it + 1}
        .first(Int::isPrime)
    ```

### 6.3 무한 시퀀스 다루기

- 널을 리턴하는 시퀀스 생성기를 사용하거나, 시퀀스 확장 함수 중에서 takeWhile 같은 함수를 사용하자

- 처음 N개의 소수 찾기

  - 2부터 시작하는 소수의 무한 시퀀스를 6.2에서 본 nextPrime()을 통해 만들고

  - 요청한 수 만큼만 원소를 가져오는 take(). 상태가 없는 중간 연산이다

  - 최종 연산 toList()

    ```kotlin
    fun firstNPrimes(count: Int) = 
      generateSequence(2, ::nextPrime)
        .take(count)
        .toList()
    ```

- 무한 시퀀스를 잘라내고 싶다면, 마지막에 null을 리턴하는 생성 함수를 사용하면 된다.

  - **null은 시퀀스를 종료한다.**

    ```kotlin
    fun firstNPrimes(max: Int) = 
      generateSequence(2) { n -> if (n < max) nextPrime(n) else null }
        .toList()
        .dropLast(1)
    ```

- 그러나 null을 리턴하는 함수보다 takeWhile을 사용하는 것이 더 쉽다.

  ```kotlin
  fun firstNPrimes(max: Int) = 
    generateSequence(2, ::nextPrime)
      .takeWhile { it < max }
      .toList()
  ```

### 6.4 시퀀스에서 yield하기

- 시퀀스에서 값을 생성하고 싶다면, yield suspend 함수와 함께 sequence를 사용

- sequence 함수의 시그니처

  - sequence 함수의 경우 필요한 때 yield해 값을 생성하는 람다를 제공해야 한다.

  ```kotlin
  fun <T> sequence(
    block: suspend SequenceScope<T>.() -> Unit  //Java = void, Kotlin = Unit
  ): Sequence<T>
  ```

- 공식 문서의 피보나치 수열 예제

  - 새로운 원소가 생성될 때마다 yield 함수는 Pair의 첫번째 원소를 리턴한다.

  ```kotlin
  fun fibonacciSequence() = sequence {
    var terms = Pair(0,1)
  
    while (true) {
      yield(terms.first)
      terms = terms.second to terms.first + terms.second
    }
  }
  ```

- yield 함수는 이터에리터에 값을 제공하고 다음 값을 요청할 때까지 값 생성을 중단한다.

  - sequence()를 사용하는 쪽에서 take()로 값을 요청 한다.

- 따라서, yield는 suspend 함수가 생성한 시퀀스 안에서 각각의 값을 출력하는데 사용된다.

  ```kotlin
  val fibs = fibonacciSequence()
    .take(10)
    .toList()
  
  assertEquals(listOf(0, 1, 1, 2, 3, 5, 8, 13, 21, 34), fibs)
  ```

- yieldAll은 다수의 값을 이터레이터에 넘겨준다

  - 아래 함수로 시퀀스를 생성하고, take 함수로 원하는 원소만큼 얻는다.

  ```kotlin
  val sequence = sequence {
    val start = 0;
    yield(start)
    yield(1..5 step 2)
    yield(generateSequence(8) { it * 3 }) // 무한시퀀스
  }
  
  // 0, 1, 3, 5, 8, 24, 72, ....
  ```

## 7. 영역 함수

- 객체 컨텍스트 안에서 코드 블록을 실행할 목적으로 만든 함수
- let, run, apply, also 에 대하여

### 7.1 apply로 객체 생성 후에 초기화

- 객체를 사용하기 전에 생성자 인자만으로는 할 수 없는 초기화 작업에 apply 함수를 사용한다.

- 객체에 적용할 수 있는 영역 함수apply는 this를 전달하고 this를 리턴하는 확장 함수다

  ```kotlin
  inline fun <T> T.apply(block: T.() -> Unit): T
  ```

- JdbcTemplate 예제

  - 엔티티를 저장하는 동안 db가 기본키를 생성한다면, 제공된 객체가 새로운 키로 갱신 되어야 한다. 
    - executeAndReturnKey 사용.
  - officer 인스턴스가 apply에 this로 전달 되기 때문에 블록 안에서 rank 같은 속성에 접근할 수 있고
  - id 속성은 apply 블록 안에서 갱신 된 후 Officer가 리턴 된다.

  ```kotlin
  @Repository
  class JdbcOfficerDAO(private val jdbcTemplate: JdbcTemplate) {
    private val insertOfficer = SimpleJdbcInsert(jdbcTemplate)
      .withTableName("OFFICERS")
      .usingGeneratedKeyColumns("id")
    
    fun save(officer: Officer) =
      officer.apply {
        id = insertOfficer.executeAndReturnKey(
          mapOf("rank" to rank,
                "first_name" to first,
                "last_name" to last))
      }
  }
  ```

- **apply는 이미 인스턴스화된 객체의 추가 설정을 위해 사용하는 가장 일반적인 방법**이다.

### 7.2 부수 효과를 위한 also 사용

- 코드 흐름을 방해받지 않고 다른 동작을 하고 싶을 때 also 함수를 사용 한다.

- also 함수 시그니처

  - block 인자를 실행 시킨 후에 자신을 리턴한다

    ```kotlin
    public inline fun <T> T.also(block: (T) -> Unit): T
    ```

- 일반적으로 객체에 함수 호출을 연쇄시키기 위해 사용된다.

  ```kotlin
  createBook()
    .also { println(it) }
    .also { Logger.getAnonymousLogger().info(it.toString()) }
  ```

### 7.3 let 함수와 엘비스 연산자 사용

- 레퍼런스가 Null 일때 기본값을 리턴하고 싶다면

  - 엘비스 연산자를 결합한 안전 호출 연산자와 함께 let 영역 함수를 사용하자

- let 함수 시그니처

  - 가장 중요한 점은 let 함수는 컨텍스트 객체가 아닌 **블록의 결과를 리턴한다**는 것

    ```kotlin
    public inline fun <T, R> T.let(block: (T) -> R): R
    ```

  - 그러므로 let은 map처럼 동작한다.

- 문자열 대문자 변경과 문자열 처리, Nullable 처리

  ```kotlin
  fun processNullableString(str: String?) = 
    str?.let {    //안전 호출 연산자와 let 같이 사용
      when {
        it.isEmpty() -> "Empty"
        it.isBlank() -> "Blank"
        else -> it.capitalize()
      }
    } ?: "Null"  //널처리 엘비스 연산자
  ```

### 7.4 임시 변수로 let 사용하기

- 연산 결과를 임시 변수에 할당하지 않고 처리하고 싶을 때

  - 연산에 let 호출은 연쇄하고
  - let에 제공한 람다 또는 함수 레퍼런스 안에서 그 결과를 처리한다

- let 사용 이전

  - 출력을 위해 변수 할당

  ```kotlin
  val numbers = mutableListOf("one", "two", "three", "four", "five")
  val resultList = numbers.map { it.length }.filter { it > 3 }
  println(resultList)
  ```

- let 사용

  ```kotlin
  val numbers = mutableListOf("one", "two", "three", "four", "five")
  numbers.map { it.length }
    .filter { it > 3 }
    .let {
      println(it)
      ...
    }
    // 또는 메서드 레퍼런스 사용 => .let(::println)
  ```

  































