# 스프링 프레임워크 핵심 기술

> '기본에 충실하라'를 지키려고 노력중

## 스프링 소개

* 소규모 애플리케잉션 또는 기업용 애플리케이션을 자바로 개발하는데 있어 유용하고 편리한 기능을 제공하는 프레임워크
* 스프링 역사
    * 2003년 등장(개발은 이미 그 이전부터)
        * 등장시 Java EE  표준과 싸우는 것처럼 보였지만 실제로는 JEE 표준 스펙 구현 모음체(+알파)
        * 최근까지 주로 서블릿 기반 애플리케이션을 만들 때 사용해 옴
        * 스프링 5부터는 WebFlux 지원으로 서블릿 기반이 아닌 서버 애플리케이션도 개발할 수 있게 됨
* 디자인 철학
    * 모든 선택은 개발자의 몫(ex. 스프링이 특정 영속화 기술을 강요하지 않음)
    * 다양한 관점을 지향한다(유연성)
    * 하위 호환성을 지킨다(노력)
    * API를 신중하게 설계 한다(공들인다)
    * 높은 수준의 코드를 지향한다(자랑)

## IoC 컨테이너와 빈

### 1. 스프링 IoC 컨테이너와 빈

* **Inversion of Control**
    * 의존 관계 주입(Dependency Injection)이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말함.
* **스프링 IoC 컨테이너**
    * **BeanFactory**
        * 스프링 프레임워크의 중요한 인터페이스
    * 애플리케이션 컴포넌트의 중앙 저장소
    * 빈 설정 소스로부터 빈 정의를 읽어들이고, 빈을 구성하고 제공한다.
* **Bean - 빈**
    * 스프링 IoC 컨테이너가 관리 하는 객체
    * 장점
        * 의존성 관리
            * 의존성 관리를 할 수 없다고 생각해보자
            * 예를 들어, 서비스 객체 내에서 Repository 의존성을 직접 만들어서 사용하는 경우, 서비스 단의 단위 테스트를 독립적으로 작성하기가 힘들다.
            * 의존성을 주입 받을 수 있기 때문에 객체(Repository)를 Mocking 해서 얼마든지 단위 테스트를 할 수 있다.
        * 스코프
            * 싱글톤: 하나
                * 빈의 기본 스코프 설정은 싱글톤
                * 메모리 관리 등 성능 최적화 이점
            * 프로토타입: 매번 다른 객체 생성
        * 라이프 사이클 인터페이스
            * @PostConstruct 등
* **ApplicationContext**
    * ApplicationContext는 spring-context 모듈 안에 위치함
    * spring-beans 모듈의 **BeanFactory**(스프링 IoC 컨테이너)를 상속받고, 그외 다양한 기능을 가지는 인터페이스도 추가로 상속받은 인터페이스
        * **ListableBeanFactory**
        * **HierarchicalBeanFactory**
        * **EnvironmentCapable**
            * spring-core 모듈
            * 프로파일과 프로퍼티를 다루는 인터페이스
        * **MessageSource**
            * spring-context 모듈
            * 메세지 소스 처리 기능(i18n)
        * **ApplicationEventPublisher**
            * spring-context 모듈
            * 이벤트 발행 기능
        * **ResourcePatternResolver**
            * spring-core 모듈
            * **ResourceLoader**
            * 리소스 로딩 기능 등

### 2. ApplicationContext와 다양한 빈 설정 방법

* 스프링 IoC 컨테이너의 역할

    * 빈 인스턴스 생성
    * 의존 관계 설정
    * 빈 제공

* **ClassPathXmlApplicationContext**로 빈과 의존관계 등을 **XML 파일로 일일히 설정하는 방법**이 있었고, 이 굉장한 번거로움을 피하고자 등장한 것이 **component-scan** 이다.

    * 특정 패키지 하위에 있는 빈이 될 클래스들을 모두 스캔해서 빈으로 등록하는 방법이다.
    * 빈 스캐닝시 기본적으로 @Component 어노테이션을 사용해서 빈으로 등록할 수 있고,
    * @Component를 확장한 @Controller, @Service, @Repository 모두 컴포넌트 스캔의 대상이 된다.

