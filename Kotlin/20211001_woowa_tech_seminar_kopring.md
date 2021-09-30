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

