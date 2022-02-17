# 우아한 Meet-Up 코틀린

> 사내 코틀린 밋업을 정리한 내용

## 1. 처음 자바에서 코틀린으로 변경하고 느꼈던 점

> 라이브 선물 스쿼드/박정운님

### 1. Null을 다시 생각하게 됨

NPE를 유발시키는 Nullable 객체가 자바에서는 자연스러웠지만 

코틀린에서는 기본적으로 Null을 허용하지 않기 때문에 

변수나 객체 생성시 Nullable 여부에 대하여 다시 한번 고민하게 된다.

- kotlin

  ```kotlin
  val name: String = null // Error
  val name: String? = null
  ```

### 2. 더이상 '+'를 안써도 괜찮아

문자열 결합시 사용하던 '+'(또는 StringBuilder)를

코틀린에서는 언어차원에서 문자열에 $로 변수를 할당해서 문자열 결합을 할 수 있다.

- kotlin

  ```kotlin
  val name = "NJ"
  val person = Person("NJ")
  
  println("Hello $name")
  println("Hello ${person.name}")
  
  data class Person(
    val name: String
  )
  ```

### 3. 더 이상 'if (object != null)'를 안써도 괜찮아

코틀린에서는 Safe Call 이라고 불리는 `.`(dot) 을 이용해서 null 체크를 하는 기능이 있다.

코드에서 변수의 `?.` 다음 코드블럭은 null이 아닌 경우에만 진행이 되고

변수가 null인 경우에는 Elvis Operation 이라고 불리는 `?:` 을 통해서 처리할 수 있다.

아래와 같이 단계적으로 null 체크가 들어가야 하는 상황에서 Safe Call 기능을 통해 체이닝으로 처리할 수도 있다!

- java

  ```java
  if (person != null) {
    if (person.name != null) {
      return true
    } else {
      return false
    }
  } else {
    return false
  }
  ```

- kotlin

  - Safe Call 체이닝중 하나가 null일 경우 false가 반환된다

  ```kotlin
  val data = person?.name?.let {
    true
  } ?: false
  ```

### 4. switch 안녕~ 반가워 when!

switch-case 구문 보다 간결하게 분기 처리를 할 수 있다.

- java

  ```java
  switch (code) {
    case 1:
      do1();
      break;
    case 2:
      do2();
      break;
    case 3:
      do3();
      break;
    default:
      doDefault();
      break;
  }
  ```

- kotlin 

  ```kotlin
  val code = 3
  
  when (code) {
    1 -> do1()
    2 -> do2()
    3 -> do3()
    else -> doDefault()
  }
  ```

### 5. default 값을 사용할 수가 있다

코틀린에서는 함수의 파라미터에 기본값을 넣을 수 있다.

따라서, 파라미터가 있는 함수를 아규먼트 없이 호출도 가능하다!

```kotlin
fun testMethod(code: Int = 123) {
  ...
}

testMethod()     //Ok
testMethod(321)  //Ok
```

### 6. 더 이상 getter/setter 안 만들어서 좋구나

코틀린의 data 클래스는 getter, setter, toString, equalsAndHashCode, constructor를 포함한다.

```kotlin
data class Person(
  val name: String,
  val age: Int,
  val phoneNumber: String,
  val address: String
)
```

 ### 7. singleton도 간단하게 object로!

자바에서 별도의 코드로 작성했던 싱글톤 클래스를 코틀린에서는 object 키워드를 사용하여 간단하게 사용할 수 있다.

- java

  ```java
  public class Singleton {
    private Singleton() {
      
    }
    
    public static Singleton getInstance() {
      return LazyHolder.INSTANCE;
    }
    
    private static class LazyHolder {
      private static final Singleton INSTANCE = new Singleton();
    }
  }
  ```

- kotlin

  ```kotlin
  object Singleton {
    
  }
  ```

### 8. Type 체크도 간단하게 when-is/else로

자바에서 instanceof로 객체의 타입을 체크 했던 부분을 when-is/else로 간단히 처리할 수 있다.

(코틀린의 Any는 자바의 Object와 다르다. 학습해보자)

- java

  ```java
  void testMethod(Object obj) {
    if (obj instanceof String) {
      doString();
    } else if (obj instanceof Integer) {
      doInteger();
    } else if (obj instanceof Double) {
      doDouble();
    } else {
      doDefault();
    }
  }
  ```