* **Java Config**로 직접 빈 설정도 사용할 수 있다.

    * **AnnotationConfigApplicationContext**

        ```java
        @Configuration
        public class ApplicationConfig{
            
            @Bean
            public BookRepository bookRepository() {
                return new BookRepository();
            }
            
            @Bean
            public BookService bookService() {
                BookService bookService() = new BookService();
                bookService.setBookRepository(boookRepository());
                return bookService; 
            }
        }
        ```

* 사실, Java Config로 일일히 하는 것도 번거롭다. 그래서 여기서도 @ComponentScan을 사용할 수 있다 :)

    * ComponentScan 대상 지정을 basePackages와 basePackageClasses 두가지 방법으로 할 수 있지만, 더 타입세이프 한 방법은 클래스 기준으로 가져가는 것이다.

    * **이 방법이 Spring Boot를 기반으로 Spring을 사용하는 것과 가장 가까운 방법이다.**

        ```java
        @Configuration
        @ComponentScan(basePackageClasses = DemoApplication.class)
        public class ApplicationConfig {
        
        }
        ```

        ```java
        public class DemoApplication {
            public static void main(String[] args) {
                ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
                
                ...
                    
            }
        }
        ```

    * 물론 스프링 부트에서는 ApplicationContext를 만드는 것 조차 @SpringBootApplication이 해준다.

        * @SpringBootApplication을 타고 들어가보면, @SpringBootConfiguration이 @Configuration을 가지고 있고, @ComponentScan이 선언되어있다.
        * 따라서 Java Config 설정때 만들었던 ApplicationConfig도 필요 없다.
        * @SpringBootApplication이 다 해준다.

        ```java
        @SpringBootApplication
        public class DemoApplication {
            public static void main(String[] args) {
                    
            }
        }
        ```

### 3. @Autowired

* 필요한 의존 객체의 "타입"에 해당하는 빈을 찾아 주입한다.

* **@Autowired**

  * required: 기본값은 true(따라서 못 찾으면 애플리케이션 구동 실패)

* **사용할 수 있는 위치**

  * 생성자(스프링 4.3 부터는 생략 가능)
  * 세터
  * 필드

* **경우의 수**

  * 해당 타입의 빈이 없는 경우
  * 해당 타입의 빈이 한 개인 경우
  * 해당 타입의 빈이 여러 개인 경우
    * 빈 이름으로 시도
      * 같은 이름의 빈을 찾으면 해당 빈 사용
      * 같은 이름 못찾으면 실패

* **같은 타입의 빈이 여러개 일 때**

  * **@Primary**
    * 같은 타입의 빈이 여러개일 때, 필드의 이름을 변경해서 주입 받는 것 보다 @Primary를 이용하는 것을 권장함.
  * 해당 타입의 빈 모두 주입 받기
    * List 형태로 해당 타입의 빈을 모두 주입 받을 수 있음
  * @Qualifier(빈 이름으로 주입)

* **@Autowired의 동작 원리**

  * 빈 Initialization 라이프 사이클(@PostConstruct 등) 이전 또는 이후에 부가적인 작업을 할 수 있는 **BeanPostProcessor라는 라이프 사이클 콜백**이 존재한다.
    * 아래 document의 11번, 14번 순서
      * BeanPostProcessor의 postProcessBeforeInitialization() 메서드
      * BeanPostProcessor의 postProcessAfterInitialization() 메서드
    *  Initialization 라이프 사이클인 @PostConstruct는 12번 InitializingBean's 에서 이루어짐
  * **BeanPostProcessor**
    * 새로 만든 빈 인스턴스를 수정할 수 있는 라이프 사이클 인터페이스
  * AutowiredAnnotationBeanPostProcessor extends BeanPostProcessor
    * **@Autowired 동작 시점**
      * **결국 빈이 Initialization 하기 전에(postProcessBeforeInitialization()) BeanPostProcessor를 상속받은 AutowiredAnnotationBeanPostProcessor에 의해서 @Autowired를 처리한다.**
      * 아래 spring document의 표준 빈 라이프 사이클 메서드 중 11번째에 해당
    * 스프링이 제공하는 @Autowired와 @Value 애노테이션 그리고 JSR-330의 @Inject 어노테이션을 지원하는 어노테이션 처리기

