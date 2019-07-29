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
    * BeanFactory
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
    * spring-beans 모듈의 BeanFactory를 상속받고, 그외 다양한 기능을 가지는 인터페이스도 추가로 상속받은 인터페이스
        * ListableBeanFactory
        * HierarchicalBeanFactory
    * EnvironmentCapable
        * spring-core 모듈
    * MessageSource
        * spring-context 모듈
        * 메세지 소스 처리 기능(i18n)
    * ApplicationEventPublisher
        * spring-context 모듈
        * 이벤트 발행 기능
    * ResourcePatternResolver
        * spring-core 모듈
        * ResourceLoader
        * 리소스 로딩 기능 등

### 2. ApplicationContext와 다양한 빈 설정 방법

* 스프링 IoC 컨테이너의 역할

    * 빈 인스턴스 생성
    * 의존 관계 설정
    * 빈 제공

* **ClassPathXmlApplicationContext**로 빈과 의존관계 등을 **XML 파일로 일일히 설정하는 방법**이 있었고, 이 굉장한 번거로움을 피하고자 등장한 것이 **component-scan** 이다.

    * 특정 패키지 하위에 있는 빈이 될 클래스들을 모두 스캔해서 빈으로 등록하는 방법이다.
    * 빈 스캐닝시 기본적으로 @Component 어노테이션을 사용해서 빈으로 등록할 수 있고,
    * @Component를 확장받은 @Controller, @Service, @Repository 모두 컴포넌트 스캔의 대상이 된다.

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

## Reference

* https://spring.io/projects/spring-framework
* 스프링 프레임워크 핵심 기술 - 백기선