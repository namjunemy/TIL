# 3. 스프링 부트 활용

> [백기선 - 스프링 부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)

## 1. 스프링 부트 활용 소개

* 스프링 부트 핵심 기능
  * SpringApplication
  * 외부 설정
  * 프로파일
  * 로깅
  * 테스트
  * Spring-Dev-Tools
* 각종 기술 연동
  * 스프링 웹 MVC
  * 스프링 데이터
  * 스프링 시큐리티
  * REST API 클라이언트
  * 다루지 않은 내용들

## 2. SpringApplication

SpringApplication 클래스에 대해서 조금 더 살펴 본다.

### 1부

* 스프링 부트 애플리케이션의 기본 로그 레벨은 INFO 이다.

* IntelliJ의 Edit Configuration에서 VM options값을 `-Ddebug` 로 할당해서 run하면, DEBUG 레벨의 로그도 볼 수 있다.

  * DEBUG레벨로 로그를 찍을 때 한 가지 특이한 점은 어떤 자동 설정이 적용 됐는지, 적용 안된 자동 설정은 왜 안됐는지에 관한 로그를 볼 수 있다.

* FailureAnalyzers

  * 애플리케이션 start과정에서 error가 발생했을 때, 에러 메세지를 보기 좋게 출력해준다.
  * 기본적으로 스프링부트에는 여러가지 Failure Analyzer가 등록되어있다.
  * https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-startup-failure

* 배너

  * src/main/resources의 하위에 banner.txt | gif | jpg | png 파일을 위치시키면 된다.
  * yml 파일에 classpath 또는spring.banner.location을 사용해서 위치를 지정할 수 있다.
  * Text to ASCII 제너레이터를 사용해서 하면 예쁘다
    * http://patorjk.com/software/taag
  * 배너에 사용할 수 있는 변수들도 있다.
    * ${spring-boot.version} 등
  * 배너를 app.setBanner(new Banner() { ... }) 로 직접 구현할 수 있다.
  * 배너 끄는 방법

  ```java
  @SpringBootApplication
  public class Application {
  
      public static void main(String[] args) {
          SpringApplication app = new SpringApplication(Application.class);
          app.run(args);
          app.setBannerMode(Banner.Mode.OFF);
      }
  }
  ```

* SpringApplicationBuilder로 빌더 패턴 사용 가능

### 2부

* **ApplicationEvent**

  * https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-application-events-and-listeners
  * 이벤트 리스너 자체를 @Component로 만들어서 사용하면 ApplicationContext가 만들어지기 전에는 **ApplicationStartingEvent**는 반응을 안한다. 아래의 코드이다.

  ```java
  @Component
  public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {
  
      @Override
      public void onApplicationEvent(ApplicationStartingEvent event) {
          System.out.println("========================");
          System.out.println("Application is starting");
          System.out.println("========================");
      }
  }
  ```

  * **ApplicationStartingEvent** 대신 **ApplicationStartedEvent**를 사용하거나,

  ```java
    .   ____          _            __ _ _
   /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
  ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
   \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
    '  |____| .__|_| |_|_| |_\__, | / / / /
   =========|_|==============|___/=/_/_/_/
   :: Spring Boot ::        (v2.1.0.RELEASE)
  
  2018-11-15 15:19:41.137  INFO 15600 --- [           main] i.n.s.Application                        : Starting Application on DESKTOP-EI79USO with PID 15600 (C:\document\github\spring-boot-concept-and-utilization\out\production\classes started by njkim in C:\document\github\spring-boot-concept-and-utilization)
    ...
    ...
    ...
  ========================
  Application is started!!!
  ========================
  ```

  * 메인 클래스에서 **SpringApplication.addListeners()**를 사용해서 이벤트를 직접 등록해주면 된다. 이 때, SampleListener클래스는 @Component 선언을 해줄 필요가 없게 된다.

  ```java
  @SpringBootApplication
  public class Application {
  
      public static void main(String[] args) {
          SpringApplication app = new SpringApplication(Application.class);
          app.addListeners(new SampleListener());
          app.run(args);
      }
  }
  ```

  ```shell
  ========================
  Application is starting
  ========================
  
    .   ____          _            __ _ _
   /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
  ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
   \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
    '  |____| .__|_| |_|_| |_\__, | / / / /
   =========|_|==============|___/=/_/_/_/
   :: Spring Boot ::        (v2.1.0.RELEASE)
   
   ...
   ...
  ```