* **그러면 BeanPostProcessor는 어떻게 동작하나?**

  * BeanFactory(이것은 결국 스프링이 제공하는 ApplicationContext)가 BeanPostProcessor 타입의 빈들을 찾는다.

  * 그중에 AutowiredAnnotationBeanPostProcessor 빈이 등록되어 있을 것이고,

  * 일반적인 빈들에게 AutowiredAnnotationBeanPostProcessor에 있는 어노테이션을 적용하는 로직을 수행하게 된다.

  * 궁금하면 한번 찾아보자!

    ```java
    @Component
    public class MyRunner implements ApplicationRunner {
      
      @Autowired
      ApplicationContext applicationContext;
      
      @Override
      public void run(ApplicationArguments args) throws Exception {
        AutowiredAnnotationBeanPostProcessor bean =
          applicationContext.getBean(AutowiredAnnotationBeanPostProcessor.class)
          System.out.println(bean);
      }
    }
    ```

* **spring.io에서 제공하는 document - 표준 빈 라이프 사이클 초기화 메서드 순서**

  ```
  Bean factory implementations should support the standard bean lifecycle interfaces as far as possible. The full set of initialization methods and their standard order is:
  
  1. BeanNameAware's setBeanName
  2. BeanClassLoaderAware's setBeanClassLoader
  3. BeanFactoryAware's setBeanFactory
  4. EnvironmentAware's setEnvironment
  5. EmbeddedValueResolverAware's setEmbeddedValueResolver
  6. ResourceLoaderAware's setResourceLoader (only applicable when running in an application context)
  7. ApplicationEventPublisherAware's setApplicationEventPublisher (only applicable when running in 8. an application context)
  9. MessageSourceAware's setMessageSource (only applicable when running in an application context)
  ApplicationContextAware's setApplicationContext (only applicable when running in an application context)
  10. ServletContextAware's setServletContext (only applicable when running in a web application context)
  11. postProcessBeforeInitialization methods of BeanPostProcessors
  12. InitializingBean's afterPropertiesSet
  13. a custom init-method definition
  14. postProcessAfterInitialization methods of BeanPostProcessors
  
  On shutdown of a bean factory, the following lifecycle methods apply:
  
  1. ostProcessBeforeDestruction methods of DestructionAwareBeanPostProcessors
  2. DisposableBean's destroy
  3. a custom destroy-method definition
  ```

### 4. @Component와 컴포넌트 스캔

* 컴포넌트 스캔의 주요 기능

  * 스캔할 위치를 선정한다.
  * 필터 : 어떤 애노테이션을 스캔 할지 또는 제외 할지.

* 컴포넌트 스캔의 대상들

  * @Component
    * 컴포넌트를 확장한 어노테이션들
    * @Repository
    * @Service
    * @Controller
    * @Configuration
    * 등

* 컴포넌트 스캔의 동작 원리

  * @ComponentScan은 스캔할 패키지(basePackages)와 애노테이션에 대한 정보를 가지고 있다.
  * **실제 스캐닝**은 BeanFactoryPostProcessor를 확장한 **ConfigurationClassPostProcessor**에 의해서 처리되는데, 이 시점은 3. @Autowired 에서 설명한 **표준 빈 라이프사이클 초기화 메서드 단계 이전이다.**
  * 다른 모든 빈들이 만들어지기 이전에 동작한다. 다른 빈이라는 것은 @Bean으로 **직접** 등록하거나, Application을 run 하면서 **직접** functional하게 컴포넌트 스캔 해당 패키지 바깥에 있는 빈등을 말한다. 핵심은 직접!
  * 직접 만드는 빈들이 만들어 지기 이전에 컴포넌트 스캔으로 빈들을 등록해준다.

