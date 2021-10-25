## 어디 가서 코프링 매우 알은 체하기!

> 2021.10.01 우아한테크세미나
>
> 우아한형제들 - Jason

## 코틀린이란

- JVM, 안드로이드, 자바스크립트 및 네이티브를 대상으로 하는 정적 타입 지정 언어
  - 컴파일 시점에 타입이 정해져 있다.(== 자바)
- 젯브레인에서 개발한 오픈 소스(아파치 라이선스 2.0)
- OO 스타일과 FP 스타일을 모두 사용할 수 있으며 두 요소를 혼합하여 사용할 수 있다.
- 간결하고 실용적이며 안전하고 기존 언어와의 상호 운용성을 중시한다.
  - \+ 코루틴(코틀린 라이브러리)
  - 코틀린은 자바와 마찬가지로 섬이름에서 따왔다.(러시아에서 군사적으로 중요한 위치에 있음ㅋㅋ)

## 아이템 1. 코틀린 표준 라이브러리를 익히고 사용하라

- 코틀린 1.3부터 모든 플랫폼에서 사용할 수 있는 kotlin.random.Random이 도입되었다.

- 더 이상 Random을 사용할지 ThreadLocalRandom을 사용할지 고민할 필요가 없다.

- 자바와 관련된 import문을 제거할 수 있다.

- 표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.

  - AS-IS

  ```java
  import java.util.Random
  import java.util.concurrent.ThreadLocalRandom
    
  Random().nextInt()
  ThreadLocalRandom.current().nextInt()
  ```

  - TO-BE

  ```kotlin
  import kotlin.random.Random
  
  Random.nextInt() // Thread Safe
  ```

- 코틀린은 읽기 전용 컬렉션과 변경 가능한 컬렉션을 구별해 제공한다.

- 인터페이스를 만족하는 실제 컬렉션이 반환된다. 따라서 플랫폼별 컬렉션을 사용할 수 있다.

## 코틀린 / JVM

- 코틀린 맛보기

  ![image-20211023184104364](./img/01.png)

  ```kotlin
  class Person(val name: String, val age: Int = 1) {
    var nickname: String? = null
  }
  ```

- 위의 코틀린 코드를 자바로 아래와 같이 표현할 수 있다.

  ```java
  public final class Person {
    @NotNull
    private final String name;
    private final int age;
    @Nullable
    private String nickname;
    
    public Person(String name) {
      this(name, 1);
    }
    
    public Person(@NotNull String name, int age) {
      this.name = name;
      this.age = age;
    }
    
    @NotNull
    public final String getName() {
      return name;
    }
    
    public final int getAge() {
      return age;
    }
    
    @Nullable
    public final String getNickname() {
      return nickname;
    }
    
    public final void setNickname(@Nullable String nickname) {
      this.nickname = nickname;
    }
  }
  ```

- 다른점은

  - @NotNull, @Nullable
    - 코틀린에서는 ? 키워드가 Nullable의 의미를 갖는다. 기본은 NotNull
  - final

- 안전하게 코딩하기 위해 위의 관용구들을 사용해야 했었지만, 

- 코틀린에서는 위의 기능을 제공하기 때문에 기본적으로 안전하게 코딩할 수 있고, 반대로 안전하지 않게 코딩하고 싶을 때 의식적으로 선택할 수 있다.

- **final 키워드를 조심해야 한다.**

  - 코틀린에서 기존 자바 라이브러리나 프레임워크와 같이 쓰게되면 결국 final 때문에 문제가 발생하게 된다.

## 아이템 2. 자바로 역컴파일하는 습관을 들여라

- 코틀린 숙련도를 향상시키는 가장 좋은 방법 중 하나는 작성한 코드가 자바로 어떻게 표현되는지 확인하는 것이다.

- 역컴퍼일을 통해 예기치 않은 코드 생성을 방지할 수 있다.

- 기존 자바 라이브러리와 프레임워크를 사용하며 문제가 발생할 때 빠르게 확인할 수 있다.

- IntelliJ IDEA에서 

  ```
  Tools > Kotlin > Show Kotlin Bytecode => Decompile
  ```

#### 코틀린 컴파일

- 코틀린과 자바를 어떻게 같이 사용할 수 있을까?(컴파일 레벨)

