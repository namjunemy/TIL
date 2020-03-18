# Spring 5 recipes

> 스프링 5 레시피 - 마틴 데니엄, 다니엘 루비오, 조시 롱 지음 참조.
>
> 스프링 애플리케이션 개발에 유용한 161가지 문제 해결 기법

# 스프링 코어

## 2-1. 자바로 POJO 구성하기

### 구성 클래스에서 @Configuration과 @Bean을 붙여 POJO 생성

```java
@Configuration
public class SequesceGeneratorConfiguration {
  
  @Bean
  public SequenceGenerator sequenceGenerator() {
    
  }
}
```

* @Bean(name="myBean1") 과 같이 name을 설정하면 빈 생성시 메서드 명은 무시

### IoC 컨테이너를 초기화하여 애너테이션 스캐닝하기

* 애너테이션을 붙인 자바 클래스를 스캐닝하려면 우선 IoC 컨테이너를 인스턴스화 해야 한다. 그래야 스프링이 나중에 @Configuration과 @Bean등을 발견하고 IoC 컨테이너에서 빈 인스턴스를 가져올 수 있다.

* 스프링은 기본 구현체인 Bean Factory와 이와 호환되는 고급 구현체인 Application Context, 두 가지 IoC 컨테이너를 제공한다.

* 애플리케이션 컨텍스트는 기본 기능에 충실하면서도 빈 팩토리보다 발전된 기능을 지니고 있으므로 리소스에 제약을 받는 상황이 아니면 가급적 애플리케이션 컨텍스트를 사용하는게 좋다. 

* BeanFactory와 ApplicationContext는 각각 빈 팩토리와 애플리케이션 컨텍스트에 접근하는 인터페이스이다. ApplicationContext는 BeanFactory의 하위 인터페이스여서 호환성이 보장된다.

* ApplicationContext는 인터네이스이기 때문에 사용하려면 구현체가 필요하다. 스프링은 몇가지 애플리케이션 컨텍스트 구현체를 마련해 놓았다.

* 그중 가장 최근 작품이면서 유연한 AnnotationConfigApplicationContext를 권장한다.

  ```java
  ApplicationContext context = new AnnotationConfigApplicationContext(SequeceGeneratorConfiguration.class);
  ```

* 애플리케이션 컨텍스트를 인스턴스화 하면, 이후에 객체 레퍼런스(코드에서 context)는 POJO 인스턴스 또는 빈에 엑세스하는 창구 역할을 한다.

### IoC 컨테이너에서 POJO 인스턴스/빈 가져오기

* 구성 클래스에 선언된 빈을 빈 팩토리 또는 애플리케이션 컨텍스트에서 가져오려면 유일한 빈 이름을 getBean() 메서드의 인수로 호출해야 한다. 리턴타입은 Object 이므로 실제 타입에 맞게 캐스팅이 필요하다

  ```java
  SequenceGenerator generator = (SequenceGenerator) context.getBean("sequenceGenerator");
  ```

* 캐스팅 하지 않으려면, 빈 클래스명 지정.

  ```java
  SequenceGenerator generator = context.getBean("sequenceGenerator", SequenceGenerator.class);
  
  // 빈이 하나뿐이라면 빈의 이름 생략 가능
  SequenceGenerator generator = context.getBean(SequenceGenerator.class);
  ```

* 이런식으로 POJO 인스턴스/빈을 스프링 외부에서 생성자를 이용해 여느 객체처럼 사용할 수 있다.

  ```java
  public class Main {
    
    public static void main(String[] args) {
      ApplicationContext context = 
        new AnnotationConfigApplicationContext(SequenceGeneratorConfiguration.class);
      
      SequenceGenerator generator = context.getBean(SequesceGenerator.class);
      
      System.out.println(generator.getSequence());
      System.out.println(generator.getSequence());
    }
  }
  
  // return example
  3010000A
  3010001A
  ```

### POJO 클래스에 @Component를 붙여 빈 생성

* 클래스에 @Component("sequesceDao")를 붙이면 스프링은 이 클래스를 이용해서 POJO를 생성한다.

* @Component에 넣은 값은 빈 인스턴스의 ID로 설정하며, 값이 없으면 소문자로 시작하는 비 규격 클래스명을 빈 이름으로 기본 할당한다. 여기서는 sequenceDaoImpl

  ```java
  @Component("sequenceDao")
  public class SequenceDaoImpl implements SequenceDao {
    
    ...
  }
  ```

* @Component는 스프링이 발견할 수 있게 POJO에 붙이는 범용 에너테이션이다.

* 스프링에는 Persistence, Service, Presentation 각 영속화, 서비스, 표현 세가지 계층이 있는데,

* @Repository, @Service, @Controller가 이 세 계층을 가리키는 에너테이션이다.

* **POJO의 쓰임새가 명확하지 않을 때는 그냥 @Component를 붙여도 되지만 특정 용도에 맞는 부가 혜택을 누리려면 구체적으로 명시하는 편이 좋다.**

  * **예를 들면, @Repository는 발생한 예외를 DataAccessException으로 감싸 던지기므로 디버깅시 유리하다.**

### 애너테이션을 스캐닝하는 필터로 IoC 컨테이너 초기화하기

* 기본적으로 스프링은 @Configuration, @Bean, @Component, @Repository, @Service, @Controller가 달린 클래스를 모두 감지한다.

* 이때 하나 이상의 포함/제외 필터를 적용해서 스캐닝 과정을 커스터마이징 할 수 있다.

* 스프링에서 지원하는 필터 표현식은 네 종류이다.

  * annotation, assignable은 각각 애너테이션 타입 및 클래스/인터페이스를 지정하고,
  * regex, aspectj는 각각 정규표현식과 AspectJ 포인트컷 표현식으로 클래스를 매치하는 용도로 쓰인다.

* use-default-filters 속성으로 기본 필터를 해제할 수도 있다.

* 다음과 같이 선언하면 io.namjune.speingrecipes.sequence 패키지에 속한 클래스 중 이름이 Dao나 Service가 포함된 것들은 모두 넣고, @Controller를 붙인 클래스는 스캐닝 대상에서 제외한다.

  ```java
  @ComponentScan(
  	includeFilters = {
      @ComponentScan.Filter(
      	type = FilterType.REGEX,
        pattern = {"io.namjune.speingrecipes.sequence.*Dao",
                   "io.namjune.speingrecipes.sequence.*Service"})
    },
    includeFilters = {
      @ComponentScan.Filter(
      	type = FilterType.ANNOTATION
        classes = {org.springframework.stereotype.Controller.class}})
    }
  )
  ```

## 2-2. 생성자 호출해서 POJO 생성하기

### 자바 구성 클래스 작성하기

* IoC 컨테이너에 POJO 인스턴스를 정의하려면 자바 구성 클래스를 작성해 값을 초기화한다.

* 생성자로 POJO 인스턴스/빈을 생성하는 자바 구성 클래스를 작성해보자

  ```java
  @Configuration
  public class ShopConfiguration {
    
    @Bean
    public Product aaa() {
      Battery p1 = new Battery("AAA", 2.5);
      p1.setRechargeable(true);
      return p1;
    }
    
    @Bean
    public Product bbb() {
      Disc p2 = new Disc("CD-RW", 1.5);
      p2.setCapacity(700);
      return p2
    }
  }
  
  public class Main {
    
    public static void main(String[] args) throws Exception {
      
      ApplicationContext context = 
        new AnnotationConfigApplicationContext(ShopConfiguration.class);
      
      Product aaa = context.getBean("aaa", Product.class);
      Product bbb = context.getBean("bbb", Product.class);
      
      System.out.println(aaa);
      System.out.println(bbb);
    }
  }
  ```

## 2-3. POJO 레퍼런스와 자동 연결을 이용해 다른 POJO와 상호 작용하기

* 자바 구성 클래스에 정의된 POJO/빈 인스턴스들 사이의 참조 관계는 표준 자바 코드로도 맺어줄 수 있다.
* 필드, 세터메서드, 생성자, 또는 다른 아무 메서드에 @Autowired를 붙이면 POJO 레퍼런스를 자동으로 연결해 쓸 수 있다.

### 자바 구성 클래스에서 POJO 참조하기

* 자바 구성 클래스에 POJO 인스턴스를 정의하면, 모든게 자바 코드로 있어서 얼마든지 POJO를 참조할 수 있다.

* 아래 코드와같이 @Bean으로 선언된 DatePrefixGenerator는 관례상 메서드명과 동일한 datePrefixGenerator로 빈을 가져올 수 있지만, **빈 인스턴스화 코드 자체가 표준 자바 메서드로 구현되어 있으므로 그냥 자바 메서드를 호출하듯 불러 써도 된다.**

* datePrefixGenerator() 메서드를 호출해도 빈을 가져올 수 있다.

  ```java
  @Configuration
  public class SequesceConfiguration {
    
    @Bean
    public DatePrefixGenerator datePrefixGenerator() {
      DatePrefixGenerator dpg = new DatePrefixGenerator();
      dpg.setPattern("yyyyMMdd");
      return dpg;
    }
    
    @Bean
    public SequenceGenerator sequenceGenerator() {
      SequenceGenerator sequence = new SequenceGenerator();
      sequence.setInitial(100000);
      sequence.setSuffix("A");
      sequence.setPrefixGenerator(datePrefixGenerator());
      return sequence;
    }
  }
  ```

### POJO 필드에 @Autowired를 붙여 자동 연결

* 서비스 객체를 생성하는 서비스 클래스는 실제로 자주 쓰이는 베스트 프랙티스로, DAO를 직접 호출하는 대신 일종의 파사드(관문)를 두는 패턴이다.

* 서비스 객체는 내부적으로 DAO들과 연동한다.

  ```java
  @Component
  public class SequenceService {
    
    @Autowired
    private SequenceDao sequenceDao;
    
    ...
  }
  ```

* SequenceService 클래스는 @Component를 붙였기 때문에 스프링 빈으로 등록 된다. 빈 이름은 클래스명 그대로 sequenceService

* @Autowired가 있기 때문에 SequenceDao(인터페이스 타입) 타입의 빈이 이 프로퍼티에 자동 연결 된다.

* **배열형 프로퍼티에 @Autowired를 붙이면 스프링은 매치된 빈을 모두 찾아 자동 연결한다.**

  ```java
  public class SequenceGenerator {
    
    @Autowired
    private PrefixGenerator[] prefixGenerators;
    
    ...
  }
  ```

* **Type-safe 한 컬렉션에 @Autowired를 붙이면 스프링은 이 컬렉션과 타입 호환되는 빈을 모두 찾아서 자동 연결한다.**

  ```java
  public class SequenceGenerator {
    
    @Autowired
    private List<PrefixGenerator> prefixGenerators;
    
    ...
  }
  ```

* **Type-safe 한 Map에 @Autowired를 붙이면 스프링은 이 컬렉션과 타입 호환되는 빈을 모두 찾아서 빈 이름이 키인 맵에 추가한다.**

  ```java
  public class SequenceGenerator {
    
    @Autowired
    private Map<String, PrefixGenerator> prefixGenerators;
    
    ...
  }
  ```

### @Autowired로 POJO 메서드와 생성자를 자동 연결하기, 자동 연결을 선택적으로 제어하기

* @Autowired는 POJO 세터 메서드에서도 직접 적용할 수 있다. PrefixGeneretor와 타입 호환되는 빈이 연결된다.

  ```java
  public class SequenceGenerator {
    ...
      
    @Autowired
    public void setprefixGenerator(PrefixGeneretor prefixGenerator) {
      this.prefixGenerator = prefixGenerator;
    }
  }
  ```