* Functional 하게 빈 등록하는 방법

  * 스프링 5부터는 application initialize 시점에 컴포넌트 스캔 패키지 외부의 빈을 직접 등록할 수 있다.

  * 이 방법은 리플렉션과 프록시를 사용하지 않기 때문에, 초기 구동시 모든 빈이 등록할 때 드는 비용(성능)상의 이점을 가져갈 수 있다.

  * 하지만, 이것 때문에 컴포넌트 스캔 대신 Functional 하게 모든 빈을 일일히 등록하는 건 좀 불편하지 않을까..

    ```java
    public static void main(String[] args) {
      new SpringApplicationBuilder(Application.class)
        .sources(Application.class)
        .initializers((ApplicationContextInitializer<GenericApplicationContext>) ctx -> {
          ctx.registerBean(MyBean.class);
        })
        .run(args);
    }
    ```

### 5. 빈의 스코프

* ApplicationContext에 생성된 빈들은 스코프를 가지고 있다. 아무 설정도 해주지 않았으면. default인 싱글톤 스코프를 가진다.

* 스코프

  * 싱글톤
    * 애플리케이션 전반에 걸쳐서, 해당 빈의 인스턴스가 오직 한개
  * 프로토타입
    * 빈을 주입 받아올 때마다 새로운 빈이 생성된다.

* 프로토타입 빈이 싱글톤 빈을 참조하면?

  * 아무 문제 없다.

* **싱글톤 빈이 프로토타입 빈을 참조하면?**

  * 프로토타입 빈이 업데이트가 안되네...???

    * 싱글톤 빈을 통해서 프로토타입 빈을 조회하면 동일한 빈이 참조 된단 이야기.

  * 업데이트 하려면, 여러가지 방법이 있지만 그중 하나. scoped-proxy를 쓰는 방법

    * **scoped-proxy**

      * 빈 선언부에 스코프 설정과 같이 함

        ```java
        @Bean
        @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
        public class Proto {
          
        }
        ```

      * proxyMode를 설정해야함. default는 프록시를 사용하지 않는다는 옵션.

      * 빈에 적용해야하므로 ScopedProxyMode.TARGET_CLASS를 프록시 모드로 설정한다.

      * 이 설정은, 클래스기반의 프록시로 감싸서 빈으로 등록하고, 다른 빈들이 사용할 때 프록시로 감싼 빈을 사용하게 하라는 설정이다.

      * 이렇게 하지 않으면, 프로토타입 빈을 바꿔줄 여지가 없다.

      * 싱글톤 빈에서 프로토타입 빈을 사용하려면 이와 같은 설정을 하고, 싱글톤 빈이 프록시로 감싸진 빈을 참조하게 해서 매번 다른 빈을 참조하게 해야한다.

    * Object-Provider

      * 소스 코드에 침투함.

    * Provider(표준)

* 싱글톤 객체 사용시 주의할 점

  * 필드가 공유된다.
  * 멀티스레드 환경에서 빈의 필드를 변경하려고 하기때문에 스레드 세이프한 코딩이 필수적이다.
  * 모든 싱글톤 빈들은 ApplicationContext 초기 구동시 인스턴스가 생성된다.

### 6. Environment 1부. 프로파일

* ApplicationContext는 BeanFactory 기능 외에도, 여러가지 기능을 가지고 있다. 그중 하나가 프로파일과 프로퍼티를 다루는 것인데 ApplicationContext가 EnvironmentCapable 인터페이스를 상속 받았기 때문에 그 기능을 사용할 수 있다.

* 프로파일

  * 빈들의 그룹

  * **Environment**의 역할은 활성화 할 프로파일 확인 및 설정하는 것이다.

    ```java
    @Autowired
    ApplicationContext ctx;
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
      Environment environment = ctx.getEnvironment();
      System.out.println(Arrays.toString(environment.getActiveProfiles()));  // 기본은 비어있음
      System.out.println(Arrays.toString(environment.getDefaultProfiles()));  // [default]
    }
    ```