* **애플리케이션 아규먼트 사용하기**

  * ApplicationArguments를 빈으로 등록해 주니까 가져다 쓰면된다.
  * jar파일을 실행할 때, `# java -jar [path]/~~~SNAPSHOT.jar -Dfoo --bar` 명령으로 실행했을 때 아래의 결과가 나온다.

  ```java
  @Component
  public class ArgumentsTester {
    
    public ArgumentsTester(ApplicationArguments arguments) {
      System.out.println("foo: " + arguments.containsOption("foo"));
      System.out.println("bar: " + arguments.containsOption("bar"));
    }
  }
  ```

  ```shell
  ...
  foo: false
  bar: true
  ...
  ```

* **애플리케이션 실행항 뒤 먼가 실행하고 싶을 때**

  * ApplicationRunner(추천) 또는 CommandLineRunner
  * @Order를 통해서 순서도 지정할 수 있다.

## 3-1. 외부설정 1부

### 프로퍼티 우선 순위

1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
2. 테스트에 있는 @TestPropertySource
3. @SpringBootTest 애노테이션의 properties 애트리뷰트
4. 커맨드라인 아규먼트
5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로퍼티)에 들어있는 프로퍼티
6. ServletConfig 파라미터
7. ServletContext 파라미터
8. Java:comp/env JNDI 애트리뷰트
9. System.getProperties() 자바 시스템 프로퍼티
10. OS 환경 변수
11. RandomValuePropertySource
12. JAR 밖에 있는 특정 프로파일용 application.properties
13. JAR 안에 있는 특정 프로파일용 application.properties
14. JAR 밖에 있는 application.properties
15. JAR 안에 있는 application.properties
16. @PropertySource
17. 기본 프로퍼티(SpringApplication.setDefaultProperties)

#### application.properties 우선 순위(높은게 낮은걸 덮어 쓴다.)

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/

#### properties에서 랜덤값 설정하기

* ${random.자료형}

* server.port의 경우 0을 할당해야 가용범위 안의 포트를 찾아서 맵핑해줌

## 3-2. 외부설정 2부