* 스프링은 기본적으로 @Autowired를 붙인 필수 프로퍼티에 해당하는 빈을 찾지 못하면 예외를 던진다.

* 따라서, **선택적인 프로퍼티는 @Autowired의 required 속성값을 false로 지정해 예외처리 할 수있다.**

  ```java
  public class SequenceGenerator {
    ...
      
    @Autowired(required = false)
    public void setprefixGenerator(PrefixGeneretor prefixGenerator) {
      this.prefixGenerator = prefixGenerator;
    }
  }
  ```

* 생성자에도 @Autowired를 붙여 자동 연결 할 수 있다. 스프링은 생성자 인수가 몇개든 각 인수형과 호환되는 빈을 연결한다.

  * 스프링 4.3버전부터 생성자가 하나뿐인 클래스의 생성자는 자동 연결하는 것이 기본이므로 굳이 @Autowired를 붙이지 않아도 된다.

  ```java
  @Service
  public class SequenceService {
    
    private final SequenceDao sequenceDao;
    
    @Autowired
    public SequenceService(SequenceDao sequenceDao) {
      this.sequenceDao = sequenceDao;
    }
    
    ...
  }
  ```

### 애너테이션으로 모호한 자동 연결 명시하기

* 타입을 기준으로 자동 연결하면 IoC 컨테이너에 호환 타입이 여럿 존재하거나 프로퍼티가 (배열, 리스트, 맵 등) 그룹형이 아닐 경우 연결이 제대로 되지 않는다.
* 타입이 같은 빈이 여럿이라면 @Primary, @Qualifier로 해결할 수 있다.

### @Primary

* 스프링에서는 @Primary를 붙여 후보 빈을 명시한다. 여러 빈이 연결 대상일 때 특정한 빈에 우선권을 부여하는 방법이다.

  ```java
  @Component
  @Primary
  public class DatePrefixGenerator imprements prefixGenerator {
    
    ...
  }
  ```

### Qualifier로 모호한 자동 연결 명시하기

* @Qualifier에 이름을 주어 후보 빈을 명시할 수도 있다.

  ```java
  @Component
  public class SequenceGenerator {
    
    @Autowired
    @Qualifier("datePrefixGenerator")
    private PrefixGenerator prefixGenerator;
    
    ...
  }
  ```

* @Qualifier는 메서드 인수를 연결하는데도 쓰일 수 있다.

  ```java
  @Component
  public class SequenceGenerator {
    
    @Autowired
    public void myOwnCustomInjectionName(
      @Qualifier("datePrefixGenerator") PrefixGenerator prefixGenerator) {
      this.prefixGenerator = prefixGenerator;
    }
  }
  ```

### 여러 곳에 분산된 POJO 참조 문제 해결하기

* 스프링 부트가 아닌 프로젝트에서는 애플리케이션 규모가 커질수록 POJO 설정을 하나의 자바 구성 클래스에 담아두기 어렵기 때문에 보통 POJO 기능에 따라 여러 자바 구성 클래스로 나누어 관리한다.

* 근데, 자바 구성 클래스가 여럿 공존하면 상이한 클래스에 정의된 POJO를 자동 연결하거나 참조하는 일이 복잡해진다.

* 한가지 방법은 자바 구성클래스가 위치한 경로마다 애플리케이션 컨텍스트를 초기화하는 방법이다.

* 각 자바 구성 클래서에 선언된 POJO를 컨텍스트와 레퍼런스로 읽으면, POJO간 자동연결이 가능하다.

  ```java
  AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(PrefixConfiguration.class, SequenceGeneratorConfiguration.class);
  ```

* @Import로 구성 파일을 나누어 임포트 하는 방법도 있다.

* 두개의 빈이 각각 다른 구성 클래서에 정의되어 있다면, @Import를 통해서 해당 클래스에 정의한 POJO들을 모두 현재 구성 클래스의 스코프로 가져올 수 있다.

* 그런다음 @Value와 SpEL을 써서 필드에 주입할 수 있다.

  ```java
  @Configuration
  @Import(PrefixConfiguration.class)
  public class SequenceConfiguration {
    
    @Value("#{datePrefixGenerator}")
    private PrefixGenerator prefixGenerator;
    
    ...
  }
  ```

## 2-4. @Resource와 @Inject를 붙여 POJO 자동 연결하기

* 스프링 전용 @Autowired 대신, 자바 표준 애너테이션 @Resource, @Inject로 POJO를 자동 연결할 수 있다.
* @Resource는 JSR-250(Common Annotations for the Java Platform)에 규정된 애너테이션으로, 이름으로 POJO 레퍼런스를 찾아 연결한다.
* @Inject는 JSR-330(Standard Annotations for Injection)에 규정된 애너테이션으로, 타입으로 POJO 레퍼런스를 찾아 연결한다.

### @Resource

* 타입으로 POJO를 찾아 자동 연결하는 기능은 @Resource나 @Autowired나 마찬가지지만,

* 타입이 같은 POJO가 여럿일 때 @Autowired를 쓰면 가리키는 대상이 모호해진다. 결국 @Qualifier를 써서 이름으로 다시 POJO를 찾아야 하는 불편함이 따른다.

* @Resource는 기능상 @Autowired + @Qualifier 이므로 대상이 명확하다.

  ```java
  public class SequenceGenerator {
    
    @Resource
    private PrefixGenerator prefixGenerator;
    
    ...
  }
  ```

### @Inject

* @Resource와 @Autowired 처럼 @Inject도 일단 타입으로 POJO를 찾는다.

  ```java
  public class SequenceGenerator {
    
    @Inject
    private PrefixGenerator prefixGenerator;
    
    ...
  }
  ```

* 하지만 타입이 같은 POJO가 여럿일 때는 @Inject를 이용하려면 POJO 주입 클래스와 주입 지점을 구별하기 위해 커스텀 애너테이션을 작성해야 한다.

  ```java
  @Qualifier
  @Target({ElementType.TYPE, ElementType.FIELD, ElementType.PARAMETER})
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  public @interface DatePrefixAnnotation {
    
  }
  ```

* 위의 커스텀 애너테이션에서 붙인 @Qualifier는 스프링에서 제공하는 것이 아닌, javax.inject 패키지에 속한 애너테이션이다.

* 커스텀 애너테이션을 작성한 다음, 빈 인스턴스를 생성하는 POJO 주입클래스에 붙인다.

  ```java
  @DatePrefixAnnotation
  public class DatePrefixGenerator implements PrefixGenerator {
    ...
  }
  ```

* 이제 POJO 속성 또는 주입 지점에 커스텀 애너테이션을 붙이면 더이상 모호해질 일은 없다.

  ```java
  public class SequenceGenerator {
    
    @Inject
    @DatePrefixAnnotation
    private PrefixGenerator prefixGenerator;
    ...
  }
  ```

* @Autowired, @Resource, @Inject 셋중 어느걸 쓰더라도 결과는 같다. @Autowired는 스프링, @Resource, @Inject는 자바 표준(JSR, Java Specification Requests)에 근거한다. 이름 기반이라면 @Resource, 타입 기반이라면 아무거나 써도 된다.

## 2-5. @Scope를 붙여 POJO 스코프 지정하기

* @Component 같은 애너테이션을 POJO에 붙이는 건 빈 생성에 관한 템플릿을 정의하는 것이지, 실제 빈 인스턴스를 정의하는게 아니다.

* getBean() 메서드로 빈을 요청하거나 다른 빈에서 참조할 때 스프링은 빈 스코프에 따라 어느 빈 인스턴스를 반환할지 결정해야 한다.

* 스프링의 빈 스코프

  | 스코프        | 설명                                                         |
  | ------------- | ------------------------------------------------------------ |
  | singleton     | IoC 컨테이너당 빈 인스턴스 하나 생성                         |
  | prototype     | 요청할 때마다 빈 인스턴스 새로 생성                          |
  | request       | HTTP 요청당 하나의 빈 인스턴스를 생성. 웹 애플리케이션 컨텍스트에만 해당 됨 |
  | session       | HTTP 세션당 빈 인스턴스 하나 생성. 웹 애플리케이션 컨텍스트에만 해당 됨 |
  | globalSession | 전역 HTTP 세션당 빈 인스턴스 하나를 생성한다. 포털 애플리케이션 컨텍스트에만 해당된다. |

* 아래와 같은 쇼핑카트 클래스가 존재한다.

  ```java
  @Component
  public class ShoppingCart {
    
    private List<Product> items = new ArrayList<>();
    
    public void addItem(Product item) {
      items.add(item);
    }
    
    public List<Product> getItems() {
      return items;
    }
  }
  ```

* 상품 빈들을 카트에 추가핧 수 있게 자바 구성파일에 선언한다.

  ```java
  @Configuration
  @ComponentScan("io.namjune.springrecipes.shop")
  public class ShopConfiguration {
    
    @Bean
    public Product a() {
      ...
    }
    
    @Bean
    public Product b() {
      ...
    }
    
    @Bean
    public Product c() {
      ...
    }
  }
  ```

* 카트를 상품에 넣어가면서 테스해보자.

  ```java
  public class Main {
    
    public static void main(String[] args) {
      ApplicationContext context = 
        new AnnotationConfigApplicationContext(ShopConfiguration.class);
      
      Product a = context.getBean("a", Product.class);
      Product b = context.getBean("b", Product.class);
      Product c = context.getBean("a", Product.class);
     
      ShoppingCart cart1 = context.getBean("shoppingCart", ShoppingCart.class);
      cart1.add(a);
      cart1.add(b);
      System.out.println("Cart 1 contains " + cart1.getItems());
      
      ShoppingCart cart2 = context.getBean("shoppingCart", ShoppingCart.class);
      cart2.addItem(c);
      System.out.println("Cart 2 contains " + cart2.getItems());
    }
  }
  ```

* ShoppingCart의 빈 스코프 설정을 아무것도 하지 않았으므로 기본으로 singleton 빈이 생성된다.

* 따라서, 두 고객이 동일한 쇼핑카트를 공유하는 상황이 벌어진다.

  ```java
  Cart 1 contains [a, b]
  Cart 2 contains [a, b, c]
  ```

* 쇼핑몰 방문 고객마다 상이한 카트 인스턴스를 만들어야 맞는 상황이다. ShppingCart 클래스의 빈 스코프를 요청마다 생성하는 prototype으로 설정하자.

  ```java
  @Component
  @Scope("prototype")
  public class ShoppingCart {
    ...
  }
  ```

* 이제 두 고객은 각자의 쇼핑카트를 소유할 수 있다.

## 2-6. 외부 리소스(텍스트, XML, 프로퍼티, 이미지 파일)의 데이터 사용하기

### 프로퍼티 파일 데이터를 이용해서 POJO 초깃값 설정하기

* 클래스패스에 아래와 같이 discounts.properties 파일에 키-값이 들어 있다고 가정하자.

  ```properties
  // discounts.properties
  
  specialcustomer.discount=0.1
  summer.discount=0.15
  endofyear.discount=0.2
  ```

* 다음과 같이 구성파일을 작성해서 프로퍼티 파일을 참조할 수 있다.

  * 스프링은 자바 클래스패스(접두어 classpath:)에서 discounts.properties 파일을 찾는다.
  * @PropertySource를 붙여서 프로퍼티 파일을 로드하려면 PropertySourcesPlaceholderConfigurer 빈을 @Bean으로 선언해야 한다.
  * 그러면 스프링은 discounts.property 파일을 자동으로 연결하므로, 파일에 나열된 프로퍼티를 빈 프로퍼티로 활용할 수 있다.
  * Properties 파일에서 가져온 프로퍼티값을 담을 자바 변수를 정의하고, @Value placeholder 표현식을 넣어서 프로퍼티 값을 변수에 할당한다. 형식은 @Value("${key:default_value}") 이다.

  ```java
  @Configuration
  @PropertySource("classpath:discounts.properties")
  @ComponentScan("io.namjune.springrecipes.shop")
  public class ShopConfiguration {
    
    // key가 endofyear.discount이고, 기본 값은 0 이다.ㅎ
    @Value("${endofyear.discount:0}")
    private double specialEndofyearDiscountField;
    
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
      return new PropertySourcesPlaceholderConfigurer();
    }
  }
  ```