* 프로파일 유스케이스

  * 테스트 환경에서는 A라는 빈을, 배포환경에서는 B라는 빈을 쓰고 싶다.
  * 이 빈은 모니터링 용도니까 테스트할 때는 필요없고, 배포할 때만 등록이 되면 좋겠다.

* 프로파일 정의하기

  * 아래의 설정 파일은 테스트라는 프로파일에서만 사용되는 빈설정 파일이다.

  * 클래스에 정의

    ```java
    @Configuration
    @Profiles("test")
    public class TestConfiguration {
      
      @Bean
      public BookRepository bookRepository() {
        return new TestBookRepository();
      }
    }
    ```

    ```java
    @Repository
    @Profiles("test")
    public class TestBookRepository implements BookRepository {
    }
    ```

  * 메소드에 정의

    ```java
    @Configuration
    public class TestConfiguration {
      
      @Bean
      @Profiles("test")
      public BookRepository bookRepository() {
        return new TestBookRepository();
      }
    }
    ```

* 프로파일 설정하기

  * 인텔리제이에서는 Active Profiles에 추가 또는, VM options에 `-Dspring.profiles.active="test"`를 추가하면 된다.
  * 또는 jar파일 실행시 VM 옵션을 주면 된다. `$ java -Dspring.profiles.active="test" -jar [파일이름].jar`
  * 테스트 할 경우에는 
    * 테스트 코드 실행시 특정 환경을 맞춰야 한다면 **@ActiveProfiles** 를 사용하면 된다.

* 프로파일 표현식

  * ! (not)

  * & (and)

  * | (or)

  * 사용 예

    * prod환경이 아닌 경우에만 빈을 등록

      ```java
      @Repository
      @Profiles("!prod")
      public class TestBookRepository implements BookRepository {
      }
      ```

### 7. Environment 2부. 프로퍼티

* 프로퍼티

  * 어플리케이션에 등록되어 있는 여러가지 키-밸류 쌍으로 정의되어 있는 정보
  * 다양한 방법으로 정의할 수 있는 설정값
  * 여기서 Environment의 역할은 프로퍼티 소스 설정 및 프로퍼티 값 가져오기

* 만약 어플리케이션 실행시 VM options 으로 `-Dapp.name=spring5` 를 줬다면, 어플리케이션에서는 아래와 같이 접근이 가능하다.

  ```java
  @Autowired
  ApplicationContext ctx;
  
  @Override
  public void run(ApplicationArguments args) throws Exception {
    Environment environment = ctx.getEnvironment();
    environment.getProperty("app.name"); // 프로퍼티 키 값으로 접근
  }
  ```

* 프로퍼티 파일을 만들고, 그 파일을 참조해서 프로퍼티 값을 참조하는 것도 가능하다.

  * resources/app.properties

    ```java
    app.about=test
    ```

  * Application.java

    * **@PropertySource 활용**

      ```java
      @SpringBootApplication
      @PropertySource("classpath:/app.properties")
      public class Application {
        public static void main(String[] args) {
          
        }
      }
      ```

  * AppRunner.java

    * Environment를 통해서 프로퍼티 값 가져오기

      ```java
      @Autowired
      ApplicationContext ctx;
      
      @Override
      public void run(ApplicationArguments args) throws Exception {
        Environment environment = ctx.getEnvironment();
        environment.getProperty("app.about"); // 프로퍼티 키 값으로 접근 test 출력
      }
      ```

* **프로퍼티에는 우선 순위가 있다.** 계층형(hierarchical)으로 접근한다는 말이 우선순위에 따라서 값이 결정 된다는 이야기이다.
  * 같은 옵션을 JVM 시스템 프로퍼티로 줄 때와 프로퍼티 소스(.properties 파일)로 넣은게 높을까?
    * JVM 옵션으로 줄때가 높다.
  * **StandardServletEnvironment의 우선순위**
    * ServletConfig 매개변수
    * ServletContext 매개변수
    * JNDI (java:comp/env/)
    * JVM 시스템 프로퍼티 (-Dkey="value")
    * JVM 시스템 환경 변수(운영 체제 환경 변수)