- kotlin

  ```kotlin
  fun testMethod(obj: Any) {
    when (obj) {
      is String -> doString()
      is Int -> doInt()
      is Double -> doDouble()
      else -> doDefault()
    }
  }
  ```

### 9. Mutable과 Immutable에 대해서 다시 한번 고민

자바에서도 final이 있지만, 코틀린에서는 선언으로 변수, 객체 생성에 지정해야하기 때문에 한 번 더 고민할 수 있다.

val = immutable, var mutable

```kotlin
private val immutableObject: ImmutableObject
private var mutablePoint: Int = 0
```

### 10. 확장 함수는 너무 좋다

확장 함수는 특정 자료형에서 바로 특정 함수를 사용할 수 있는 기능이다.

```kotlin
private fun Int myPlus(number: Int) = this + number


12.myPlus(32)
```

### 11. 상속 및 implements가 직관적이다

아래와 같이 `,` 로 상속과 implements를 표현할 수 있다. 하지만, 자바와 마찬가지로 다중상속을 허용하지 않아서 class 2개를 상속 받으면 에러난다.

```kotlin
class TeslaService : CarService(), MyInterface1, MyInterface2
```

### 12. 함수 아규먼트에 이름을 줄 수 있다

함수 아규먼트에 이름을 줄 수 있어서 다른 값이 할당되는 것을 방지할 수 있다.

```kotlin
data class Person(
  val name: String,
  val age: Int,
  val phoneNumber: String,
  val address: String
)

val jungwoon = Person(
  age = 29,
  name = "NJ",
  address = "Korea",
  phoneNumber = "01012341234"
)
```

## 코틀린 스타일 테스트 코드 작성하기

> Reference
>
> - 공통시스템개발팀/김규남님 밋업 발표
>
> - https://kotlinlang.org/docs/home.html
> - https://developer.android.com/kotlin/style-guide

### 코틀린 스타일의 테스트 코드?