* 프로퍼티 파일 데이터를 빈 프로퍼티 구성 외의 다른 용도로 쓰려면 스프링의 Resource 매커니즘을 이용한다.

### POJO에서 외부 리소스 파일 데이터를 가져와 사용하기

* 애플리케이션 시동시 클래스패스에 위치한 banner.txt라는 텍스트 파일 안에 넣은 문구를 배너로 보여주려고 한다.

  ```text
  //banner.txt
  
  *******************
  Wellcome to My Shop
  *******************
  ```

* 배너를 읽어 콘솔에 출력하는 POJO 클래스이다.

  * banner가 스프링 Resource 타입으로 선언되어있고, 그 값은 빈 인스턴스 생성시 세터 주입을 사용해서 채워진다.
  * showBanner() 메서드는 Files 클래스의 lines(), forEachOrdered() 메서드를 이용해서 배너 파일의 내용을 읽어 한줄씩 출력한다.

  ```java
  public class BannerLoader {
    
    private Resource banner;
    
    public void setBanner(Resource banner) {
      this.banner = banner;
    }
    
    @PostConstructor
    public void showBanner() throws IOException {
      Files.lines(Paths.get(banner.getURI()), Charset.forName("UTF-8"))
        .forEachOrdered(System.out::println);
    }
  }
  ```

* BannerLoader를 인스턴스화 하기 위한 구성 클래스를 작성하자.

  * @Value("classpath:discounts.properties") 덕분에 스프링이 클래스패스에서 banner.txt 파일을 찾아서
  * banner 필드에 주입한다.
  * 세터 주입을 거쳐서 BannerLoader 빈에 할당 되고, @PostConstructor로 인해서 IoC 컨테이너 구성 시점에 파일에 있는 배너가 출력 된다.

  ```java
  @Configuration
  @PropertySource("classpath:discounts.properties")
  @ComponentScan("io.namjune.springrecipes.shop")
  public class ShopConfiguration {
   
    @Value("classpath:discounts.properties")
    // 절대경로 명시 - @Value("file:c:/shop/banner.txt")
    // 특정 패키지 - @Value("classpath:io/namjune/springrecipes/shop/banner.txt")
    private Resource banner;
    
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
      return new PropertySourcesPlaceholderConfigurer();
    }
    
    @Bean
    public BannerLoader bannerLoader() {
      BannerLoader bl = new BannerLoader();
      bl.setBanner(banner);
      return bl;
    }
  }
  ```

* 자바 클래스패스에 있는 리소스들은 ClassPathResource로 액세스할 수 있지만, 

  ```java
  // 경로를 지정하지 않으면 클래스패스 루트이다.
  Resource resource = new ClassPathResource("discount.properties");
  
  // 특정 패키지에 위치한 리소스는 클래스패스 루트부터 절대 경로를 명시하면 된다.
  Resource resource = new ClassPathResource("io/namjune/springrecipes/shop.banner.txt");
  ```

* 외부 파일시스템에 있는 리소스는 FileSystemResouece를 사용하며,

  ```java
  Resource resource = new FileSystemResource("c:/shop/banner.txt");
  ```

* URL로 외부 리소스를 액세스하려면 스프링 UrlResource를 사용한다.

  ```java
  Resource resource = new UrlResource("http://www.apress.com/");
  ```

## 2-7. 프로퍼티 파일에서 로케일마다 다른 다국어 메시지 해석하기

* 스프링의 MessageSource 인터페이스는 리소스 번들 메시지를 처리하는 메서드가 몇가지 정의되어 있다.

* ResourceBundleMessageSource는 가장많이 쓰는 MessageSource 구현체로, 로케일별로 분리된 리소스 번들 메세지를 해석한다.

* ResourceBundleMessageSource POJO를 구현하고 Config 파일에 @Bean을 붙여 선언하면 애플리케이션에 필요한 i18n 데이터를 가져다 쓸 수 있다.

* 클래스패스 루트에 messages_en_US.properties 파일을 작성하자

  ```properties
  alert.checkout=A shopping cart has been checked out.
  alert.inventory.checkout=A shopping cart with {0} has been checked out at {1}
  ```

* Config 파일이 빈 인스턴스 정의하기

  * 빈 인스턴스는 반드시 messageSource라고 명명해야 애플리케이션 컨텍스트가 자동 감지한다.

  ```java
  @Configuration
  public class ShopConfiguration {
    
    @Bean
    public ReloadableResourceBundleMessageSource messageSource() {
     
      ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
      messageSource.setBasenames("classpath:messages");
      messageSource.setCacheSeconds(1);
      return messageSource;
    }
  }
  ```

* 위와 같이 MessageSource를 정의하고 영어가 주 언어인 미국 로케일에서 텍스트 메시지를 찾으면,

  * messages_en_US.properties 파일이 제일 먼저 잡힌다. 이 파일이 없다면,
  * messages_en.properties를 그 다음으로 찾고, 이 파일도 없다면
  * 기본 파일인 messages.properties를 선택한다.

* 애플리케이션 컨텍스트를 구성해서 getMessage()로 메시지 해석하기

  * getMessage()의 인수는 키, 메시지 매개변수 배열, 대상 로케일 순이다.
  * 메시지 매개변수 배열은 Object 배열 형태로 넘긴다.

  ```java
  public class Main {
    
    public static void main(String[] args) {
      ApplicationContext context = new AnnotationConfigApplicationContext(ShoopConfiguration.class);
      
      String alert = context.getMessage("alert.checkout", null, Localu.US);
      // A shopping cart with {0} has been checked out at {1}
      String alertInventory = context.getMessage("alert.inventory.checkout", new Object[]{"[DVD-RW 3.0]", new Date()}, Locale.US);
      
      System.out.println("alert.checkout is: " + alert);
      System.out.println("alert.inventory.checkout is: " + alertInventory);
    }
  }
  ```

* 텍스트 메시를 해석하는 빈에는 MessageSource를 주입해야 한다.

  ```java
  @Component
  public class Cashier {
    
    @Autowired
    private MessageSource messageSource;
    
    public void setMessageSource(MessageSource messageSource) {
      this.messageSource = messageSource;
    }
    
    public void checkout(ShoppingCart cart) theow IOException {
      String alert = messageSource.getMessage("alert.inventory.checkout", new Object[]{ cart.getItems(), new Date() }, Locale.US);
      System.out.println(alert);
    }
  }
  ```

## 2-8. 애너테이션을 이용해 POJO 초기화/폐기 커스터마이징하기

### @Bean 정의부에서 initMethod, destroyMethod 속성 설정

* 자바 구성클래스의 @Bean 정의부에서 initMethod, destroyMethod를 설정하면, 스프링은 이들을 각각 초기화, 폐기 콜백 메서드로 인지한다.

  ```java
  public class Cashier {
    
    ...
      
    public void openFile() throws IOException {
      ...
    }
    
    public void closeFile() throws IOException {
      ...
    }
  }
  ```

* 자바 구성파일

  ```java
  @Configuration
  public class ShopConfiguration {
    
    @Bean(initMethod = "openFile", destroyMethod = "closeFile")
    public Cashier cashier() {
      ...
    }
  }
  ```

### @PostConstructor와 @PreDestory로 POJO 초기화/폐기 메서드 지정

* 자바 Configuration 파일 외부에 @Component로 POJO 클래스를 정의할 경우 @PostConstructor와 @PreDestory로 직접 초기화/폐기 메서드를 지정한다.

* 빈 생성 이후에 openFile() 메서드를, 빈 폐기 이전에 closeFile() 메서드가 실행 된다.

  ```java
  @Component
  public class Cashier {
    
    ...
    
    @PostConstructor
    public void openFile() throws IOException {
      ...
    }
    
    @PreDestroy
    public void closeFile() throws IOException {
      ...
    }
  }
  ```

### @Lazy로 느긋하게 POJO  초기화하기

* 기본적으로 스프링은 모든 POJO를 조급하게 초기화 한다.(Eager Initialiaztion)

* 애플리케이션 시동과 동시에  POJO들을 초기화 하게 되는데, 환경에 따라서 **빈을 처음 요청하기 전**까지 초기화 과정을 미루는 게 더 나을 때도 있다.

* 이렇게 나중에 초기화 하는 개념을 느긋한 초기화라고 한다. 이 경우 시동 시점에 리소스를 집중 소모하지 않아도 된다.

* 아래 클래스는 @Lazy 덕분에 애플리케이션이 요구하거나 다른 POJO가 참조하기 전까지 초기화되지 않는다.

  ```java
  @Component
  @Scope("prototype")
  @Lazy
  public class ShoppingCart {
    ...
  }
  ```

### @DependsOn으로 초기화 순서 정하기

* POJO가 늘어날 수록 POJO 초기화 횟수도 증가한다.

* 여러 자바 Configuration에 분산 선언된 많은 POJO가 서로를 참조하다 보면 경합 조건(Race Condition)이 일어나기 쉽다.

* 이런 경우 의존관계에 있는 빈의 초기화가 먼저 이루어질 수 있도록 설정할 수 있다.

* @DependsOn은 어떤 POJO가 다른 POJO 보다 먼저 초기화 되도록 강제하며, 그 과정에서 에러가 나도 쉬운 메시지를 돌려 준다.

* @DependsOn은 빈의 초기화 순서를 보장 한다.

* 아래 설정 파일에서 datePrefixGenerator 빈은 sequenceGenerator 빈 보다 반드시 먼저 생성 된다.

  ```java
  @Configuration
  public class SequenceConfiguration {
    
    @Bean
    @DependsOn("datePrefixGenerator")
    public SequenceGenerator sequenceGenerator() {
      ...
    }
  }
  ```

* @DependsOn의 속성값으로는 { } 로 둘러싼 CSV 리스트 형태로 의존하는 빈을 여러개 지정할 수 있다.

  * ex) @DependsOn({"datePrefixGenerator, numberPrefixGenerator"})

## 2-9. 후처리기(BeanPostProcessor)를 만들어 POJO 검증/수정하기

### 모든 빈 인스턴스를 처리하는 후처리기(BeanPostProcessor) 생성

* 빈 후처리기는 BeanPostProcessor 인터페이스를 구현한 객체이다.

* 이 인터페이스를 구현한 빈을 발견하면, 스프링은 자신이 관장하는 모든 빈 인스턴스에  postProcessBeforeInitialization(), posrProcessAfterInitialization() 두 메서드를 적용한다.

* 두 메서드는 하는 일이 없어도 반드시 원본 빈 인스턴스를 반환해야 한다.

* 애플리케이션 컨텍스트는 BeanProcessor 구현 빈을 감지해서 컨테이너 안에 있는 다른 빈 인스턴스에 일괄 적용 한다.

  ```java
  @Component
  public class AuditCheckBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("In AuditCheckBeanPostProcessor.postProcessBeforeInitialization, processing bean type: " + bean.getClass());
      return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      return bean;
    }
  }
  ```

### 특정 타입의 빈 인스턴스만 처리하는 후처리기(BeanPostProcessor)

* Product 타입인 빈 인스턴스에만 빈 후처리기를 등록할 수 있다.