* 커밋 로그
  * [외부설정 - 타입-세이프 프로퍼티 @ConfigurationProperties 사용하기](https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/8fa0a1cd7e0b569a5aa1e9613c8a9c03564d72f5)
  * [외부설정 - 프로퍼티 타입 컨버전, 프로퍼티 값 검증 [@Validated](https://github.com/Validated)](https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/ce9c5e9f811f8fc4902d29c7a9f703e18a43609c)

* 타입-세이프라는 의미는 @Value("${namjune.name}")과 같이 직접 프로퍼티 값을 입력해서 발생할수 있는 에러를 내지 않을 수 있다는 의미이다. @ConfigurationProperties으로 정의하고 빈으로 만든 뒤 getter를 통해서 값을 가져오기 때문에 @Value로 직접 쓰는 것 보다 안전하게 사용할 수있다.

  * 프로퍼티스 파일 자체가 타입-세이프 하다는 의미는 아니다.

* 타입-세이프 프로퍼티 @ConfigurationProperties

  * 여러 프로퍼티를 묶어서 읽어올 수 있음

  * 빈으로 등록해서 다른 빈에 주입할 수 있음

    * @EnableConfigurationProperties
    * **@Component** 
    * @Bean
    * **스프링부트 애플리케이션에서는 @EnableConfigurationProperties이 등록이 되어 있으므로 @ConfigurationProperties가 선언되어있는 클래스에 @Component를 추가하여 빈으로 만들어 주기만 하면 된다.**

  * 융통성 있는 바인딩(RelaxedBinding)

    * context-path(케밥)
    * context_path(언더스코어)

    * contextPath(카멜)
    * CONTEXTPATH

  * 프로퍼티 타입 컨버전

    * 프로퍼티 파일에 txt가 문자로 입력되지만, int로 컨버전 되어서 들어간다.

    * @DurationUnit

      * 시간정보를 받고 싶을 때 사용하면 컨버전이 이루어 진다.
      * AppProperties.java

      ```java
      public class AppProperties.java {
        ...
        @DurationUnit(ChronoUnit.SECONDS)
      	private Duration sessionTimeout = Duration.ofSeconds(30);
        ...
      }
      ```

      * application.yml

      ```yaml
      nj:
        name: namjune
        age: ${random.int(0,100)}
        fullName: ${nj.name} Kim
        sessionTimeout: 30
      ```

      ```txt
      ==========================
      namjune
      64
      namjune Kim
      PT30S
      ==========================
      ```

  * 프로퍼티 값 검증

    * 프로퍼티 값을 검증하고 싶을때, @Validated 애노테이션을 정의하고 JSR303 구현체인 hibernate-validator 애노테이션을 사용해서 검증한다.
    * @Validated
    * JSR-303(@NotNull, ...) 구현체 = hibernate-validator
    * 메타 정보 생성
    * @Value에서는
      * SpEL 을 사용할 수 있지만..
      * 위에 있는 기능들은 전부 사용 못한다.

## 4. 프로파일

* 커밋로그
  * [스프링 부트 프로파일 설정](https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/1cc4c087672fc7771dc8c896df21bb50405eb72d)

* 어떤 특정한 프로파일에서만 특정한 빈을 등록하고 싶거나, 특정 프로파일에서만 애플리케이션의 동작을 다르게 하고 싶을때 프로파일을 사용했었다.

* @Profile 애노테이션은 어디에?
  * @Configuration
  * @Component
* 어떤 프로파일을 활성화 할 것인가?
  * spring.profiles.active
* 어떤 프로파일을 추가할 것인가?
  * 프로파일안에 특정 프로파일이 정의된 프로퍼티 파일을 인클루드 해서 사용할 수 있다.
  * spring.profiles.include
* 프로파일용 프로퍼티
  * application-{profile}.properties

## 5-1. 로깅 1부

* 커밋로그
  * [기본 로깅 설정](https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/05da2843fd7c07671ff73eb1d7e98a8bd13a1afd)
* 스프링 부트는 기본적으로 로깅 파사드 **Commons Logging**을 사용한다. 결국 SLF4j를 사용하게 된다. 소스코드에서도 SLF4j를 사용하면 된다.
  * 로깅 파사드는 실제 로깅을 하지 않고, 로거 API들을 추상화 해놓은 인터페이스들이다.
  * 주로 프레임워크들은 로깅 파사드를 이용한다. 프레임워크를 사용하는 애플리케이션들의 로거 사용을 자유롭게 해주기 위해서.
* 로깅 파사드의 장점은 로거들을 바꿔서 사용할 수 있다는 것이다.
  * JUL(Java Utility Logging), Log4J2, **Logback**
* 정리하자면 스프링부트에서 찍히는 로그는 Commons Logging -> SLF4j -> Logback의 흐름을 타고 결국 **Logback**에 의해서 찍힌다.
* 아래의 spring-boot-stater-logging 의존성을 통해서 확인할 수 있다.
  * jul-to-slf4j 라이브러리와 log4j-to-slf4j를 통해서 slf4j로 로그를 보내고,
  * slf4j-api 라이브러리를 통해서 받은 로그들을 결국 logback으로 처리한다.

![](https://github.com/namjunemy/TIL/blob/master/SpringBoot/img/03_logging_dependency.PNG?raw=true)

* 스프링 부트 기본 로깅
  * --debug 옵션
    * 일부 코어 라이브러리(embedded container, Hibernate, Spring Boot)만 디버깅 모드로
  * --trace
    * 전부 다 디 버깅 모드로
  * 컬러 출력
    * spring.output.ansi.enabled
  * 파일 출력
    * logging.file 또는 logging.path
    * 로그파일은 기본적으로 10M까지 저장되고, 넘치면 아카이빙하는 등 여러가지 설정도 할 수 있다.
  * 로그 레벨 조정
    * logging.level.패키지 = 로그 레벨

## 5-2. 로깅 2부

* 커밋로그
  * [커스텀 로그 설정]()

* 커스텀 로그 설정 파일 사용하기
  * Logback: logback-spring.xml
    * https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-logback-for-logging
  * Log4J2: log4j2-spring.xml
  * JUL(비추천): logging.properties
  * Logback extension
    * **logback-spring.xml을 사용하면 logback.xml을 사용하는 것과 같고, 스프링부트에서 추가로 아래의 익스텐션을 사용할 수 있게 제공한다.**
    * 프로파일 \<springProfile name="프로파일"\>
    * Environment 프로퍼티\<springProperty\>
* 로거를 Log4j2로 변경하기
  * https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging

## 6-1. 테스트

* 시작은 일단 spring-boot-starter-test를 추가하는 것 부터

  * test scope으로 추가


### @SpringBootTest

* @SpringBootTest가 하는 역할은 @SpringBootApplication을 찾아서 테스트를 위한 빈들을 다 생성한다. 그리고 @MockBean으로 정의된 빈을 찾아서 교체한다.

* @RunWith(SpringRunner.class)랑 같이 써야 함

* 빈 설정 파일은 안해주나? 알아서 찾는다. (@SpringBootApplication)


#### SpringBootTest.webEnvironment

* MOCK : mock servlet environment. 내장 톰캣 구동 안함.

  * 커밋로그 : https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/f051482f41506c88cb64fada0ff7b975fef0c099

  ```java
  package io.namjune.springbootconceptandutilization.sample;
  
  import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
  import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
  import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
  import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
  
  import org.junit.Test;
  import org.junit.runner.RunWith;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.test.context.junit4.SpringRunner;
  import org.springframework.test.web.servlet.MockMvc;
  
  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
  @AutoConfigureMockMvc
  public class SampleControllerTest {
  
      @Autowired
      MockMvc mockMvc;
  
      @Test
      public void hello() throws Exception {
          mockMvc.perform(get("/hello"))
              .andExpect(status().isOk())
              .andExpect(content().string("hello namjune"))
              .andDo(print());
      }
  }
  ```

* RANDOM_PORT, DEFINED_PORT: 내장 톰캣 사용 함
  * RANDOM_PORT를 사용하면 실제 내장 톰캣을 사용한다. 이때는 MockMvc 대신 **RestTemplate**를 사용할 수 있다.
  * 실제 가용한 포트로 내장톰캣을 띄우고 응답을 받아서 테스트를 수행한다.
    * 커밋로그 : https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/6611dc0a8f2edb50b159272c1c728ffd5860ee6d

  ```java
  package io.namjune.springbootconceptandutilization.sample;
  
  import static org.assertj.core.api.Assertions.assertThat;
  
  import org.junit.Test;
  import org.junit.runner.RunWith;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
  import org.springframework.boot.test.web.client.TestRestTemplate;
  import org.springframework.test.context.junit4.SpringRunner;
  
  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
  public class SampleControllerTest {
  
      @Autowired
      TestRestTemplate testRestTemplate;
  
      @Test
      public void hello() {
          String result = testRestTemplate.getForObject("/hello", String.class);
          assertThat(result).isEqualTo("hello namjune");
      }
  }
  ```

* spring5 webflux에서 추가된 RestClient중에 하나인 WebTestClient도 사용할 수 있다. 기존에 사용하던 WebClient는 synchronous하게 동작하기 때문에 요청 하나 보내고 그 요청이 끝나고 난 다음에 다음 요청을 보낼 수 있었지만, WebTestClient는 asynchronous하게 동작하므로 요청을 보내고 기다리지 않는다. 후에 응답이 오면, 콜백 이벤트를 실행할 수 있다. 따라서, Test코드에서도 WebClient와 비슷한 API를 사용할 수 있다.
  * webflux 의존성을 추가해야 한다.
  * API가 restTemplate보다 가독성이 좋다.(추천)
  * **아래의 @MockBean 참고**
    * 커밋로그 : https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/99a3bf23c989dd4c293473e54e71c492258a9543

  ```java
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  ...
  ```

  ```java
  package io.namjune.springbootconceptandutilization.sample;
  
  import static org.mockito.Mockito.when;
  
  import org.junit.Test;
  import org.junit.runner.RunWith;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
  import org.springframework.boot.test.mock.mockito.MockBean;
  import org.springframework.test.context.junit4.SpringRunner;
  import org.springframework.test.web.reactive.server.WebTestClient;
  
  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
  public class SampleControllerTest {
  
      @Autowired
      WebTestClient webTestClient;
  
      @MockBean
      SampleService mockSampleService;
  
      @Test
      public void hello() {
          when(mockSampleService.getName()).thenReturn("kim");
  
          webTestClient.get().uri("/hello").exchange()
              .expectStatus().isOk()
              .expectBody(String.class).isEqualTo("hello kim");
  
      }
  }
  ```

* NONE: 서블릿 환경 제공 안 함.

### @MockBean

* 위의 경우 테스트가 너무 크다. Controller 테스트코드에서 Service단까지 흘러간다. 컨트롤러만 테스트하고 싶을 경우 서비스 객체를 MockBean으로 만들어서 사용할 수 있다.
* ApplicationContext에 들어있는 빈을 Mock으로 만든 객체로 교체함
* 모든 @Test 마다 자동으로 리셋. 직접 리셋을 관리 하지 않아도 된다.

```java
package io.namjune.springbootconceptandutilization.sample;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.Mockito.when;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SampleControllerTest {

    @Autowired
    TestRestTemplate testRestTemplate;

    @MockBean
    SampleService mockSampleService;

    @Test
    public void hello() {
        when(mockSampleService.getName()).thenReturn("kim");

        String result = testRestTemplate.getForObject("/hello", String.class);
        assertThat(result).isEqualTo("hello kim");
    }
}
```

### 슬라이스 테스트

* 레이어 별로 잘라서 테스트 하고 싶을 때
* @JsonTest
  * json 테스트를 하고싶을 경우 @SpringBootTest 대신 @JsonTest를 선언하고, JacksonTester를 주입받아서 사용하면 된다.
* @WebMvcTest
  * 컨트롤러만 따로 테스트 할 경우 사용한다. 웹과 관련된 클래스들만 빈으로 등록이 되고 일반적인 컴포넌트들은 빈으로 등록이 되지 않는다. 이렇게 의존성이 끊기기 때문에, 사용하는 다른 의존성 예를 들면 서비스와 같은 객체들은@MockBean을 사용해서 만들어 사용해야 한다. 
  * 커밋로그 : https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/76a0e095576f0b52b23f2cec3f60f050d1bb2042
* @WebFluxTest
* @DataJpaTest
* ...

## 6-2. 테스트 유틸

* 스프링 테스트가 제공하는 테스트 유틸리티가 4가지 있다.
  * **OutputCapture**
  * TestPropertyValues
  * TestRestTemplate
  * ConfigFileApplicationContextInitializer
* Junit에 있는 Rule을 확장해서 만든 OutputCapture가 제일 많이 쓰인다.
  * OutputCapture는 로그를 비롯해서 콘솔에 찍히는 모든 것을 캡쳐한다.
    * 로그 메세지가 어떻게 찍히는지 테스트할 수 있다.
  * @Rule을 선언하고,
  * Junit이 제공하는 OutputCapture를 public으로 만든다.(@Rule의 제약사항. 빈을 주입받는게 아님)

```java
@RestController
@RequiredArgsConstructor
public class SampleController {

    Logger logger = LoggerFactory.getLogger(SampleController.class);

    private final SampleService sampleService;

    @GetMapping("/hello")
    public String hello() {
        logger.info("hello logger");
        System.out.println("hello sout");
        return "hello " + sampleService.getName();
    }
}
```

```java
@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class SampleControllerTest {

    @Rule
    public OutputCapture outputCapture = new OutputCapture();

    @MockBean
    SampleService mockSampleService;

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        when(mockSampleService.getName()).thenReturn("kim");

        mockMvc.perform(get("/hello"))
            .andExpect(content().string("hello kim"));

        assertThat(outputCapture.toString())
            .contains("hello")
            .contains("sout");
    }
}
```

