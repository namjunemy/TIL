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