* postProcessBeforeInitialization(), posrProcessAfterInitialization() 두 메서드는 하는 일이 없어도 처리할 빈 인스턴스를 반드시 반환해야 한다.

* **바꿔서 말하면, 이 과정에서 원본 빈 인스턴스를 다른 인스턴스로 바꿔치기 할 수도 있다는 의미**가 된다.

  ```java
  @Component
  public class ProductCheckBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      if (bean instanceof Product) {
        System.out.println("In ProductCheckBeanPostProcessor.postProcessBeforeInitialization, processing bean type: " + bean.getClass());
      }
      return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      if (bean instanceof Product) {
        System.out.println("In ProductCheckBeanPostProcessor.postProcessAfterInitialization, processing bean type: " + bean.getClass());
      }
     
      return bean;
    }
  }
  ```

### @Required로 프로퍼티 검사하기

* 특정 빈 프로퍼티가 설정되었는지 체크하고 싶은 경우에는, 커스텀 후처리기(RequiredAnnotationBeanPostProcessor)를 작성하고 해당 프로퍼티에  @Required를 설정한다.

* 스프링 빈 후처리기 RequiredAnnotationBeanPostProcessor는 @Required가 붙은 프로퍼티 값이  null인지만 판단한다.

* 해당 프로퍼티가 null일 경우 BeanInitializationException 예외를 던진다.

* @Required로 검증하려면 RequiredAnnotationBeanPostProcessor가 빈으로 등록되어있어야 한다.

  ```java
  public class SequenceGenerator {
    
    private PrefixGenerator prefixGenerator;
    private String suffix;
    
    @Required
    public void setPrefixGenerator(PrefixGenerator prefixGenerator) {
      this.prefixGenerator = prefixGenerator;
    }
    
    @Required
    public void setSuffix(String suffix) {
      this.suffix = suffix;
    }
  }
  ```


## 2-10. 팩토리(정적 메서드, 인스턴스 메서드, 스프링 Factory Bean)로 POJO 생성하기

* 자바 Configuration 클래스의 @Bean 메서드는(일반 자바 구문으로) 정적 팩토리를 호출하거나 인스턴스 팩토리 메서드를 호출해서 POJO를 생성할 수 있다.

### 정적 팩토리 메서드로 POJO 생성하기

* 아래 ProductCreator 클래스에서 정적 팩토리 메서드 createProduct는 productId에 해당하는 Product 객체를 생성한다.

* id에 따라서 인스턴스화 할 실제 Product 객체를 내부 로직으로 결정한다.

  ```java
  public class ProductCreator {
    
    public static Product createProduct(String productId) {
      if ("aaa."equals(productId)) {
        return new Battery("AAA", 2.5);
      } else if ("bbb".equals(productId)) {
        return new Disc("CD-RW", 1.5);
    } else if ("ccc".equals(productId)) {
        return new Disc("DVD-RW", 3.0);
      }
      throw new IllegalArgumentException("Unknown product!");
    }
  }
  ```
  
* 자바 구성 클래스 @Bean 메서드에서 일반 자바 구문으로 정적 팩토리 메서드를 호출해서 POJO를 생성한다.

  ```java
  @Configuration
  public class ShopConfiguration {
    
    @Bean
    public Product aaa() {
      return ProductCreator.createProduct("aaa");
    }
    
    @Bean
    public Product bbb() {
      return ProductCreator.createProduct("bbb");
    }
    
    @Bean
    public Product ccc() {
      return ProductCreator.createProduct("ccc");
    }
  }
  ```

### 인스턴스 팩토리 메서드로 POJO 생성하기

* 맵을 구성해서 상품 정보를 담아두는 방법도 있다.

* 인스턴스 팩토리 메서드 createProduct() 에서는 productId에 해당하는 상품을 맵에서 찾는다.

  ```java
  public class ProductCreator {
    
    private Map<String, Product> products;
    
    public void setProducts(Map<String, Product> products) {
      this.products = products;
    }
    
    public Product createProduct(String productId) {
      Product product = products.get(productId);
      if (product != null) {
        return product;
      }
      throw new IllegalArgumentException("Unknown product");
    }
  }
  ```

* ProductCreator에서 상품을 생성하기 위해서 먼저 팩토리 값을 인스턴스화하고, 팩토리의 파사드 역할을 하는 @Bean 을 선언한다. 마지막으로 팩토리를 호출하고 createProduct() 메서드를 호출해서 다른 빈들을 인스턴스화 한다.

  ```java
  @Configuration
  public class ShopConfiguration {
    
    @Bean
    public ProductCreator productCreatorFactory() {
      ProductCreator factory = new ProductCreator();
      Map<String, Product> products = new HashMap<>();
      products.put("aaa", new Battery("AAA", 2.5));
      products.put("bbb", new Disc("CD-RW", 1.5));
      products.put("ccc", new Disc("DVD-RW", 3.0));
      factory.setProducts(products);
      return factory;
    }
    
    @Bean
    public Product aaa() {
      return productCreatorFactory().createProduct("aaa");
    }
    
    @Bean
    public Product bbb() {
      return productCreatorFactory().createProduct("bbb");
    }
    
    @Bean
    public Product ccc() {
      return productCreatorFactory().createProduct("ccc");
    }
  }
  ```

### 스프링 팩토리 빈으로 POJO 생성하기

* 직접 팩토리 빈을 구현할 일은 별로 없겠지만, 내부 작동 원리를 이해하는 건 여러모로 유익하다. 그리고 나는 이런게 재미있다.

* 할인가가 적용된 상품을 생성하는 팩토리 빈을 작성해보자.

* 이 빈은 product, discount 두 프로퍼티값을 받아 주어진 상품에 할인가를 계산하여 적용하고 Product 빈을 새로 만들어 반환한다.

  * 팩토리 빈은 제네릭 클래스 AbstractFactoryBean\<T\>를 상속하고, createInstance() 메서드를 오버라이드해서 대상 빈 인스턴스를 생성한다.
  * 자동 연결이 가능하록 getObjectType() 메서드로 대상 빈 타입을 반환한다.

  ```java
  public class DiscountFactoryBean extends AbstractFactoryBean<Product> {
    
    private Product product;
    private double discount;
    
    public void setProduct(Product product) {
      this.product = product;
    }
    
    public void setDiscount(double discount) {
      this.discount = discount;
    }
    
    @Override
    public Class<> getObjectType() {
      return product.getClass();
    }
    
    @Override
    protected Product createInstance throws Exception {
      product.setPrice(product.getPrice() * (1 - discount));
      return Product;
    }
  }
  ```

* Product 인스턴스를 생성하는 팩토리 빈에 DiscountFactoryBean을 적용해보자.

  ```java
  @Configuration
  @ComponentScan("io.namjune.springrecipes.shop")
  public class ShopConfiguration {
   
    @Bean
    public Product aaa() {
      return productCreatorFactory().createProduct("aaa");
    }
    
    @Bean
    public Product bbb() {
      return productCreatorFactory().createProduct("bbb");
    }
    
    @Bean
    public Product ccc() {
      return productCreatorFactory().createProduct("ccc");
    }
    
    @Bean
    public DiscountFactoryBean discountFactoryBeanAAA() {
      DiscountFactoryBean factory = new DiacountFactoryBean();
      factory.setProduct(aaa());
      factory.setDiscount(0.2);
      return factory;
    }
    
    @Bean
    public DiscountFactoryBean discountFactoryBeanBBB() {
      DiscountFactoryBean factory = new DiacountFactoryBean();
      factory.setProduct(bbb());
      factory.setDiscount(0.21);
      return factory;
    }
    
    @Bean
    public DiscountFactoryBean discountFactoryBeanCCC() {
      DiscountFactoryBean factory = new DiacountFactoryBean();
      factory.setProduct(ccc());
      factory.setDiscount(0.1);
      return factory;
    }
  }
  ```

## 2-11. 스프링 환경 및 프로파일 마다 다른 POJO 로드하기

* 동일한 POJO 인스턴스/빈을 여러 애플리케이션 시나리오(예: 개발, 테스트, 운영)별로 초깃값을 달리해서 구성할 수 있다. 
* 자바 구성 클래스를 여러개 만들고 각 클래스마다 POJO 인스턴스/빈을 묶는다.
* 이렇게 묶은 의도를 잘 표현할 수 있게 프로파일을 명명하고 자바 구성클래스에 @Profile을 설정한다. 
* 그리고, 애플리케이션 컨텍스트가 시나리오에 맞는 구성 클래스 파일을 읽어들여 실행하게 만들면 된다.

### @Profile로 자바 구성 클래스를 프로파일 별로 작성하기

* @Profile은 클래스 레벨로 선언했기 때문에 자바 구성 클래스에 속한 모든 @Bean 인스턴스는 해당 프로파일에 편입 된다.

  ```java
  @Configuration
  @Profile("global")
  @ComponentScan("io.namjune.springrecipes.shop")
  public class ShopConfiguration {
    
    @Bean(initMethod = "openFile", destroyMethod = "closeFile")
    public Cashier cashier() {
      ...
    }
  }
  ```

  ```java
  @Configuration
  @Profile({"summer", "winter"})
  public class ShopConfigurationSumWin {
    
    @Bean
    public Product aaa() {
      ...
    }
    
    @Bean
    public Product bbb() {
      ...
    }
    
    @Bean
    public Product ccc() {
      ...
    }
  }
  ```

### 프로파일을 환경에 로드하기

* 프로파일에 속한 빈을 애플리케이션에 로드하려면 프로파일을 활성화 해야 한다.

* 프로파일 여러개를 한 번에 로드하는 것도 가능하며, 자바 런타임 플래그나 WAR파일 초기화 매개변수를 지정해서 프로그램 방식으로 프로파일을 로드할 수도 있다.

* 프로그램 방식으로 프로파일을 로드하려면 먼저 컨텍스트를 가져와서 setActiveProfiles() 메서드를 호출한다.

  ```java
  AnnotationConfigApplicationContext context = new AnnotaionConfigApplicationContext();
  context.getEnvironment().setActiveProfiles("global", "winter");
  context.scan("io.namjune.springrecipes.shop");
  context.refresh();
  ```

* 자바 런타임 플래그로 로드할 프로파일을 명시할 수도 있다.

  ```java
  -Dspring.profiles.active=global,winter
  ```

* 기본 프로파일을 지정하려면

  * 프로그램 방식의 경우 setActiveProfiles() 대신 setDefaultProfiles() 메서드를 쓰면 되고,
  * 자바 런타임 플래그나, WAR파일 초기화 매개변수를 쓸때는 spring.profiles.active 대신 spring.profiles.default로 대신하면 된다.

## 2-12. POJO에게 IoC 컨테이너 리소스 알려주기

* 컴포넌트가 IoC 컨테이너와 직접적인 의존 관계를 가지도록 설계하는 방법은 바람직하진 않지만,

* 때로는 빈에서 컨테이너의 리소스를 인지해야 하는 경우도 있다.

* 빈이 IoC 컨테이너 리소스를 인지하게 하려면 Aware(인지) 인터페이스를 구현한다. 스프링은 이 인터페이스를 구현한 빈을 감지해서 대상 리소스를 세터 메서드로 주입한다.