- 코틀린 단독 사용시

  - 코틀린 컴파일러가 해당 코틀린 파일이 링킹하고 있는 다른 코틀린 파일 or 자바 파일을 컴파일해서 .class 파일을 만들어 낸다.

    ![image-20211025163251019](./img/03.png)

- 코틀린 자바 동시 사용시

  - 먼저 .kt 파일(코틀린파일)을 **코틀린 컴파일러**가 해석해서 .class 파일을 만들고,

  - 다음으로 **자바 컴파일러**가 

    - 코틀린 컴파일러가 만든 .class 파일과
    - .java 파일을 컴파일한다.
    - 이 과정에서 **애너테이션 프로세싱** 등 자바 컴파일이 동작하게 되고,
    - 결과적으로 .class 파일이 만들어진다.

  - 이 최종 .class 파일이 나중에 실행가능한 jar 등으로 만들어 진다.

    ![image-20211025162924987](./img/02.png)

## 아이템 3. 롬복 대신 데이터 클래스를 사용하라

- 데이터를 저장하거나 전달하는 것이 주 목적인 클래스를 만드는 경우가 많다.
  - 이러한 클래스의 일부 표준 및 유틸리티 함수는 데이터에서 기계적으로 파생된다.
- 자바에서는 롬복의 @Data를 사용하여 보일러플레이트 코드를 생성한다.
- **애너테이션 프로세서는 코틀린 컴파일 이후에 동작하기 때문에 롬복에서 생성된 자바 코드는 코틀린 코드에서 접근할 수 없다.**
- 코틀린 코드보다 자바 코드를 먼저 컴파일 하도록 빌드 순서를 조정하면 롬복 문제는 해결할 수 있다.
  - 하지만 자바 코드에서 코틀린 코드를 호출할 수 없게 된다.
- 추천하는 방향은
  - 일단 롬복을 없앤다. 롬복에서 디롬복 기능을 제공한다.
  - 또는, 롬복이 붙어 있는 코드들을 DTO 코드일 가능성이 많기 때문에 그것들 먼저 코틀린 코드로 마이그레이션 하는 방법도 있다.

#### 코틀린 데이터 클래스

- 데이터 클래스를 사용하면 컴파일러가 equals(), hashCode(), toString(), copy() 등을 자동으로 생성해 준다.

  - 주 생성자의 매개변수 기반으로 메서드가 자동 생성 된다.

- 주 생성자에는 하나 이상의 매개변수가 있어야 하며, 모든 매개변수는 val(불변) 또는 var(가변)로 표시해야 한다.

- copy()를 적절히 사용하면 데이터 클래스를 불변으로 관리할 수 있다.

- 코드가 간단하기 때문에 한 코틀린 파일에 여러 관련 클래스를 담는 것도 좋은 방법이다.

- 코틀린 1.5부터 자바 16의 레코드 클래스도 지원한다.

  ```kotlin
  val njkim = Person(name = '김남준', age = 29)
  val rapMonster = njkim.copy(age = 28)
  
  data class Person(val name: String, val age: Int)
  ```

- 실제로 DTO, Request/Response 를 한 파일에 몰아서 data class로 만들기도 하고,

  ```kotlin
  //Data Transfer Object
  data class RecruitmentResponse(
    val id: Long,
    val title: String,
    val term: TermResponse,
    val recruitable: Boolean,
    val hidden: Boolean,
    val startDateTime: LocalDateTime,
    val endDateTime: LocalDateTime,
    val status: RecruitmentStatus
  ) {
    constructor(recruitment: Recruitment, term: Term) : this(
      recruitment.id,
      recruitment.title,
      TermResponse(term),
      recruitment.recruitable,
      recruitment.hidden,
      recruitment.startDateTime,
      recruitment.endDateTime,
      recruitment.status
    )
  }
  
  data class TermResposne(val id: Long, val name: String) {
    constructor(term: Term) : this(term.id, term.name)
  }
  ```

- VO등과 같이 Equals와 HashCode가 중요하거나 코틀린 copy()를 사용하고 싶은 클래스도 data class를 활용할 수 있다.

  ```kotlin
  // Value Object
  data class RecruitmentPeriod(
    @Column(nullable = false)
    val startDateTime: LocalDateTime,
    
    @Column(nullable = false)
    val endDateTime: LocalDateTime
  ) {
    init {
      require(endDateTime >= startDateTime)
    }
    
    fun contains(value: LocalDateTime): Boolean =
      (startDateTime..endDateTime).contains(value)
  }
  ```

  