* 스프링 부트의 외부 설정 참고
  * 스프링 부트에서는 프로퍼티를 더 쉽게 쓸 수 있게 지원해 준다.
  * 프로파일 까지 고려한 계층형 프로퍼티 우선 순위를 제공한다.
  * **스프링 부트의 외부 설정과 프로퍼티에 관한 내용 정리**
    * [https://github.com/namjunemy/TIL/blob/master/SpringBoot/03_springboot_utilization.md#3-1-%EC%99%B8%EB%B6%80%EC%84%A4%EC%A0%95-1%EB%B6%80](https://github.com/namjunemy/TIL/blob/master/SpringBoot/03_springboot_utilization.md#3-1-외부설정-1부)

### 8. MessageSource

* 국제화(i18n) 기능을 제공하는 인터페이스가 ApplicationContext에 들어 있다.

* 메세지를 다국화 하는 방법

* ApplicationContext를 타고 들어가 보면, MessageSource를 구현하고 있다. 따라서, @Autowired로 ApplicationContext를 주입받을 수 있다면. MessageSource를 직접 주입 받을수도 있다.

* MessageSource가 제공하는 인터페이스를 직접 사용할 수 있다.

* 스프링 부트를 사용한다면, 별다른 설정을 하지 않아도messages로 시작하는 프로퍼티 파일들을 메세지 소스로 다 읽어준다.

    * 아래에서 찍어보면 알겠지만, 부트에 등록되어있는 ResourceBundleMessageSource빈이 리소스 번들을 다 읽어서 등록해준다.

    * `messages.properties`

        ```properties
        greeting = hello, {0}
        ```

    * `messages_ko_KR.properties`

        ```properties
        greeting = 안녕, {0}
        ```

    * AppRunner.java

        ```java
        @Component
        public class AppRunner implements ApplicationRunner {
            
            @Autowired
            MessageSource messageSource;
            
            @Override
            public void run(ApplicationArguments args) throws Exception {
                System.out.println(messageSource.getClass());        
                System.out.println(messageSource.getMessage("greeting", new String[]{"NJKIM"}, Locale.getDefault()));
                System.out.println(messageSource.getMessage("greeting", new String[]{"NJKIM"}, Locale.KOREA));
            }
        }
        ```

        ```text
        class.org.springframework.context.support.ResourceBundleMessageSource
        안녕, NJKIM
        hello, NJKIM
        ```

* 리로딩 기능이 있는 메세지 소스를 사용할 수도 있다.

    * 언제나 우리는 스프링부트가 제공하는 기능을 커스텀해서 쓸 수 있다.

    * 메세지 소스의 캐시타임을 3초로 주면, 3초 주기로 리로드 하게된다.

    * 빌드된 소스를 읽어서 동작하기 때문에, ide에서 빌드를 해줘야 확인가능하다.

        ```java
        @Bean
        public MessageSource messageSource() {
            ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
            messageSource.setBasename("classpath:/messages");
            messageSource.setDefaultEncofing("UTF-8");
            messageSource.setCacheSeconds(3);
            return messageSource;
        }
        ```

### 9. ApplicationEventPublisher

* 이어서 ApplicationContext가 상속 받고 있는 또하나의 인터페이스 ApplicationEventPublisher는 옵저버 패턴의 구현체로 이벤트 프로그래밍에 필요한 인터페이스를 제공한다.

* 이벤트를 반드는 방법

  * ApplicationEvent를 상속 받아서 구현**했었다**.

    ```java
    public class MyEvent extends ApplicationEvent {
      
      private int data;
      
      public MyEvent(Object source, int data) {
        super(source);
        this.data = data;
      }
      
      public int getData() {
        return data;
      }
    }
    ```

  * **스프링 4.2부터는 이 클래스를 상속 받지 않아도 이벤트로 사용할 수 있다.**

    * 이것이 스프링 프레임워크가 추구하는 철학인 비침투성이다. POJO
    * 코드에서 보면 spring 패키지에서 import가 하나도 없다.
    * spring 소스 코드가 개발자의 소스코드에 들어가지 않는다. 노출되지 않는다.
    * 이렇게 POJO기반의 프로그래밍을 할 때, 테스트 코드 작성이 더 쉬워지고, 유지보수 하기 쉬워진다.

    ```java
    public class MyEvent {
      
      private Object source;
      private int data;
      
      public MyEvent(Object source, int data) {
        this.object = object;
        this.data = data;
      }
      
      public Object getSource() {
        return source;
      }
      
      public int getData() {
        return data;
      }
    }
    ```

* 이벤트를 발생 시키는 방법

  * ApplicationContext가 ApplicationEventPublisher를 상속받고 있으므로 applicationContext에서 직접 publishEvent()를 호출 할 수도 있고.

  * ApplicationEventPublisher를 Injection받아서

  * ApplicationEbentPublisher.publishEvent()로 발생시킬 수 있다.

    ```java
    @Component
    public class TestRunner implements ApplicationRunner {
      
      @AutoWired
      ApplicationEventPublisher eventPublisher;
      
      @Override
      public void run(ApplicationArguments args) throws Exception {
        eventPublisher.publishEvent(new MyEvent(this, 100));
      }
    }
    ```

* 이벤트를 처리하는 방법

  * ApplicationListener<이벤트> 구현한 클래스를 만들어서 **빈으로 등록** 해야 했었다.

    ```java
    @Component
    public class MyEventHandler implements ApplicationListener<MyEvent> {
      
      @Override
      publiv void onApplicationEvent(MyEvent event) {
        System.out.println("이벤트 받음, 데이터는 " + event.getData());
      }
    }
    ```

  * **마찬가지로 스프링 4.2 부터는 리스터를 구현하지 않아도 된다.**

    * 빈으로는 등록 되어야 한다. 그래야 스프링이 누구한테 전달해야 하는지 알 수 있다. 핸들러는 항상 빈으로.
    * **@EventListener** 를 사용해서 빈의 메소드에 사용할 수 있다.

    ```java
    @Component
    public class MyEventHandler {
      
      @EventListener
      publiv void handleMyEvent(MyEvent event) {
        System.out.println("이벤트 받음, 데이터는 " + event.getData());
      }
    }
    ```

  * 기본적으로 이벤트 처리는 동기. Syschronized 방식이다. 순서를 보장하지 않는다.

  * 순서를 정하고 싶다면 @Order와 함께 사용하고,

  * 비동기적으로 실행하고 싶다면 @Async와 함께 사용하면 된다.

    * @Async를 사용하려면 @EnableAsync와 함께 사용해야 하고, 스레드풀 관련 설정을 해야 한다.

* 그 어떠한 스프링 코드도 노출되어 있지 않는 POJO 기반의 프로그래밍으로 이벤트 기반 프로그래밍을 하자.

* **스프링이 제공하는 기본 이벤트**

  * 스프링 부트는 스프링의 이벤트를 확장해서 다양한 이벤트를 제공해준다.
    *  https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-application-events-and-listeners
  * ContextRefreshedEvent
    * ApplicationContext를 초기화 했거나, 리프레시 했을 때 발생
  * ContextStartedEvent
    * ApplicationContext를 start()하여 라이프사이클 빈들이 시작 신호를 받은 시점에 발생
  * ContextStoppedEvent
    * ApplicationContext를 stop()하여 라이프사이클 빈들이 정지 신호를 받은 시점에 발생
  * ContextClosedEvent
    * ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에 발생
  * RequestHandledEvent
    * HTTP 요청을 처리했을 때 발생

### Reference

* https://spring.io/projects/spring-framework
* 스프링 프레임워크 핵심 기술 - 백기선