* 자주 쓰는 스프링 Aware 인터페이스

  * ApplicationContext는 MessageSource, ApplicationEventPublisher, ResourceLoader를 모두 상속한 인터페이스라서, ApplicationContext만 인지하면 나머지 서비스도 엑세스 할 수 있다.
  * **그러나 요건을 충족하는 최소한의 범위 내에서 타이트하게 Aware 인터페이스를 선택하는 게 바람직하다.**

  | Aware 인터페이스               | 대상 리소스 타입                                             |
  | ------------------------------ | ------------------------------------------------------------ |
  | BeanNameAware                  | IoC 컨테이너에 구성한 인스턴스의 빈 이름                     |
  | BeanFactoryAware               | 현재 빈 팩토리, 컨테이너 서비스를 호출하는 데 쓰인다.        |
  | ApplicationContextAware        | 현재 애플리케이션 컨텍스트, 컨테이너 서비스를 호출하는 데 쓰인다. |
  | MessageSourceAware             | 메시지 소스, 텍스트 메시지를 해석하는 데 쓰인다.             |
  | ApplicationEventPublisherAware | 애플리케이션 이벤트 퍼블리셔, 애플리케이션 이벤트를 발행하는 데 쓰인다. |
  | ResourceLoaderAware            | 리소스 로더. 외부 리소스를 로드하는 데 쓰인다.               |
  | EnvironmentAware               | ApplicationContext 인터페이스에 묶인 org.springframework.core.env.Environment 인스턴스 |

* Aware 인터페이스의 세터 메서드는 스프링이 빈 프로퍼티를 설정한 이후, 초기화 콜백 메서드를 호출하기 이전에 호출한다. 정리하자면 아래의 순서이다.

  * 생성자나 팩토리 메서드를 호출해서 빈 인스턴스를 생성한다.
  * 빈 프로퍼티에 값, 빈 레퍼런스를 설정한다.
  * Aware 인터페이스에 정의한 세터메서드를 호출한다.
  * 빈 인스턴스는 각 BeanPostProcessor에 있는 postProcessBeforeInitialization() 메서드로 넘겨 초기화 콜백 메서드를 호출한다.
  * 빈 인스턴스는 각 BeanPostProcessor에 있는 PostProcessAfterInitialization() 메서드로 넘긴다. 이제 빈을 사용할 준비가 끝났다.
  * 컨테이너가 종료되면 폐기 콜백 메서드를 호출한다.

* **주의!** Aware 인터페이스를 구현한 클래스는 스프링과 엮이게 되므로 IoC 컨테이너 외부에서는 제대로 작동하지 않는다는 것을 꼭 기억해야 한다. 스프링에 종속된 인터페이스를 꼭 구현해야 할 필요가 있는지 잘 따져봐야 한다.

  * 스프링 최신 버전에서는 사실 Aware 인터페이스를 구현할 필요가 없다.
  * 왜냐면 @Autowired로 얼마든지 ApplicationContext를 가져올 수 있다. 물론 프레임워크나 라이브러리를 개발할 때에는 Aware를 구현하는 것이 나을 수도 있다.

* 아래의 Cashier POJO가 자신의 빈 이름을 인지해서 저장하는 파일의 이름으로 쓰려고 하면, BeanNameAware를 구현하도록 수정해야 한다.

  ```java
  public class Cashier implements BeanNameAware {
    ...
    private String fileName;
    
    @Override
    public void setBeanName(String beanName) {
      this.fileName = beanName;
    }
  }
  ```


## 2-13. 애너테이션을 활용해 애스펙트 지향 프로그래밍하기

* 애스팩트를 정의하려면 일단 자바 클래스에 @Aspect를 붙이고, 메서드별로 적절한 애너테이션을 붙여 어드바이스로 만든다.
* 어드바이스 애너테이션은 @Before, @After, @AfterReturning, @AfterThrowing, @Around 5개 중 하나를 쓸 수 있다.
* IoC 컨테이너에서 애스펙트 애너테이션 기능을 활성화하려면 구성 클래스 중 하나에 @EnableAspectAutoProxy를 붙인다.
* 기본적으로 스프링은 인터페이스 기반의 JDK 동적(dynamic) 프록시를 생성하여 AOP를 적용한다. 인터페이스를 사용할 수 없거나 애플리케이션 설계상 사용하지 않을 경우엔 CGLIB으로 프록시를 만들 수 있다.
* @EnableAspectAutoProxy에서 proxyTargetClass 속성을 true로 설정하면 동적프록시 대신 CGLIB을 사용한다.
* 스프링에서는 AspectJ와 동일한 애너테이션으로 애너테이션 기반 AOP를 구현한다. 포인트것을 파싱, 매치하는 AspectJ 라이브러리를 그대로 빌려왔다.
* 하지만, AOP 런타임 자체는 순수 스프링 AOP 이기 때문에 AspectJ 컴파일러나 위버(weaver)와는 아무런 의존 관계가 없다.

### 애스펙트, 어드바이스, 포인트컷 선언

* 애스펙트는 여러 타입과 객체에 공통 관심사(로깅, 트랜잭션 관리 등)를 모듈화한 자바 클래스로, @Aspect를 붙여 표시한다.
* AOP에서 말하는 애스펙트란 어디에서(포인트컷) 무엇을 할 것인지(어드바이스)를 합쳐 놓은 개념이다.
* **어드바이스** 는 **@Advice를 붙인 단순 자바 메서드**로, AspectJ는 @Before, @After, @AfterReturning, @AfterThrowing, @Around 다섯 개 어드바이스 애너테이션을 지원한다.
* **포인트컷** 은 **어드바이스에 적용할 타입 및 객체를 찾는 표현식**이다.

### @Before 어드바이스

* Before 어드바이스는 특정 프로그램 **실행 지점 이전**의 공통 관심사를 처리하는 메서드로, @Before를 붙이고 포인트컷 표현식을 애너테이션 값으로 지정한다.

  * 이 포인트컷 표현식은 ArithmeticCalculator 인터페이스의 add() 메서드 실행을 가리킨다.
  * 와일드카드(*)는 모든 제어자(public, protected, pricate), 모든 반환형을 매치함을 의미한다.
  * 인수 목록 부분에 쓴 두 점(..)은 인수 갯수는 몇 개라도 좋다는 뜻이다. 
  * @Aspect만 붙여서는 스프링이 클래스패스에서 자동 감지하지 않기 때문에 POJO 마다 개별적으로 @Component선언을 해야 한다. 

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    private Log log = LogFactory.getLog(this.getClass());
    
    @Before("execution(* ArtimeticCalculator.add(..))")
    public void logBefore() {
      log.info("The method add() begins");
    }
  }
  ```

* 자바 Configuration 클래스에 @EnableAspectJAutoProxy를 붙여 POJO와 애스펙트를 스프링이 스캐닝하게 한다.

  ```java
  @Configuration
  @EmableAspectJAutoProxy
  @ComponentScan
  public class CalcuratorConfiguration {
    
  }
  ```

* 포인트컷으로 매치한 실행 지점을 **조인포인트**(JoinPoint)라고 한다.

* 포인트컷은 여러 조인포인트를 매치하기 위해 지정한 표현식이고, 이렇게 매치된 조인포인트에서 해야 할 일이 바로 **어드바이스** 이다.

* 어드바이스가 현재 조인포인트의 세부적인 내용에 액세스하려면 JoinPont형 인수를 어드바이스 메서드에 선언해야 한다.

* 그러면 메서드면, 인수값 등 자세한 조인포인트 정보를 조회할 수 있다.

* 클래스명, 메서드명에 와일드카드를 써서 모든 메서드에 예외없이 포인트 컷을 적용하는 코드는 아래와 같다.

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    ...
      
    @Before("execution(* *.*(..))")
    public void logBefore(JoinPoint joinPoint) {
      log.info("The Method " + joinPoint.getSignature.getName() 
               + "() begins with " + Arrays.toString(joinPoint.getArgs()));
    }
  }
  ```

### @After 어드바이스

* After 어드바이스는 조인포인트가 끝나면 실행되는 메서드, @After를 붙여 선언한다.

* 조인포인트가 정상 실행되든, 도중에 예외가 발생하든 상관없이 실행된다.

  ```java
  @Aspect
  @Component
  public class ClaculatorLoggingAspect {
    ...
    
    @After("execution(* *.*(..))")
    public void logAfter(JoinPoint joinPoint) {
      log.info("The method " + joinPoint.getSignature().getName() + "() ends");
    }
  }
  ```

### @AfterReturning 어드바이스

* After 어드바이스는 조인포인트 실행의 성공 여부와 상관없이 작동한다.

* logAfterReturning() 메서드에서 조인포인트가 값을 반환할 경우에만 로깅하고 싶다면, 다음과 같이 AfterReturning 어드바이스로 대체하면 된다.

* Return 값을 가져오려면, @AfterReturning의 returning 속성으로 지정한 변수명을 어드바이스 메서드의 인수로 지정한다.

* 스프링은 런타임에 조인포인트의 반환값을 이 인수에 넣어 전달한다.

* 이때 포인트컷 표현식은 pointcut 속성으로 따로 정의한다.

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    ...
    
    @AfterReturning(
      pointcut = "execution(* *.*(..))",
      returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
      log.info("The method {}() ends with {}", joinPoint.getSignature().getName(), result);
    }
  }
  ```

### @AfterThrowing 어드바이스

* After Throwing 어드바이스는 조인포인트 실행 도중 예외가 날 경우에만 실행된다.

* 작동 원리는 @AfterReturning과 같다. 

* 발생한 예외는 @AfterThrowing의 throwing 속성에 담아 전달할 수 있다.

* 아래와 같이 모든 에러/예외 클래스의 상위 타입인 Throwable을 어드바이스에 적용하면, 조인포인트에서 발생한 에러/예외를 전부 가져온다.

* 특정한 예외만 관심있다면, 충분히 NPE만 잡을 수도 있다.

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    ...
    
    @AfterThrowing(
      pointcut = "execution(* *.*(..))",
      throwing = "e")
    public void logAfterReturning(JoinPoint joinPoint, Throwable e) {
      log.info("An exception {} has been thrown in {}()",
               e, joinPoint.getSignature().getName());
    }
  }
  ```

### @Around 어드바이스

* Around는 가장 강력한 어드바이스다. 조인포인트를 완전히 장악하기 때문에,

* 앞서 살펴본 어드바이스 모두 Around 어드바이스로 조합할 수 있다.

* 심지어 원본 조인포인트를 언제 실행할지, 실행 할지말지, 계속 실행할지 여부 까지 제어할 수 있다.

* 아래 코드는 Before, After Returning, After Throwing 어드바이스를 Around 어드바이스로 조합한 코드이다.

  * Around 어드바이스의 조인포인트 인수형은 ProceedingJoinPoint로 고정되어 있다.
  * JoinPoint의 하위 인터페이스인 ProceedingJoinPoint를 이용하면 원본 조인포인트를 언제 진행할지 그 시점을 제어할 수 있다.

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    private Logger log = LoggerFactory.getLogger(this.getClass());
    
    @Around("execution(* *.*(..))")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
      // @Before
      log.info("The method {}() begins with {}", joinPoint.getSignature().getName(), Arrays.toString(joinPoint.getArgs()));
      
      try {
        Object result = joinPoint.proceed();
        // @AfterRetuning
        log.info("The method{}() ends with", joinPoint.getSignature().getName(), result);
        return result;
      } catch (IllegalArgumentException e) {
        // @AfterThrowing
        log.error("Illegal argument {} in {}()", Arrays.toString(joinPoint.getArgs()), joinPoint.getSignature.getName());
        throw e;
      }
    }
  }
  ```

* Around는 매우 강력하고 유연해서 원본 인수값을 바꾸거나, 최종 반환값을 바꾸는 것도 가능하지만, 산혹 원본 조인포인트를 진행하는 호출을 잊어버리기 쉬우므로 사용을 주의해야 한다.

* **최소한의 요선을 충족하면서도 가장 기능이 약한 어드바이스를 쓰는게 바람직하다.**

  * ApplicationContextAware가 강력하지만 가장 타이트하게 MessageSourceAware를 쓰는 것과 같은 이치이다.

## 2-14. 조인포인트 정보 가져오기

* 어드바이스에서 조인포인트 정보를 액세스 할 수 있다.

* 조인포인트 유형(스프링 AOP의 메서드 실행만 해당), 메서드 시그니처(선언 타입 및 메서드명), 인수값, 대상 객체(프록시로 감싼 원본 빈), 프록시 객체

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    private Logger log = LoggerFactory.getLogger(this.getClass());
    
    @Before("execution(* *.*(..))")
    public void logJoinPoint(JoinPoint joinPoint) {
      
      // method-execution
      log.info("JoinPoint Kind : {}", joinPoint.getKind()); 
      
      // io.namjune.springrecipes.shop.calculator.ArithmeticCalculator
      log.info("signathre declaring type : {}", joinPoint.getSignature().getDeclaringTypeName());
      
      // add
      log.info("Signature name : {}", joinPoint.getSignature().getName());
  
      // [1.0, 2.0]
      log.info("Arguments : {}", Arrays.toString(joinPoint.getArgs()));
      
      // io.namjune.springrecipes.calculator.ArithmeticCalculator
      log.info("Target class : {}", joinPoint.getTarget().getClass().getName());
      
      // com.sum.proxy.$Proxy6
      log.info("This class : {}", joinPoint.getThis().getClass().getName());
    }
  }
  ```

