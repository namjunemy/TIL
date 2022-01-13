# 코틀린 스터디

> Oreilly / 코틀린 쿡북
>
> 2022.01.13 ~ 

## 코틀린 Basic

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







