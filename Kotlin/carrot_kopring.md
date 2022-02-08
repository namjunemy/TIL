# 코프링2개월 생존기

> 당근 서버 밋업 / 박용권님
>
> 🥕 밋업자료: [https://www.slideshare.net/arawnkr/12...](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa3N5N0tNSnVZbjZkYnFOSDFEWVhlN05QcGx3Z3xBQ3Jtc0trZWFXdHZGYlVZVWNIUUd5cEdYSzVoTmtFRjVKQnlDdnR3X2NKcDRzbC10ZzRmLXI3eVd2ZkVkV29FcFNyQVhVbk1UZElZMnkwcmtpWkF5NHVSZ2FfWl96bU85NUEyYl9rZUpJdVFEMnVaaE43WVAwNA&q=https%3A%2F%2Fwww.slideshare.net%2Farawnkr%2F12-2-250865357) 
>
> 🥕 밋업코드: [https://github.com/arawn/kotlin-suppo...](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqay1lM0lROTI0UF9XYXdLTldQZGhYUUd4YlV1UXxBQ3Jtc0ttZmFpQy1KRmJEZTRrZnlLRV9BZHhRUE10VTNacFQ2azdsdUFxckczVFlLUEZtVExmR3FYdE1QZ3Y0U0hwcVFJUlREVnN5U0RfYUVFMml0SW5MdmtubTQtR1pUanM4ZDhfTUJXNkdPM1R6X3BPNmdPNA&q=https%3A%2F%2Fgithub.com%2Farawn%2Fkotlin-support-in-spring)

## 코틀린의 철학

- 코틀린은 자바와의 상호운용성에 초점을 맞춘 실용적이고 **간결**하며 **안전**한 언어이다. -kotlin in action-

## Keyword

- ?

- ?.

- ?:

- !!

- return if문

- fun mapItem(item: NewsItem<*>) = if 문

  - 함수에 if문으로 값 대입

- when ~ is ~ else

- sealed class

  - abstract class와 유사

  - 다른점은 제한된 클래스의 계층 구조를 만들때 사용할 수 있다.

  - 클래스 외부에 자신을 상속한 클래스를 둘 수 없다.

  - 만약 상속하려면 sealed class 내부에 중첩해서 선언해야 한다

  - when식에서 sealed class의 모든 하위 클래스를 처리한다면. else 분기가 필요 없다.

    ```kotlin
    sealed class NewsItem<out C> {
      val type: String
        get() = javaClass.simpleName
      abstract val content = C
      
      data class NewTopic(...) : NewsItem<TopicDetails>()
      data class NewPost(...) : NewsItem<Post>()
    }
    
    
    fun mapItem(item: NewsItem<*>) = when (item) {
      is NewsItem.NewTopic -> NewsItemDto(item.c...t.title, item.c...r.username)
      is NewsItem.NewPost -> NewsItemDto(item.c...t.text, item.c...t.author)
      // else 없어도 컴파일 OK
    }
    ```

  - 만약 위의 NewsItem에 하위 data class가 추가 되고, mapItem의 when에서 모든 하위 클래스를 처리하지 않으면 컴파일 에러가 난다.

- 생성자 오버로딩 하지말고, Named arguments로 인자의 이름을 붙여 생성하자.

- 생성자 파라미터 작성시 Default arguments를 사용하면 기본 값을 부여할 수 있다.

- 사실상 언어레벨에서 Builder 패턴을 지원..!

- Anonymous classes

- 함수형 인터페이스

  - interface앞에 fun 키워드를 불이면, 코틀린에서는 함수형 인터페이스로 인지한다

  - 람다로 간결한 코드를 작성할 수 있다.

    ```kotlin
    fun interface AuthorMapper {
      fun map(ref: AggregateReference<User, Long>): User
    }
    ```

    ```kotlin
    ...
    TopicDetails.of(
      topic,
      { ref ->
         userQueryRepository.findById(ref.id!!)
      }
    )
    ```

- 함수 타입

  - 코틀린에서 함수는 일급 시민이다. 따라서 함수를 일반 값처럼 다룰 수 있다.

  - 함수를 변수에 대입하거나

  - 함수를 다른 함수에 전달하거나

  - 함수에 새로운 함수를 만들어서 반환할 수 있다.

    ```kotlin
    ...
    fun of(
      topic: Topic,
      authorMapper: (ref: AggregateReference<User, Long>) -> User
    ) : TopicDetails {
      ...
    } 
    ```

- 코딩 컨벤션(공식 사이트) + 린트

- 스코프 함수

  - let
  - apply
  - run

- 편하고, 안전하게 꺼내쓰기

  - 스프링은 PropertyResolver를 확장해 편의기능을 코틀린 확장 함수로 제공한다.

- 테스트 

  - @Autowired

    - 생성자를 통해 주입해야할 클래스가 하나라면, @Autowired를 통해 생성자를 통해서 주입
    - 여러개면 스프링이 제공하는 @TestConstructor(autowireMode = AutowireMode.ALL) 제공.

  - MockMvcTest

    - andExpect() 체이닝으로 검증 하던 것을

    - 코틀린 dsl로 변경해서 조금 더 목적에 맞고 간결하게 작성 가능.

    - expect 블록 안에서 status, view, model을 함께 검증

      ```kotlin
      mockMvc.get("/forum/topics") {
        accept = MediaType.TEXT_HTML
      }.andExpect {
        status { isOk() }
        view { name("forum/topics") }
        model { attributeExists("topics") }
      }
      
      ```

    - DSL이란?

      - 영역 특화 언어. Domain-Specific Language
      - 코드의 가독과 유지보수성을 좋게 유지
      - 코드 자동완성 기능
      - 컴파일 시점에 문법 오류 인지

  - Mockito Test

    - @MockkBean

    - 코틀린에서 **DSL 스타일로 목 처리**할 수 있도록 도와줌

      ```kotlin
      @MockkBean
      private lateinit var forumReader: ForumReader
      
      ...
      
      every { forumReader.loadTopics() } return topics 
      ```

- 스프링에서 비동기 요청 처리를 하려면 컨트롤러에서 반환타입에 Mono, Flux 등을 명시해야 했지만

  - 스프링팀에서 코루틴을 적극 수용하여 비동기 처리 코드인데, 동기요청인 코드 처럼 작성 가능

- 코루틴 훒어보기

  - suspend, await

  - Github에서 정보를 받아오는 비동기 함수 예시를 보자.

  - GithubOperations.kt

    ```kotlin
    class GithubOperations {
      suspend fun fetchUser(username: String): User {..}
      suspend fun fetchOrganizations(user: User): List<Organization> {..}
      suspend fun fetchRepositories(user: User): List<Repository> {..}
    }
    ```

  - 코루틴 스콥

    ```kotlin
    suspend fun getGithubInfo(username: String, accessToken: String) = coroutineScope {
      val operations = GithubOperations(accessToken)
      
      val user: User = operations.fetchUser(username)
      val organizations = async { operations.fetchOrganizations(user) }
      val repositories = async { operations.fetchRepositories(user) }
      
      GithubInfo(user, organizations.await(), repositories.await())
    }
    ```

- 코틀린과 라우터 DSL

  ```kotlin
  @Bean
  fun personRouter(personHandler: PersonHandler) = router {
    "/person".nest {
      accept(APPLICATION_JSON).nest {
        GET("/{id}", personHandler::getPerson)
        GET(personHandler::listPeople)
      }
      POST(personHandler::createPerson)
    }
  }
  ```

- 스프링 푸(Spring Fu)

  - 스프링부트를 구성하는 DSL
  - JaFu
    - Java DSL
  - KoFu
    - Kotlin DSL
  - DSL로 구성된 스프링부트는 어노테이션과 리플렉션을 최소한으로 쓰고, 자동 클래스 탐지를 거의 쓰지 않는다.
  - 실제로 어노테이션 기반보다 40% 빠르다.
  - 더 나은 것이 아니라, 다른 방법을 제안한 것.