## 2-15. @Order로 애스펙트 우선순위 설정하기

* Log를 찍는 애스펙트 말고, 계산기의 인수를 검증하는 애스펙트를 하나 더 만들었다고 가정하자.

* 로깅 애스펙트와 검증 애스펙트 둘 다 쓰자면 어느 쪽을 먼저 적용해야할지 알 수 없다.

* 따라서, 애스펙트의 우선순위를 부여할 수 있다.

* **Ordered 인터페이스를 구현해서 getOrder() 오버라이드 한다. 이 메서드가 반환하는 값이 작을수록 우선순위가 높게 세팅 된다.**

* 또는, **@Order를 활용해서 우선순위값을 세팅한다. 마찬가지로 작을수록 우선순위가 높다.**

  ```java
  @Aspect
  @Component
  @Order(0)
  public class CalculatorValidationAspect{
    ...
  }
  
  @Aspect
  @Component
  @Order(1)
  public class CalculatorLoggingAspect{
    ...
  }
  ```


## 2-16. 애스펙트 포인트컷 재사용하기

* 포인트컷 표현식을 여러번 반복해서 쓸 경우엔 매번 어드바이스에 직접 정의하는 것 보다, 재사용할 방법이 필요하다.

* @Pointcut을 이용하면 포인트컷만 따로 정의해서 여러 어드바이스에서 재사용할 수 있다.

* 애스펙트에서 포인트컷은 @Pointcut을 붙인 단순 메서드로 선언할 수 있다.

* @Pointcut은 메서드 바디는 보통 비워두고, 포인트컷의 가시성은 메서드 접근제한자로 조정한다.

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
  	...
    
    @Pointcut("execution(* *.*(..))")
    private void loggingOperation() {}
    
    @Before("loggingOperation()")
    public void logBefore(JoinPoint joinPoint) {
      ...
    }
    
    @AfterReturning(
    	pointcut = "loggingOperation()",
      returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
      ...
    }  
  }
  ```

* 여러 애스펙트가 포인트컷을 공유하는 경우에는 공통 클래스 한 곳에 포인트컷을 모아두는게 좋다.

* 당연히 메서드는 public으로 선언해야 한다.

  ```java
  @Aspect
  @Component
  public class CalculatorPointcuts {
    
    @Pointcut("execution(* *.*(..))")
    public void loggingOperation() {}
    
    ...
  }
  ```

* 외부에서 포인트컷을 참조할 때는 클래스명도 함께 적는다. 참조할 포인트컷이 현재 애스펙트와 다른 패키지에 있으면 패키지명까지 기재한다.

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    
    @Before("CalculatorPointcuts.loggingOperation()")
    public void logBefore(JoinPoint joinPoint) {
      ...
    }
    
    ...
  }
  ```

## 2-17. AspectJ 포인트컷 표현식 작성하기

* AspectJ는 다양한 종류의 조인포인트를 매치할 수 있는 강력한 표현식 언어를 제공한다.
* 하지만, 스프링 AOP가 지원하는 조인포인트 대상은 IoC 컨테이너 안에 선언된 빈에 국한된다.
* 스프링 AOP에서는 AspectJ 포인트컷 언어를 활용해 포인트컷을 정의하며, 런타임에 AspectJ 라이브러리를 이용해서 포인트컷 표현식을 해석한다.
* IoC 컨테이너 스코프를 벗어나 포인트컷 표현식을 쓰면 IllegalArgumentException이 발생한다.
* 스프링에 구현된 포인트컷 표현식 패턴을 알아본다.

### 매서드 시그니처 패턴

* 포인트컷 표현식의 가장 일반적인 모습은 시그니처를 기준으로 여러 메서드를 매치하는 것이다.

* 예를 들면, 다음 포인트컷 표현식은 ArithmeticCalculator 인터페이스에 선언한 메서드 전부를 매치한다.

  * `execution(* io.namjune.springrecipes.calculator.ArithmeticCalculator.*(..))`

* 대상 클래스나 인터페이스가 애스펙트와 같은 패키지에 있으면 패키지명은 안 써도 된다.

* 다음 포인트컷 표현식은 모든 public 메서드를 매치한다.

  * `execution(public * ArithmethcCalculator.*(..))`

* 메서드 반환형을 특정할 수도 있다. 다음은 double형을 반환하는 메서드만 매치한다.

  * `execution(public double ArithmeticCalculator.*(..))`

* 인수 목록도 제약을 둘 수 있다. 다음은 첫 번째 인수가 double 형인 메서드만 매치하며 두 점(..)은 두 번째 이후 인수는 몇개라도 상관 없음을 의미한다.

  * `execution(public double ArithmeticCalculator.*(double, ..))`

* 또는 인수형과 갯수가 정확히 매치되게 할 수도 있다.

  * `execution(public double ArithmeticCalculator.*(double, double))`

* AspectJ는 충분히 강력하지만, 간혹 매치하고 싶은 메서드들 사이에 공통 특성이 없는 경우도 있다.

* 이럴때는 메서드/타입 레벨에 다음과 같은 **커스텀 애너테이션을 만들어 붙이면 된다.**

  * 로깅이 필요한 모든 메서드에 커스텀 애너테이션 @LoggingRequired를 붙이면 된다.
  * 클레스 레벨에 붙이면 모든 메서드에 적용된다.
  * 단, 애너테이션은 상속되지 않으므로 인터페이스가 아닌 구현 클래스에만 붙여야 한다.

  ```java
  @Target({Element.METHOD, Element.TYPE})
  @Retention(RetentionPolict.RUNTIME)
  @Documented
  public @interfacc LoggingRequired {
    
  }
  ```

* 커스텀 애너테이션 활용 클래스

  ```java
  @LoggingRequired
  public class ArithmeticCalculatorImpl implements ArithmeticCalculator {
    
    public double add(double a, double b) {
      ...
    }
    
    public double sub(double a, double b) {
      ...
    }
  }
  ```

* 그 다음, @LoggingRequired를 붙인 클래스/메서드를 스캐닝하도록 @Pointcut의 annotation 안에 포인트컷 표현식을 세팅해 준다.

  ```java
  @Aspect
  @Component
  public class CalculatorPointcuts {
  
    @Pointcut("annotation(io.namjune.springrecipes.calculator.LoggingRequired)")
    public void loggingOperation() {}
  }
  ```

### 타입 시그니처 패턴

* 특정한 타입 내부의 모든 조인포인트를 매치하는 포인트컷 표현식도 있다.

* 스프링 AOP에 적용하면 그 타입 안에 구현된 메서드를 실행할 때만 어드바이스가 적용되도록 포인트컷 적용 범위를 줄일 수 있다.

* 다음 포인트컷은 io.namjune.springrecipes.calculator 패키지의 전체 메서드 실행 조인포인트를 매치한다.

  * `within(io.namjune.springrecipes.calculator.*)`

* 하위 패키지도 함께 매치하려면 와일드카드 앞에 점 하나를 더 쓴다.

  * `within(io.namjune.springrecipes.calculator..*)`

* 어느 한 클래스 내부에 구현된 메서드 실행 조인포인트를 매치하려면 아래와 같다.

  * `within(io.namjune.springrecipes.calculator.ArithmeticCalculatorImpl)`

* 해당 클래스의 패키지가 애스펙트와 같으면 패키지명은 안 써도 된다.

* ArithmeticCalculator 인터페이스를 구현한 모든 클래스의 메서드 실행 조인포인트를 매치하려면 맨 뒤에(+)를 덧붙인다.

  * `within(ArithmeticCalculator+)`

* 위에서 만든 커스텀 애노테이션 @LoggingRequired는 클래스/메서드 레벨에 적용 가능하다.

* 다음 @Pointcut에 within키워드로 @LoggingRequired를 붙인 모든 클래스/메서드의 조인포인트를 매치할 수 있다.

  ```java
  @Pointcut("within(io.namjune.springrecipes.calculator.LoggingRequired)")
  public void loggingOperation() {}
  ```

### 포인트컷 표현식 조합하기

* AspectJ 포인트컷 표현식은 **, ||, ! 등의 연산자로 조합할 수 있다.

* 다름 포인트 컷은 ArithmeticCalculator 또는 UnitCalculator 인터페이스를 구현한 클래스의 조인포인트를 매치한다.

  `within(ArithmeticCalculator+) || within(UnitCalculator+)`

* 다른 포인트컷을 가리키는 레퍼런스 모두 연산자로 묶을 수 있다. 아래처럼 메서드로 표현한 포인트컷 표현식도 조합할 수 있다 

  ```java
  @Aspectt
  @Component
  public class CalculatorPointCuts {
    
    @Pointcut("within(ArithmeticCalculator+)")
    public void arithmeticOperation() {}
    
    @Pointcut("within(UnitCalcunator+)")
    public void unitOperation() {}
    
    @Pointcut("arithmeticOperation() || unitOperation()")
    public void loggingOperation() {}
  }
  ```

### 포인트컷 매개변수 선언하기

* 조인포인트 정보는 어드바이스 메서드에서 JoinPoint형 인수를 사용해서 리플렉션으로 액세스 할 수 있다.

* 이외에도 몇가지 특수한 포인트컷 표현식을 쓰면 선언적인 방법으로 조인포인트 정보를 얻을 수도 있다.

* target()과 args()로 현재 조인포인트의 대상 객체 및 인수값을 포착하면 포인트컷 매개변수로 빼낼 수 있다. 

