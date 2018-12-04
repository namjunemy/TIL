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