- 코틀린을 사용해도 Junit, assertion, Mockito 등을 그대로 사용할 수 있지 않나? 
- 있다.
- 그런데, Junit, assertion, Mockito를 사용하면 리시버, 람다 등을 활용해 중괄호 `{}` 를 사용하는 코틀린 스타일의 코드 작성이 어렵다.
  - [Type safe builders](https://play.kotlinlang.org/byExample/09_Kotlin_JS/06_HtmlBuilder?_gl=1*scn3we*_ga*MTExMjE5OTk1NC4xNjExNjY4NzEx*_ga_J6T75801PF*MTYyNDM2NTA0Mi4xMDguMC4xNjI0MzY1MDQyLjA.&_ga=2.54260653.40338281.1624343617-1112199954.1611668711)
  - [Kotlin Standard Library](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/#kotlin.collections.List)
  - [Kotlin Scope Functions (let, also, apply, run ..)](https://kotlinlang.org/docs/scope-functions.html)

- Junit, assertion, Mockito 사용

  ```kotlin
  @Test
  fun `아이디로 유저 조회시 유저가 반환된다`() {
    val expectedUser = User(name = "NJ")
    given(userRepository.findById(1L)).willReturn(expectedUser)
    
    val searchedUser = userService.getById(1L)
    
    assertThat(searchedUser.name).isEqualTo(expectedUser.name)
  }
  ```

- Kotest, Kotest Assertion, Mockk 사용

  - given -> every
  - willReturn -> answers 
  - isEqualTo -> shouldBe
    - 두개의 변수 가운데 오는 함수를 **Infix Function** 이라고 한다.

  ```kotlin
  "아이디로 유저 조회시 유저가 반환된다" {
    val expectedUser = User(name = "NJ")
    every { userRepository.findById(1L) } answers { expectedUser }
    
    val searchedUser = userService.getById(1L)
    
    searchedUser.name shouldBe expectedUser.name
  }
  
  "1과 2를 더하면 3이 반환된다" {
    val result = sut.calculate("1 + 2")
    
    result shouldBe 3
  }
  ```

### 코틀린 스타일로 작성하면 뭐가 좋나요?

- 장점
  - 코틀린 스타일의 코드와 자바 스타일의 테스트 코드의 혼재 방지
  - Kotest 등 프레임워크의 강력한 기능 활용 가능
    - Kotest Test Frameworks(Describe Spec. Behavior Spec 등)
- 단점
  - 새로운 프렘워크, 라이브러리(Kotest, Mockk)에 대한 약간의 러닝커브 존재
  - 일부 이슈 존재
    - [Why is mocking so slow to start in Kotlin?](https://stackoverflow.com/questions/62208145/why-is-mocking-so-slow-to-start-in-kotlin)
      - mockk의 byteBuddy 사용으로 로딩속도 문제

- 발표자 의견
  - 무조건 Kotest나 Mockk 등을 도입하기 보다는 프로젝트 시작 단계에서 팀의 코틀린 숙련도나 상황을 고려하자

### 코틀린에서 사용하는 테스트 도구들

- Kotest
- Mockk
- MockkBean, Spek

### Kotest

- 코틀린 진영에서 가장 많이 사용하는 테스트 프레임워크
- 코틀린 DSL 스타일을 통한 테스트 작성 지원
- 다양한 테스트를 위한 레이아웃 제공
  - String Spec, Annotation Spec, Describe Spec
- https://github.com/kotest/kotest

- **Annotation Spec**

  - 기존 Junit과 가장 유사한 방식의 레이아웃

    ```kotlin
    internal class CalculatorAnnotationSpec: AnnotationSpec() {
        private val sut = Calculator()
      
        private val calculations = listOf(
            "1 + 3 * 5" to 20.0,
            "2 - 8 / 3 - 3" to -5.0,
            "1 + 2 + 3 + 4 + 5" to 15.0
        )
        private val blanks = listOf("", " ", "    ")
        private val invalidInputs = listOf("1 & 2", "1 + 5 % 1")
    
        @Test
        fun `1과 2를 더하면 3이 반환된다`() {
            val result = sut.calculate("1 + 2")
    
            result shouldBe 3
        }
    
        @Test
        fun `식을 입력하면, 해당하는 결과값이 반환된다`() {
            calculations.forAll { (expression, answer) ->
                val result = sut.calculate(expression)
    
                result shouldBe answer
            }
        }
    
        @Test
        fun `입력값이 null 이거나 빈 공백 문자일 경우 IllegalArgumentException 예외를 던진다`() {
            blanks.forAll {
                shouldThrow<IllegalArgumentException> {
                    sut.calculate(it)
                }
            }
        }
    
        @Test
        fun `사칙연산 기호 이외에 다른 문자가 연산자로 들어오는 경우 IllegalArgumentException 예외를 던진다 `() {
            invalidInputs.forAll {
                shouldThrow<IllegalArgumentException> {
                    sut.calculate(it)
                }
            }
        }
    }
    
    ```

- **String Spec**

  - Test 어노테이션 없고, 문자열과 중괄호로 간단하게 사용할 수 있는 테스트 레이아웃

    ```kotlin
    internal class CalculatorStringSpec : StringSpec({
        val sut = Calculator()
    
        "1과 2를 더하면 3이 반환된다" {
            val result = sut.calculate("1 + 2")
    
            result shouldBe 3
        }
    
        "식을 입력하면, 해당하는 결과값이 반환된다" {
            val inputs = listOf(
                "1 + 3 * 5" to 20.0,
                "2 - 8 / 3 - 3" to -5.0,
                "1 + 2 + 3 + 4 + 5" to 15.0
            )
    
            inputs.forAll { (expression, answer) ->
                val result = sut.calculate(expression)
    
                result shouldBe answer
            }
    
        }
    
        "입력값이 null 이거나 빈 공백 문자일 경우 IllegalArgumentException 예외를 던진다" {
            val inputs = listOf("", " ", "    ")
    
            inputs.forAll {
                shouldThrow<IllegalArgumentException> {
                    sut.calculate(it)
                }
            }
        }
    
        "사칙연산 기호 이외에 다른 문자가 연산자로 들어오는 경우 IllegalArgumentException 예외를 던진다" {
            val inputs = listOf("1 & 2", "1 + 5 % 1")
    
            inputs.forAll {
                shouldThrow<IllegalArgumentException> {
                    sut.calculate(it)
                }
            }
        }
    })
    
    ```

- **Behavior Spec**

  - Given, When, Then 패턴의 테스트를 코틀린 DSL을 이용해 블럭 단위로 정의한다.

  - 실제로 테스트 결과에서도 given, when, then이 계층단위로 나누어져서 표시된다.

    ```kotlin
    internal class CalculatorBehaviorSpec : BehaviorSpec({
        val sut = Calculator()
      
        given("calculate") {
            val expression = "1 + 2"
            `when`("1과 2를 더하면") {
                val result = sut.calculate(expression)
                then("3이 반환된다") {
                    result shouldBe 3
                }
            }
    
            val calculations = listOf(
                "1 + 3 * 5" to 20.0,
                "2 - 8 / 3 - 3" to -5.0,
                "1 + 2 + 3 + 4 + 5" to 15.0
            )
            `when`("수식을 입력하면") {
                then("해당하는 결과값이 반환된다") {
                    calculations.forAll { (expression, answer) ->
                        val result = sut.calculate(expression)
    
                        result shouldBe answer
                    }
                }
            }
    
            val blanks = listOf("", " ", "    ")
            `when`("입력값이 null이거나 빈 값인 경우") {
                then("IllegalArgumentException 예외를 던진다"){
                    blanks.forAll {
                        shouldThrow<IllegalArgumentException> {
                            sut.calculate(it)
                        }
                    }
                }
            }
    
            val invalidInputs = listOf("1 & 2", "1 + 5 % 1")
            `when`("사칙연산 기호 이외에 다른 연산자가 들어오는 경우") {
                then("IllegalArgumentException 예외를 던진다"){
                    invalidInputs.forAll {
                        shouldThrow<IllegalArgumentException> {
                            sut.calculate(it)
                        }
                    }
                }
            }
        }
    })
    ```

- **Describe Spec**

  - Junit5로 계층 구조 테스트 작성하기(with DCI 패턴)

    - https://johngrib.github.io/wiki/junit5-nested/

  - Describe(설명대상), Context(상황), It(결과) 패턴의 테스트를 코틀린 DSL을 이용해 정의한다.

  - 실제로 테스트 결과에서도 DCI가 계층단위로 나누어져서 표시된다.

    ```kotlin
    internal class CalculatorDescribeSpec : DescribeSpec({
        val sut = Calculator()
    
        describe("calculate") {
            context("식이 주어지면") {
                val inputs = listOf(
                    "1 + 3 * 5" to 20.0,
                    "2 - 8 / 3 - 3" to -5.0,
                    "1 + 2 + 3 + 4 + 5" to 15.0
                )
    
                it("해당 식에 대한 결과값이 반환된다") {
                    inputs.forAll { (expression, data) ->
                        val result = sut.calculate(expression)
    
                        result shouldBe data
                    }
                }
            }
    
            context("0으로 나누는 경우") {
                it("Infinity를 반환한다") {
                    val result = sut.calculate("1 / 0")
    
                    result shouldBe Double.POSITIVE_INFINITY
                }
            }
    
            context("입력값이 null이거나 공백인 경우") {
                val blanks = listOf("", " ", "      ")
                it("IllegalArgumentException 예외를 던진다") {
                    blanks.forAll {
                        shouldThrow<IllegalArgumentException> {
                            sut.calculate(it)
                        }
                    }
                }
            }
    
            context("사칙연산 기호 이외에 다른 문자가 연산자로 들어오는 경우") {
                val invalidInputs = listOf("1 & 2", "1 + 5 % 1")
                it("IllegalArgumentException 예외를 던진다") {
                    invalidInputs.forAll {
                        shouldThrow<IllegalArgumentException> {
                            sut.calculate(it)
                        }
                    }
                }
            }
        }
    })
    ```

- **Spec with @SpringBootTest**

  - @SpringBootTest를 사용한 통합 테스트에도 Kotest 테스트 레이아웃 적용 가능

  - `testImplementation("io.kotest:kotest-extensions-spring")` 의존성 추가 필요

  - init 블럭안에 들어가야 하지만 충분히 사용하기에 매력적이다.

    ```kotlin
    @SpringBootTest
    internal class CalculatorSpringBootSpec : DescribeSpec() {
        override fun extensions() = listOf(SpringExtension)
    
        @Autowired
        private lateinit var calculatorService: CalculatorService
    
        init {
            this.describe("calculate") {
                context("식이 주어지면") {
                    val inputs = listOf(
                        "1 + 3 * 5" to 20.0,
                        "2 - 8 / 3 - 3" to -5.0,
                        "1 + 2 + 3 + 4 + 5" to 15.0
                    )
                    it("해당 식에 대한 결과값이 반환된다") {
                        inputs.forAll { (expression, data) ->
                            val result = calculatorService.calculate(expression)
    
                            result shouldBe data
                        }
                    }
                }
            }
        }
    }
    ```

- Kotest Isolation Mode

  - 테스트 간의 격리 모드를 지정한다
  - 기본 값 사용시 **Mocking 등에 의해 충돌**이 발생할 수 있다
  - Kotest Isolation Mode
    - SingleInstance - Default
    - InstancePerTest
    - InstancePerLeaf
  - 테스트 간 완전한 격리를 위해서는 **InstancePerLeaf** 를 사용하자
    - Leaf - it 블럭과 같은 가장 말단에서의 격리를 의미한다.

### Mockk

- 코틀린 진영의 Mock 라이브러리

- https://mockk.io/

- 코틀린 DSL 스타일로 mocking 가능

- 일부 키워드가 다르지만 Mockito와 큰 차이는 없다

  ```kotlin
  // Mockito
  given(userRepository.findById(1L)).willReture(expectedUser)
  
  // Mockk
  every { userRepository.findById(1L) } answer { expectedUser }
  ```

- 기본적인 사용법

  - 모의 객체 생성

    - mockk 함수는 타입 파라미터를 이용해서 생성할 모의 객체의 타입을 전달받는다.

    - 변수나 프로퍼티의 타입이 명시적으로 정의되어 있으면 타입 추론이 가능하므로 생략해도 된다.

      ```kotlin
      // 1. mockk<타입>()
      private val mockValidator1 = mockk<CreationValidator>()
      
      // 2. 타입 추론
      private val mockValidator2 : CreationValidator = mockk()
      ```

  - Answer 정의

    - io.mockk.every 함수를 사용하면 된다.

      ```kotlin
      @Test
      fun someMockTest() {
          every { mock.someMethod(1) } returns "OK" // "OK" 리턴
          every { mock.someMethod(2) } throws SomeException() // 익셉션 발생
          every { mock.call() } just Runs // Unit 함수 실행
          
          assertEquals("OK", mock.someMethod(1))
          assertThrows<SomeException> { mock.someMethod(2) }
      }
      ```

  - Argument matching

    - 의의 인자 값과 일치하도록 설정하려면 any()를 사용한다.

      ```kotlin
      @Test
      fun someMockTest() {
          every { mock.anyMethod(any(), 3) } returns "OK"
          
          assertEquals("OK", mock.anyMethod(10, 3))
      }
      ```

  - Relaxed mock

    - MockK는 호출 대상에 대한 스텁 정의를 하지 않으면 오류를 발생한다.

      ```kotlin
      val mock = mockk<Some>()
      
      mock.someMethod(1) // --> io.mockk.MockKException: no answer found for: Some(#1).someMethod(1)
      ```

    - 이를 완화하는 방법은 Relaxed mock을 생성하는 것이다. 

    - mockk()의 relaxed 파라미터 값을 true로 전달하면 Relaxed mock을 생성할 수 있다.

      ```kotlin
      val mock = mockk<Some>(relaxed = true)
      mock.someMethod(1) // --> 0 리턴
      ```

  - 함수 호출 여부 검증

    - io.mockk.verify 함수를 사용해서 호출 여부를 검증할 수 있다.

      ```kotlin
      val mock = mockk<Some>(relaxed = true)
      
      mock.someMethod(1)
      mock.anyMethod(1, 3)
      
      verify { mock.someMethod(1) }
      verify { mock.anyMethod(any(), 3) }
      ```

  - 인자 캡처

    - 인자를 캡처하고 싶을 땐 slot()과 capture()를 사용한다.

      ```kotlin
      val mock = mockk<Some>()
      
      val argSlot = slot<Int>()
      every { mock.someMethod(capture(argSlot)) } returns 3
      
      mock.someMethod(5)
      
      val realArg = argSlot.captured
      assertEquals(5, realArg)
      ```

### SpringMockk

- 기존에 사용하던 @MockBean과 @SpyBean을 대체할 수 있다.

- @MockkBean, @SpykBean 으로 사용

- https://github.com/Ninja-Squad/springmockk

- https://mockk.io/

  ```kotlin
  @ExtendWith(SpringExtension::class)
  @WebMvcTest
  class GreetingControllerTest {
    @MockkBean
    private lateinit var greetingService: GreetingService
    
    @Autowired
    private lateinit var controller: GreetingController
    
    @Test
    fun `should greet by delegating to the greeting service`() {
      every { greetingService.greet("NJ") } returns "Hi NJ"
      
      assertThat(controller.greet("NJ")).isEqualTo("Hi NJ")
      verify { greetingService.greet(NJ) }
    }
  }
  ```