* 이 매개변수는 자신과 이름이 똑같은 어드바이스 메서드의 인수로 전달한다.

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    
    @Before("execution(* *.*(..)) && target(target) && args(a, b)")
    public void logParameter(Object target, double a, double b) {
      log.info("Target class : {}", target.getClass().getName());
      log.info("Arguments : {}, {}", a, b);
    }
  }
  ```

* 독립적인 포인트컷을 선언해서 재사용할 경우에는 포인트컷 메서드의 인수 목록에도 함께 넣는다.

  ```java
  @Aspect
  public class CalculatorPointcuts {
    ...
      
    @Pointcut("execution(* *.*(..)) && target(target) && args(a, b)")
    public void parameterPointcut(Object target, double a, double b) {}
  }
  ```

* 위와 같이 매개변수화한 포인트컷을 참조하는 모든 어드바이스는 같은 이름의 메서드 인수를 선언해서 포인트컷 매개변수를 참조할 수 있다.

  ```java
  @Aspect
  @Component
  public class CalculatorLoggingAspect {
    ...
      
    @Before("CalculatorPointcuts.parameterPointcut(target, a, b)")
    public void logParameter(Object target, double a, double b) {
      log.info("Target class : {}", target.getClass().getName());
      log.info("Arguments : {}, {}", a, b);
    }
  }
  ```

## 2-18. 인트로덕션을 이용해 POJO에 기능 더하기

* 인트로덕션(introduction)은 AOP 어드바이스의 특별한 타입이다.

* 객체가 어떤 인터페이스의 구현클래스를 공급받아서 동적으로 인터페이스를 구현하는 기술이다.

* 게다가 여러 구현클래스를 지닌 여러 인터페이스를 동시에 인트로듀스할 수 있어서, 사실상 다중상속도 가능하다.

* 아래의 두 인터페이스가 존재한다.

  ```java
  public interface MaxCalculator {
    public double max(double a, double b);
  }
  
  public interface MinCalculator {
    public double min(double a, double b);
  }
  
  public class MaxCalculatorImpl implements MaxCalculator {
    ...
  }
  
  public class MinCalculatorImpl implements MinCalculator {
    ...
  }
  ```

* 두 인터페이스를 구현해서 어느 한 클래스에서 둘을 상속받아서 max(), min()을 동시에 사용할 수 는 없다. 자바는 다중상속이 안되기 때문에다.

* 구현 코드를 복붙하던지, 하나는 인터페이스 하나는 추상클래스로 구현하는 방법 뿐인데,

* 이때 인트로덕션 사용하면 인터페이스 둘 다 동적으로 구현한 것처럼 구현클래스 MaxCalculatorImpl과 MinCalculatorImpl을 이용할 수 있다. 다중 상속한것 처럼 쓸 수 있는 것이다.

  * 이게 어떻게 될까? 기본적으로 스프링은 인터페이스 기반의 JDK 동적(dynamic) 프록시를 생성하여 AOP를 적용한다.
  * 인트로덕션은 동적 프록시에 인터페이스를 추가하는 방식으로 작동한다.
  * 이 인터페이스에 선언된 메서드를 프록시 객체에서 호출하면 프록시는 백엔드 구현클래스에 처리를 위임한다. 

* 인트로덕션 역시 어드바이스 처럼 애스펙트 안에서 필드에 @DeclareParents를 붙여 선언한다.

  * 인트로덕션 대상 클래스는 value로 지정하며, 이 애너테이션을 붙인 필드형에 따라 들여올 인터페이스가 결정된다.
  * 이 인터페이스에서 사용할 구현 클래스는 defaultImpl에 명시한다.

  ```java
  @Aspect
  @Component
  public class CalculatorIntroduction {
    
    @DeclareParents(
    	value = "io.namjune.springrecipes.calculator.ArithmeticCalculatorImpl",
      defalutImpl = "MaxCalculatorImpl"
    )
    public MaxCalculator maxCalculator;
    
    @DeclareParents(
    	value = "io.namjune.springrecipes.calculator.ArithmeticCalculatorImpl",
      defalutImpl = "MinCalculatorImpl"
    )
    public MinCalculator minCalculator;
  }
  ```

* 이렇게 설정한 두 인트로덕션을 사용해서 ArithmeticCalculatorImpl 클래스로 두 인터페이스를 동적으로 들여올 수 있다.

* MaxCalculator와 MinCalculator 인터페이스 두개를 ArithmeticCalculatorImpl에 들여왔으면, 해당 인터페이스로 캐스팅하고, min(), max()를 활용할 수 있다.

  ```java
  public class Main {
    public static void main(String[] args) {
      ...
      ArithmeticCalculator arithmeticCalculator = (ArithmeticCalculator) context.getBean("arithmeticCalculator");
      ...
      MaxCalculator maxCalculator = (MaxCalculator) arithmeticCalculator;
      maxCalculator.max(1, 2);
      
      MinCalculator minCalculator = (MinCalculator) arithmeticCalculator;
      minCalculator.max(1, 2)
    }
  }
  ```

## 2-19. AOP 인트로덕션을 이용해서 POJO에 상태 추가하기

* 기존 객체에 새로운 상태를 추가해서 호출 횟수, 최종 수정 일자 등 사용 내역을 파악하고 싶은 경우가 있다

* 모든 객체가 동일한 베이스 클래스를 상속하는 건 해결책이 될 순 없다. 레이어가 다른 구조가 다른 여러 클래스에 상태를 추가하기란 더 어렵다.

* 원본 클래스에는 호출 횟수를 담을 카운트 필드가 없기 때문에, 구현클래스를 스프링 AOP 인트로덕션으로 공급 받아서 처리한다.

* 카운터 인터페이스 작성

  ```java
  public interface Counter {
    public void increase();
    public int getCount();
  }
  ```

* 구현 클래스 생성, 호출 횟수는 count 필드에 저장

  ```java
  public class CounterImpl implements Counter {
    private int count;
    
    @Override
    public void increase() {
      count++
    }
    
    @Override
    public int getCount() {
      return count;
    }
  }
  ```

* Counter 인터페이스를 CounterImpl로 구현한 다음, 모든 Calculator 객체에 공급하기 위해 다음과 같이 타입 매치 표현식을 통해서 인트로덕션을 적용한다. 

  * Calculator 인터페이스의 구현체들(*CalculatorImpl)에게 Counter 인터페이스의 구현체인 CounterImpl을 공급 한다.

  ```java
  @Aspect
  @Component
  public class CalculatorIntroduction {
    ...
    
    @DeclareParents(
      value = "io.namjune.springrecipes.calculator.*CalculatorImpl",
      defaultImpl = CounterImpl.class)
    public Counter counter;
  }
  ```

* 위에서 인터페이스를 동적으로 구현 한 뒤, 호출 횟수를 기록하기 위해서 계산기 메서드를 한번씩 호출할 때마다 counter 값을 하나씩 증가시키려면, 아래 코드처럼 After 어드바이스를 적용해야 한다.

  * 그리고, Counter 인터페이스를 구현한 객체는 프록시가 유일하므로 반드시 현재 조인포인트의 대상객체는 target이 아닌 this 객체를 가져와서 사용해야 한다.

    ```java
    @Aspect
    @Component
    public class CalculatorIntroduction {
      ...
      
      @After("execution(* io.namjune.springrecipes.calculator.*Calculator.*(..))"
            + " && this(counter)")
      public void increaseCouunt(Counter counter) {
        counter.increase();
      }
    }
    ```

* Main 클래스에서 각 Calculator 객체를 Counter형으로 캐스팅해서 호출 횟수를 얻을 수 있다.

  ```java
  public class Main {
    public static void main(String[] args) {
      ...
      ArithmeticCalculator arithmeticCalculator = (ArithmeticCalculator) context.getBean("arithmeticCalculator");
      
      UnitCalculator unitCalculator = (UnitCalculator) context.getBean("unitCalculator");
      ...
      
      Counter arithmeticCounter = (Counter) arithmeticCalculator;
      System.out.println(arithmeticCounter.getCount());
      
      Counter unitCounter = (Counter) unitCalculator;
      System.out.println(unitCounter.getCount());
    }
  }
  ```

## 2-20. AspectJ 애스펙트를 로드 타임 위빙하기

* 스프링 AOP 프레임워크는 제한된 타입의 AspectJ 포인트컷만 지원하며 **IoC 컨테이너에 선언한 빈에 한하여 애스펙트를 적용할 수 있다.**
* 따라서, 포인트컷 타입을 추가하거나, IoC 컨테이너 외부에 있는 객체에 애스펙트를 적용하려면 스프링 애플리케이션에서 AspectJ 프레임워크를 직접 끌어쓰는 수 밖에 없다.
* 위빙(Weaving)은 애스펙트를 대상 객체에 적용하는 과정을 말한다.
* **스프링 AOP는 런타임에 동적 프록시를 활용해서 위빙**하는 반면, AspectJ 프레임워크는 **컴파일 타임 위빙**, **로드 타임 위빙**을 모두 지원한다.
  * AspectJ에서 컴파일 타임 위빙은 ajc라는 전용 컴파일러가 담당한다. 컴파일 타임 위빙은 애스펙트를 자바 소스파일에 엮고 위빙된 바이너리 클래스 파일을 결과물로 내놓는다. 이미 컴파일된 클래스 파일이나 JAR 파일 안에도 애스펙트를 욱여 넣을 수 있다. 이를 **포스트 컴파일 타임(컴파일 이후에 적용되는) 위빙** 이라고 한다.
    * **컴파일 타임 위빙, 포스트 컴파일 타임 위빙 모두 클래스를 IoC 컨테이너에 선언하기 이전에 수행할 수 있으며, 스프링은 이 위빙 과정에 전혀 관여하지 않는다.**
  * AspectJ 로드타임 위빙은 JVM이 클래스 로더를 이용해 대상 클래스를 로드하는 시점에 일어난다. 바이트코드에 코드를 넣어 클래스를 위빙하려면 특수한 클래스가 필요한데, AspectJ와 스프링 둘 다 클래스 로더에 위빙 기능을 부여한 로드 타임 위버를 제곤한다. 로드 타임 위버는 간단한 설정으로 바로 사용할 수 있다.

## 2-21. 스프링에서 커스텀 AspectJ 애스펙트 구성하기

* AspectJ 프레임워크가 사용하는 애스펙트는 이 프레임워크가 자체적으로 인스턴스화하기 때문에 **원하는 대로 구성하려면 AspectJ 프레임워크에서 인스턴스를 가져와야 한다.**

* 모든 AspectJ 애스펙트에는 Aspects라는 팩토리 클래스가 있고 이 클래스의 정적 팩토리 메서드 aspectOf()를 호출하면 현재 애스펙트 인스턴스를 액세스할 수 있다. 원하는 대로 애스펙트를 구성하려면 IoC 컨테이너에서 빈을 만들때Aspects.aspectOf(ComplexCachingAspect.class)를 호출해서 빈을 선언한다.

* 그런 다음 AspectJ 위버로 애플리케이션을 실행해보자.

  ```java
  @Configuration
  @ConponentScan
  public class CalculatorConfiguration {
    
    @Bean
    public ComplexCachingAspect complexCachingAspect() {
      Map<String, Complex> cache = new HashMap<>();
      cache.put("2,3", new Complex(2, 3));
      cache.put("3,5", new Complex(3, 5));
      
      ComplexCachingAspect complexCachingAspect = Aspects.aspectOf(ComplexCachingAspect.class);
      complexCachingAspect.setCache(cache);
      return complexCachingAspect;
    }
  }
  ```

## 2-22. AOP를 이용해 POJO를 도메인 객체에 주입하기

* 스프링의 의존체 주입 능력 덕분에 IoC 컨테이너에 선언된 빈들은 서로 연결할 수 있지만, 컨테이너 밖에서 만든 객체는 스프링 빈과 연결할 방법이 마땅치 않다. 코드를 수동으로 작성해 연결할 수밖에 없다.
* IoC 컨테이너 외부에서 생성된 객체는 대부분 도메인 객체로, new 연산자 또는 DB 쿼리 겨로가로 생긴다. 이렇게 스프링 밖에서 만든 도메인 객체 안에 스프링 빈을 주입하려면 AOP의 도움이 절실하다.
* 사실 스프링 빈을 주입하는 것도 공통 관심사 중 하나이다.
* 도메인 객체는 스프링이 만든 객체가 아니어서 스프링 AOP로는 주입할 수 없다.
* 따라서 이런 용도에 알맞게 스프링이 제공하는 AspectJ 애스펙트를 AspectJ 프레임워크에서 가져다 쓰면 된다.

* 필요시 추가 학습 필요.

## 2-23. 스프링 TaskExecutor로 동시성 적용하기

* Thread 기반의 동시성(Concurrent) 프로그램을 스프링으로 개발하고 싶은데, 이렇다 할 표준 가이드가 없어서 어떻게 접근해야 할 지 모를 경우 

  * 스프링 TaskExecutor(작업 실행기) 추상체가 정답이다. TaskExecutor는 기본 자바의 Executor, CommonJ의 WorkManager 등 다양한 구현체를 제공하며 필요하면 커스텀 구현체를 만들어 쓸 수도 있다.
  * 스프링은 이들 구현체를 모두 자바 Executor 인터페이스로 단일화 했다. 

* 자바 SE가 제공하는 표준 스레드를 이용해서 개발하다 보면 자칫 지겹고 난해한 늪에 빠지기 쉽다. 동시성은 서버사이드 컴포넌트의 아주 중요한 요건이지만 엔터프라이즈 자바 세계에는 딱히 표준이라 할 만한 것이 없다.

* 스레드를 명시적으로 생성하거나 조작하는 행위를 금지하는 내용이 일부 자바 EE 명세에 포함되어 있을 따름이다.

* 자바 기술자들은 오래 전부터 스레드 및 동시성을 다루기 위해 여러 가지 궁리를 했다. 우선 초기 JDK 1.0 시절부터 표준 java.lang.Thread가 있었고, 작업을 주기적으로 실행시킬 수 있는 TimerTask가 1.3 버전에 등장했다. 

* 이어서 자바 5부터는 java.util.concurrent 패키지를 선보였고, 이와 동시에 java.util.concurrent.Executor를 중심으로 Thread Pool을 생성하는 체계가 완전히 개편 되었다.

* Executor API는 의외로 간단하다.

  ```java
  package java.util.concurrent;
  
  public inferface Executor {
    void execute(Runnable command);
  }
  ```

* 스레드 관리 기능이 강화된 ExecutorService 이하 인터페이스는 shutdown()처럼 스레드에 이벤트를 일으키는 메서드를 제공한다. 자바 5부터 여러 구현 클래스가 추가됐다. 재부분 java.util.concurrent 패키지의 정적 팩토리 메서드를 사용해 쓸 수 있다. 자바 SE 클래스를 응용한 예제를 몇 가지 살펴보자.

* **ExecutorService 클래스에는 Future\<T\> 형 객체를 반환하는 submit() 메서드가 있다**. **Future\<T\> 인스턴스는 대개 비동기 실행 스레드의 진행 상황을 추적하는 용도로 쓰인다. Future.idDone(), Future.isCalcelled() 메서드는 각각 어떤 잡이 완료되었는지, 취소되었는지 확인한다.**

* **run() 메서드가 반환형 없는 Runnable 인스턴스 내부에서** **ExecutorService와 submit()을 사용할 경우**, **반환된 Future의 get() 메서드를 호출하면 null 또는 전송시 지정한 값(다음 코드는 Boolean.TRUE)이 반환된다.**

  ```java
  public static void main(String[] args) {
    Runnable task = new Runnable() {
      public void run() {
        try {
          Thread.sleep(1000 * 60);
          System.out.println("Done sleeping for a minute, returning! ");
        } catch(Exception e) {
          ...
        }
      }
    }
    
    ExecutorService executorService = Executors.newCachedThreadPool();
    
    if (executorService.submit(task, Boolean.TRUE).get().equals(Boolean.TRUE)) {
      System.out.println("Job has finished!");
    }
  }
  ```

* 위의 배경 지식 정도만 있으면 다양한 구현체마다 독특한 기능을 응용할 수 있다. 다음은 Runnable을 응용해 시간 경과를 나타낸 클래스이다.

  ```java
  public class DemostrationRunnable implements Runnable {
    
    @Override
    public void run() {
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(Thread.currentThread().getName());
      System.out.printf("Hello %s \n", new Date);
    }
  }
  ```

* 위의 DemostrationRunnable 인스턴스를 자바 Executors와 스프링 TaskExecutor에서 사용해보자.

  * 각 ExecutorService의 submit(task)을 호출하고, get()을 호출했을때(인자를 넘겨주지 않았으므로) null을 반환한다.
  * submit(task, Boolean.TRUE)를 호출했다면, get() 호출시 Boolean.TRUE가 반환 된다.
  * 그러면 작업이 완료라고 판단하고 log를 찍는다.

  ```java
  public class ExecutorDemo {
    public static void main(String[] args) throws Throwable {
      Runnable task = new DemostrationRunnable();
      
      ExecutorService cachedThreadPoolExecutorService = Executors.newCachedThreadPool();
      if (cachedThreadPoolExecutorService.submit(task).get() == null) {
        System.out.printf("The cachedThreadPoolExecutorService " 
                          + "has succeeded at %s \n", new Date());
      }
      
      ExecutorService fixedThreadPool = Executors.newFixedThreadPool(100);
      if (fixedThreadPool.submit(task).get() == null) {
        System.out.printf("fixedThreadPool has succeeded at %s \n", new Date());
      }
      
      ExecutorService es = Executors.newCachedThreadPool();
      if (es.submit(tash, Boolean.TRUE).get().equals(Boolean.TRUE)) {
        System.out.println("Job has finished!");
      }
      
      ScheduledExecutorService scheduledThreadExecutorService = Executors.newScheduledThreadPool(10);
      if (scheduledThreadExecutorService.schedule(task, 30, TimeUnit.SECONDS)
          .get() == null) {
        System.out.printf("The scheduledThreadExecutorService " 
                          + "has succeeded at %s \n", new Date());
      }
      
      scheduledThreadExecutorService.scheduledAtFixedRate(task, 0, 5, TimeUnit.SECONDS);
    }
  }
  ```

  * ExecutorService.submit(task).get()이 null을 반환하는 이유는 하위 인터페이스에서 Callable\<T\> 를 인수로 받는 submit() 메서드를 호출하면 Callable의 call() 메서드가 반환한 값을 그대로 반환하기 때문이다.
  * ExecutorService.submit(task, Boolean.TRUE).get()이 TRUE를 반환하는 이유는  Runnable과 T를 받아서 T를 반환하기 때문이다.

* 스프링은 자바 5의 java.util.concurrent.Executor를 상속한 org.springframeworks.core.task.TaskExecutor 인터페이스라는 통합 솔루션을 제공한다.

* 사실 TaskExecutor 인터페이스는 스프링 프레임워크 내부에서 두루 쓰인다.(스레드를 지원하는) 스프링 쿼츠 연계 및 메세지 드리븐 POJO 컨테이너 지원 기능 등에도 TaskExecutor가 쓰일 정도로 응용 범위가 넓다.

  ```java
  package org.springframework.core.task;
  
  public interface TaskExecutor extends Executor {
    void execute(Runnable task);
  }
  ```

* 스레드 프로그래밍의 기본과 ThreadPoolTaskExecutor에 대하여 공부하자!

  * [https://github.com/HomoEfficio/dev-tips/blob/master/Java-Spring%20Thread%20Programming%20%EA%B0%84%EB%8B%A8%20%EC%A0%95%EB%A6%AC.md](https://github.com/HomoEfficio/dev-tips/blob/master/Java-Spring Thread Programming 간단 정리.md)

## 2-24. POJO끼리 애플리케이션 이벤트 주고받기

* POJO들이 서로 통신할 때에는 대부분 sender가 receiver를 찾아 그 메서드를 호출한다. 이처럼 sender가 반드시 receiver를 임지해야 하는 구조는 단순하고 직접적은 통신이 가능하지만 두개의 POJO가 단단하게 결합할 수 밖에 없다.
* 스프링 애플리케이션 컨텍스트는 빈 간의 이벤트 기반 통신을 지원한다. 이벤트 기반 통신 모델에서는 실제로 receiver가 여럿 존재할 가능성이 있기 때문에 sender는 누가 수신할지 모른채 이벤트를 발행한다.
* receiver 역시 누가 이벤트를 발행했는지 알 필요 없고, 여러 sender가 발행한 여러 이벤트를 리스닝할 수도 있다. 이런식으로 sender와 receiver를 느슨하게 엮는다.
* 커스텀 ApplicationEvent를 만들어 발행한 다름, 이 이벤트를 받고 어떤 일을 하는 컴포넌트를 작성해보자.

### ApplicationEvent로 이벤트 정의하기

* 이벤트 기반 통신을 하려면 제일 먼저 이벤트 자체를 정의해야 한다.

* 쇼핑카트를 체크아웃 하면 Casher 빈이 체크아웃 시각이 기록된 CheckoutEvent를 발행한다고 가정하자.

  ```java
  public class CheckoutEvent extends ApplicationEvent {
    private final ShoppingCart cart;
    private final Date time;
    
    public CheckoutEvent(ShoppingCart cart, Date time) {
      super(cart);
      this.cart = cart;
      this.time = time;
    }
    
    ...(getters)
  }
  ```

### 이벤트 발행하기

* 이벤트를 인스턴스화 한다음 애플리케이션 이벤트 발행기에서 publishEvent() 메서드를 호출하면 이벤트가 발행 된다.

  ```java
  public class Cashier {
    ...
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;
    
    public void checkout(ShoppingCart cart) throws IOException {
      ...
        
      CheckoutEvent event = new CheckoutEvent(cart, new Date());
      applicationEventPublisher.publishEvent(event);
    }
  }
  ```

### 이벤트 리스닝하기

* ApplicationListener 인터페이스를 구현한 애플리케이션 컨텍스트에 정의된 빈은 타입 매개 변구에 매치되는 이벤트를 모두 알림받는다.

* 이런식으로 ApplicationContextEvent 같은 특정 그룹의 이벤트들을 리스닝한다.

  ```java
  @Component
  public class CheckoutListener implements ApplicationListener<CheckoutEvent> {
    
    @Override
    public void onApplicationEvent(CheckoutEvent event) {
      // 체크아웃 시각을 통해서 처리할 비즈니스로직 구현.
      System.out.println("Checkout Event [" + event.getTime() + "]")
    }
  }
  ```

* 스프링 4.2부터는 @EventListener 애너테이션을 붙여도 이벤트 리스너로 만들 수 있다.

  ```java
  @Component
  public class CheckoutListener {
    
    @EventListener
    public void onApplicationEvent(CheckoutEvent event) {
      // 체크아웃 시각을 통해서 처리할 비즈니스로직 구현.
      System.out.println("Checkout Event [" + event.getTime() + "]")
    }
  }
  ```

### 애플리케이션 컨텍스트에 전체 이벤트를 리스닝할 리스너 등록

* 등록 절차는 매우 간단하다.
* 리스너의 빈 인스턴스를 선언하거나, 컴포넌트 스캐닝으로 감지하면 된다.
* 애플리케이션 컨텍스트는 ApplicationListener 인터페이스를 구현한 빈과 @EventListener가 붙은 메서드를 가진 빈을 인지해서 이들이 관심있는 이벤트를 각각 통지한다.
* @EventListener의 큰 장점은 ApplicationListener\<CheckoutEvent\> 를 상속받지 않아도 되므로, 애초에 CheckoutEvent가 ApplicationEvent를 상속받지 않아도 되게 할 수 있다는 것이다.
* 이런 방식으로 이벤트 객체를 스프링 프레임워크에 종속된 클래스가 아닌 평범한 POJO로 되살릴 수 있다.
* 더불어 ApplicationContext 자신도 ContextClosedEvent, ContextRefreshedEvent, RequestHandledEvent 등 컨테이너 이벤트를 발행한다. 이떤 빈이든 이런 이벤트를 받고 싶다면 ApplicationListener 인터페이스를 구현하면 된다.

