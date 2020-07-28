# Tips

> 개발하면서 마주한 팁들

## Java

### Enum

* [Java enum의 사용 - 이종립님](https://johngrib.github.io/wiki/java-enum/)
* [이펙티브 자바 38 - 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라](https://jaehun2841.github.io/2019/02/04/effective-java-item38/)

* Enum을 확장해서 사용하려면 interface를 상속 받아서 구현할 수 있다.

* Enum 클래스가 내부에서 기본적으로 Enum\<T\> 를 상속받고 있기 때문에 다른 클래스를 extend 할 수 없다.

* Enum 상수 하나당 인스턴스가 만들어지며, 각각 public static final 이다.

* ENUM 타입별로 다르게 동작하는 코드를 구현하고 싶다면, 아래와 같은 방식이 있다.

  ```java
  public enum Operation {
      PLUS {
          @Override
          public double apply(double x, double y) { return x + y; }
      },
      MINUS {
        	@Override
          public double apply(double x, double y) { return x - y; }
      },
      TIMES {
        	@Override
          public double apply(double x, double y) { return x * y; }
      },
      DIVIDE {
        	@Override
          public double apply(double x, double y) { return x / y; }
      };
    
      public abstract double apply(double x, double y);
  }
  ```

* 예를 들어, VOD 또는 IMG가 저장될 수 있는 Storage 엔티티에서 FileType에 따라서 다른 행동을 구현하고 싶었을 때 위와 같이 해결 할 수 있다.

### 일급컬렉션

* https://jojoldu.tistory.com/412
  * **비지니스에 종속적인 자료구조**가 만들어져, 이후 발생할 문제가 최소화 됨
  * 일급 컬렉션은 **컬렉션의 불변을 보장**
    * java의 final은 재할당만 금지함. 따라서 list.add() 또는 map.put() 등이 가능함.
    * 일급 컬렉션 또는 래퍼 클래스로 만들어서 생성자와 get만 열어 둔다면, 불변을 보장할 수 있음.
  * **값과 로직이 함께 존재**
    * 해당 컬렉션과 그 컬렉션으로 하는 계산 로직은 서로 관계가 있는데,
    * 리스트에 데이터를 담고, 서비스 혹은 util 메서드에서 리스트 데이터를 가져와서 수행함.
    * 이렇게 되면 같은 리스트를 쓰는 여러 개발자에 의해 같은 기능을 하는 메서드를 중복 생성할 수 있는 여지가 생김.
    * 코드가 흩어질 확률이 높음.
    * 따라서, 일급 컬렉션 안에 리스트와 계산식을 같이 두면, 상태와 로직이 한곳에서 관리 됨.
  * 일급컬렉션의 이름으로 명확한 식별에 도움(글 참조)

### Effective java

* https://jaehun2841.github.io/tags/Effective-Java/

* https://github.com/ryudung/effective-java3-E
* [https://javabom.tistory.com/category/Reading%20Record/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94](https://javabom.tistory.com/category/Reading Record/이펙티브자바)
* [https://medium.com/@ddt1984/effective-java-3-e-%EC%A0%95%EB%A6%AC-c3fb43eec9d2](https://medium.com/@ddt1984/effective-java-3-e-정리-c3fb43eec9d2)
* [https://wonsohana.wordpress.com/2019/06/23/effective-java-3-e-%EC%A0%95%EB%A6%AC1/](https://wonsohana.wordpress.com/2019/06/23/effective-java-3-e-정리1/)

### 커스텀 컬렉터로 스트림 파티셔닝, Map에 담기

```java
@DisplayName("스트림 파티셔닝 테스트")
public class StreamPartitioningTest {

    @Test
    @Order(1)
    @DisplayName("소수와 비소수로 분할하기")
    public void partition01() {
        int candidate = 10;
        Map<Boolean, List<Integer>> collect = IntStream.rangeClosed(2, candidate)
            .boxed()
            .collect(Collectors.partitioningBy(this::isPrime));

        int nonPrime = collect.get(false).size();
        int prime = collect.get(true).size();

        assertThat(nonPrime).isEqualTo(5); //[4, 6, 8, 9, 10]
        assertThat(prime).isEqualTo(4);  //[2, 3, 5, 7]
    }

    private boolean isPrime(int candidate) {
        // 소수의 대상을 주어진 수의 제곱근 이하의 수로 제한 할 수 있음.
        int candidateRoot = (int) Math.sqrt(candidate);
        return IntStream.rangeClosed(2, candidateRoot)
            .noneMatch(i -> candidate % i == 0);
    }
}
```

### Primitive Type vs Wapper class

* 개발하다보면 long 과 Long을 혼용해서 쓸 때가 있다. 다른 primitive 타입도 마찬가지다.

* 나름대로 고민을 가지고 있었는데, 지금까지 알고 있는 것을 정리해 놓고 계속해서 이유를 추가하자.

* primitive type

  * 일반적인 경우에 primitive 타입을 사용한다. 기본값이 있기 떄문에 null이 존재하지 않는다.
  * 별도의 객체를 생성하지 않기 때문에 메모리 효율성을 조금 챙길 수 있다.
  * 값의 비교가 ==으로 직관적으로 비교가 가능하다.

* wapper class

  * null을 반환할 여지가 있을 경우 유용하다.

  * 객체로 필요할 경우 사용한다. 제네릭과 stream 등.

  * 대신에 객체의 == 비교는 주의해야한다.

    ```java
    Integer a = 127;
    Integer b = 127;
    Integer c = 128;
    Integer d = 128;
    
    System.out.println(a == b); //true
    Ststem.out.println(c == d); //false
    ```

    * c == d가 false인 이유는?
      * Integer.class의 안을 까보면 IntegerCache라는 static innerClass로 -128 ~ 127 범위의 값들을 caching하고 있다.
      * 127까지는 같은 객체지만, 128부터는 새로 생성된 Integer 객체이다.

## Kafka

* Spring-kafka의 KafkaTemplate을 통해 카프카 메세지 전송 하면 ListenableFuture를 반환하는데,

* 퓨처에 콜백 리스너를 등록할 수 있다.

* 전송 후 성공/실패에 따른 동작을 구현할 수 있다.

  ```java
  Message<T> message = MessageBuilder
    .withPayload(data)
    .setHeader(KafkaHeaders.TOPIC, topic)
    .build();
  
  ListenableFuture<SendResult<String, CustomMessage>> future = kafkaTemplate.send(message);
  
  future.addCallback(new ListenableFutureCallback<SendResult<String, CustomMessage>>() {
    @Override
    public void onFailure(Throwable ex) {
      log.debug("fail");
    }
  
    @Override
    public void onSuccess(SendResult<String, CustomMessage> result) {
      log.debug("success");
    }
  });
  
  ```

## Spring

### 팩토리메서드 패턴을 이용해서 스프링에서 빈 동적으로 처리하기

* 스프링에서는 Bean을 컬렉션으로 주입할 수 있다. 하나의 인터페이스를 상속받고 있는 구현체인 Bean들을 List로 주입받을 수 있다.
* 이렇게 되면 같은 행동을 하는 다른 타입의 빈들을 모조리 주입받아서 if-else 또는 switch처리하지 않아도 된다.
* 빈의 의존객체들이 줄어들고, 분기 로직도 없앨 수 있다.
* 이 방식으로 SQS가 여러 타입의 메세지를 처리해야 하는 상황을 효율적으로 처리할 수 있었다.
  * 이 과정 블로그 포스팅해보자!
* https://zorba91.tistory.com/306

### Lombok @RequiredArgsConstructor와 @Qualifier 동시 사용 주의

* Lombok의 @RequiredArgsConstructor를 붙이고, Spring의 @Qualifier를 사용하여 주입되는 빈을 지정해주지만, 빈이 여러개 찾아졌다는 에러가 발생할 수 있다.
* 원인은 @RequiredArgsConstructor가 생성자를 만들때 인자에 @Qualifier를 넣어주지 않기 때문이다.
* 인스턴스 이름을 변경하거나, Lombok config를 변경해서 사용해야 한다.
  * https://www.podo-dev.com/blogs/224

### SpringBoot 2.0 부터 @ConfigurationProperties(prefix = "camelCase") 주의

* 2.0부터는 **prefix 인식**을 **kebab-case** 또는 **소문자**만 인식한다.
* underscore나 camelCase는 인식하지 못하니 주의하자.
* 하지만 하위 속성인 요소들은 기존과 동일하게 모든 케이스 다 가능하다.

### jackson 으로 json string -> Object 변환시 snake_case to camelCase

* RestTemplate으로 외부 연동 API 응답 값을 받을때 굉장히 다양한 case convention을 경험할 수 있다.

* 하지만 restTemplate에 기본적으로 등록되어 있는 messageConverters에서는 snake_case를 제대로 매핑하지 못해서 null이 들어가곤 한다.

* 해결하려면 restTemplate에 Spring이 이미 등록한 기본값보다 우선하도록 새롭게 커스텀한 messageConverter를 높은 우선순위에 등록하자.

* jackson version 2.7 이전에서는 PropertyNamingStrategy. CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES 를 이후 버전에서는 PropertyNamingStrategy.SNAKE_CASE 를 사용해야 한다.

  ```java
  // AdapterConfig.java
  
  private ObjectMapper createCustomObjectMapper() {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.setPropertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE);
    return objectMapper;
  }
  
  private MappingJackson2HttpMessageConverter createMappingJackson2HttpMessageConverter() {
    MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
    converter.setObjectMapper(createCustomObjectMapper());
    return converter;
  }
  
  @Bean(name = "restTemplate")
  public RestTemplate restTemplate() {
    RestTemplate restTemplate = new RestTemplate(clientHttpRequestFactory());
    restTemplate.getMessageConverters().add(0, createMappingJackson2HttpMessageConverter());
    return restTemplate;
  }
  ```

### MySQL Character Set utf8mb4 설정 후 JDBC Url

* connector 5.1.47 이후 버전으로 업데이트 후
  * 5.1.47 이후버전에서 characterEncoding UTF-8이 mysql character set utf8mb4로 매핑.
  * jdbc:mysql://localhost:3306/customersdb?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    * useUnicode는 default true. 생략가능
  * https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-charsets.html
  * https://www.lesstif.com/dbms/mysql-jdbc-url-14090356.html
  * http://minsql.com/mysql/MySQLWithEmoji/

### Spring Boot에서 Gradle에 정의된 정보 가져오기.

* Boot의 application.yml에서 gradle에 정의된 정보들을 사용할 수 있다.

* gradle build 과정중에 processResources 단계에서 아래와 같은 설정을 해주면 된다.

  ```groovy
  ...
  
  processResources {
      filesMatching('**/application.yml') {
          expand(project.properties)
      }
  }
  
  ...
  ```

* https://nevercaution.github.io/spring-boot-use-gradle-value/

### Collection 타입 @Vaild 검증하기

* [https://medium.com/chequer/valid-%EC%95%A0%EB%84%88%ED%85%8C%EC%9D%B4%EC%85%98%EC%9D%84-%EC%BB%AC%EB%A0%89%EC%85%98-%ED%83%80%EC%9E%85-requestbody%EC%97%90-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-6aef15bb8dff](https://medium.com/chequer/valid-애너테이션을-컬렉션-타입-requestbody에-사용하기-6aef15bb8dff)

### Bean Validation

* [https://medium.com/@gaemi/java-%EC%99%80-spring-%EC%9D%98-validation-b5191a113f5c](https://medium.com/@gaemi/java-와-spring-의-validation-b5191a113f5c)
* [https://cnpnote.tistory.com/entry/SPRING-Spring%EC%97%90%EC%84%9C-Valid%EC%99%80-Validated%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90](https://cnpnote.tistory.com/entry/SPRING-Spring에서-Valid와-Validated의-차이점)
* https://jongmin92.github.io/2019/11/18/Spring/bean-validation-1/
* https://jongmin92.github.io/2019/11/21/Spring/bean-validation-2/
* [https://javacan.tistory.com/tag/JSR%20380](https://javacan.tistory.com/tag/JSR 380)
* https://www.notion.so/Bean-Validation-2f70a3f0aae94621886487477097abfa

## DB

### DB 스키마 가이드

* Indexes

  * 인덱스 명명 규칙은 "테이블명" + "인덱스 약어" + "컬럼명"으로 한다.

    ```sql
    CREATE TABLE person(
      id bigserial PRIMARY KEY,
      email text NOT NULL,
      first_name text NOT NULL,
      last_name text NOT NULL,
      CONSTRAINT person_ck_email_lower_case CHECK(email = LOWER(email))
    );
    
    CREATE INDEX person_ix_first_name_last_name ON person (first_name, last_name);
    ```

  * 인덱스 타입별 약어

    * Primary Key : pk
    * Index : ix
    * Check Key : ck
    * Unique Key : uk
    * Foreign Key : fk

## JPA

### 순환참조 해결 방법과 엔티티 직접반환 금지.

* ORM은 양방향 참조를 필요로 한다. 그래서 컨트롤러에서 응답값에 엔티티를 절대로 직접 반환하지 말자.
* JSON 생성 라이브러리가 양방향 관계의 엔티티를 JSON으로 Serialize하는 순간 서로의 toString()을 무한호출한다.
* 또 하나, 엔티티는 충분히 변경의 여지가 있는데, 엔티티 변경시 API 스펙 자체가 변경되므로 엔티티 직접반환은 실무에선 하지 않는 편이 좋다.
* 웬만하면 DTO로 만들어서 반환하자.
* **순환참조 해결 방법**
  * @JsonIgnore를 사용하는 방법
    * 실제로 property에 null을 할당하는 방식.
  * @JsonManagedReference와 @JsonBackReference를 사용하는 방법
    * 부모클래스 측에 @JsonManagedReference를, 자식 측에 @JsonBackReference 어노테이션을 추가해주면 된다.
    * https://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion
    * https://pasudo123.tistory.com/350

### JPA AttributeConverter 로 도메인 <-> DB 변환

* ORM, JPA위에서 개발하다 보면 도메인 모델을 DB의 VARCHAR 타입 컬럼에 매핑 해야 할 일이 빈번하게 발생한다.

* 예를 들어, 카테고리 리스트를 DB의 VARCHAR컬럼에 ,로 joining 해서 집어넣거나( ,로 joining된 문자열을 잘라서 리스트에 매핑하는 반대의 경우도.)

* 어플리케이션에서 ENUM으로 관리하는 코드를 VARCHAR 컬럼에 매핑해야하는 경우도 마찬가지다.

* 엔티티 - Content.java

  ```java
  @Entity
  public class Content {
    ...
      
    @Convert(converter = ContentCategoryListConverter.class)
    private List<CategoryCode> caregories;
    
    ...
  }
  ```

* 컨버터 - ContentCategoryListConverter.java

  ```java
  @Converter
  public class ContentCategoryListConverter implements AttributeConverter<List<CategoryCode>, String> {
    private static final String CATEGORY_DELIMITER = ",";  // 구분자
    
    // 도메인 CategoryCode 리스트를 DB VARCHAR에 맵핑
    @Override
    public String convertToDatabaseColumn(List<CategoryCode> attribute) {
      if (CollectionUtils.isEmpty(attribute)) {
        return null;
      }
      return attribute.stream()
        .map(CategoryCote::getCode)
        .collect(Collectors.joining(CATEGORY_DELIMETER));
    }
    
    // DB 컬럼에서 도메인의 CategoryCode 리스트로 변환
    @Override
    public List<CategoryCode> convertToEntityAttributes(String dbData) {
      if (StringUtils.isEmpty(dbData)) {
        return new ArrayList<>();
      }
      
      String[] split = dbData.split(CATEGORY_DELEMITER);
      return Arrays.stream(split)
        .map(CategoryCode::getCategoryByCode)
        .collect(Collectors.toList());
    }
  }
  ```

### Spring-Data JPA Auditing 

* @Configuration 클래스에 @EnableJpaAuditing 어노테이션을 선언한다.

* @CreatedBy나 @LastModifiedBy로 엔티티의 creater, updater를 관리하는 것의 기본은 스프링 시큐리티의 Principal의 이름으로 채워진다. 이 값은 SecurityContext의 인증 인스턴스에서 가져온다.

  * 커스텀한 값을 채우고싶은 경우 AuditorAware\<T>를 구현할 수 있다.
  * 그리고 해당 빈 이름을 auditorAwareRef에 매개변수 값으로 지정한다.

  ```java
  @EnableJpaAuditing(auditorAwareRef = "auditorProvider")
  public class JpaAuditingConfig {
      
    private static final String UPDATER = "수정자 이름"
    
    @Bean("auditorProvider")
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.of(UPDATER)
    }
  }
  ```

* 해당 엔티티에 AuditingEntityListener를 등록한다. 그리고 스프링 데이터에서 제공하는 엔티티 리스너 클래스인 AuditingEntityListener를 지정한다.

* 그리고 위에서 등록해놓은 변경 author 감사 기능을 통해서 생성자와 수정자를 트래킹할 수 있다.

  ```java
  @Entity
  @EntityListeners(AuditingEntityListener.class)
  public class Content{
    
    ...
      
    @Column(name = "created_date", nullable = false, updatable = false)
    @CreatedDate
    private long createdDate;
    
    @Column(name = "modified_date" 
    @LastModifiedDate
    private long modifiedDate;
            
    ...
            
    @Column(name = "created_by")
    @CreatedBy
    private String createdBy;
   
    @Column(name = "modified_by")
    @LastModifiedBy
    private String modifiedBy;
  }
  ```

## 테스트

### @Spy, @Mock, @SpyBean, @MockBean, @InjectMock

* https://jojoldu.tistory.com/226?category=635883
* https://jojoldu.tistory.com/239?category=635883
* http://blog.devenjoy.com/?p=529
* [https://velog.io/@june0313/Mockito-Mock-%EB%A6%AC%EC%8A%A4%ED%8A%B8%EB%A5%BC-%EC%A3%BC%EC%9E%85%ED%95%98%EA%B3%A0-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%98%EA%B8%B0](https://velog.io/@june0313/Mockito-Mock-리스트를-주입하고-테스트-하기)

### 테스트 케이스 스터디 step - by beyondJ2EE

* 기본편
  * **Step1 : junit 과 테스트 방법에 대한 이해**
    * 스터디에 앞서서 junit란것에 대한 개론을 먼저 이해 합니다. ppt로 되어 있기 때문에 부담 없이  보실수 있을 겁니다.  [https://www.slideshare.net/tom.zimmermann/unit-testing-with-junit](https://www.google.com/url?q=https%3A%2F%2Fwww.slideshare.net%2Ftom.zimmermann%2Funit-testing-with-junit&sa=D&sntz=1&usg=AFQjCNF97psz39TRUC7qH6cASXL4i2rlmg) 
  * **Step2 : Test Tool이 무엇이 있는지 간략하게 보기** 
    * 테스트 툴들이 어떤것이 있는지 둘러봅니다. 참고로 testNG는 거의 사용을 안합니다.  
    * [https://dzone.com/articles/10-essential-testing-tools-for-java-developers](https://www.google.com/url?q=https%3A%2F%2Fdzone.com%2Farticles%2F10-essential-testing-tools-for-java-developers&sa=D&sntz=1&usg=AFQjCNFhxIahcutRuOY1u0IU9SpmMkmNYg)  
  * **Step3 : 예제코드를 보면서 Junit 익히기**  
    * 외국 서적 "Mastering Software Testing with JUnit 5"에 나온 예제 코드는 아래와 같습니다.  
    * [https://github.com/bonigarcia/mastering-junit5](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fbonigarcia%2Fmastering-junit5&sa=D&sntz=1&usg=AFQjCNHSJno9Ln04daahZ5kKl6-zoMZtZw) 
    * 본인들이 필요한 부분만 확인하고 필요없는 부분은 skip 하세요  
    * 해당 소스를 보면서 메뉴얼을 같이 참고 하세요 
    * Junit5 User Guide : [https://junit.org/junit5/docs/snapshot/user-guide/index.pdf](https://www.google.com/url?q=https%3A%2F%2Fjunit.org%2Fjunit5%2Fdocs%2Fsnapshot%2Fuser-guide%2Findex.pdf&sa=D&sntz=1&usg=AFQjCNG5ily-hn7FOS-qfEuMn4Q4jKkUWQ)
    * mockito 한글 소개 [https://github.com/mockito/mockito/wiki/Mockito-features-in-Korean](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fmockito%2Fmockito%2Fwiki%2FMockito-features-in-Korean&sa=D&sntz=1&usg=AFQjCNF1UeIeobYsHM0H0J56HOmR-UwrQQ)
    * mockito 메뉴얼[https://www.tutorialspoint.com/mockito/](https://www.google.com/url?q=https%3A%2F%2Fwww.tutorialspoint.com%2Fmockito%2F&sa=D&sntz=1&usg=AFQjCNHaLQlVVc_zNQhBYfrRYurDdGldzw)
      * Hamcrest 메뉴얼[https://www.baeldung.com/java-junit-hamcrest-guide](https://www.google.com/url?q=https%3A%2F%2Fwww.baeldung.com%2Fjava-junit-hamcrest-guide&sa=D&sntz=1&usg=AFQjCNEsKTOQyNKcZL1hFpI5h3br9UZQGw)  					

* 응용편
  * **Step4 : Spring Boot Test Quick Guide 한글 블로그**
    * springboot에서 제공하는 테스트 어노테이션이 어떤것들이 있는지 알아 봅니다.
    * [https://meetup.toast.com/posts/124](https://www.google.com/url?q=https%3A%2F%2Fmeetup.toast.com%2Fposts%2F124&sa=D&sntz=1&usg=AFQjCNEScz9RLJMTPUCsWBZbVsBmALHfPA)  
  * **Step5 : Controller Service Repository 테스트 코드** 
    * 비즈니스 로직을 위한 테스트 코드를 알아 봅니다.
    *  [https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-test/src/test/java/sample/test](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fspring-projects%2Fspring-boot%2Ftree%2Fmaster%2Fspring-boot-samples%2Fspring-boot-sample-test%2Fsrc%2Ftest%2Fjava%2Fsample%2Ftest&sa=D&sntz=1&usg=AFQjCNGDFBJtNP2EW2-x_m-fsyP6o_8eog)
    * https://www.baeldung.com/spring-boot-testing
    * https://cheese10yun.github.io/spring-boot-test/
  * Step6 : springboot 기타 (properties, json, web, mock, context, util) 테스트 코드 
    * [https://tuhrig.de/testing-configurationproperties-in-spring-boot/](https://www.google.com/url?q=https%3A%2F%2Ftuhrig.de%2Ftesting-configurationproperties-in-spring-boot%2F&sa=D&sntz=1&usg=AFQjCNFhhgfBs9pQoR8FzZGScIkaOEnbaw)[https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-test/src/test/java/org/springframework/boot/test](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fspring-projects%2Fspring-boot%2Ftree%2Fmaster%2Fspring-boot-project%2Fspring-boot-test%2Fsrc%2Ftest%2Fjava%2Forg%2Fspringframework%2Fboot%2Ftest&sa=D&sntz=1&usg=AFQjCNGnYBRzzH1m1kgefe-nxz5a3T4JHg) 
* 각자 알아서 스터디를 하시면서 가장 중요한건 위에서 스터디한 코드 와 내용을 참고로 본인의 test-example-bestpractice를 하나 만드셔서 실무에서 테스트 코드를 짤때 참고 하는 방향으로 하면 어떨까 합니다.

## Thumbnail 처리

### CMYK -> RGB 처리하기

* 내가알고 있었던 상식은 RGB는 빛의 삼원색이고, CMY는 색의 삼원색이라는 것이다.
* 근데, 이미지 중에 CMYK 형식으로 오는 이미지가 존재했다.
* CMYK는 CMY + black으로 인쇄물에 쓰이는 4원색을 뜻한다.
* 이 cmyk파일을 그대로 읽으면 색이 깨지거나 문제가 발생하는 경우가 있다.
* cmyk파일을 rgb로 변환 후 썸네일링을 해야 정상적으로 서빙할 수 있다.
  * https://stackoverflow.com/questions/3123574/how-to-convert-from-cmyk-to-rgb-in-java-correctly
  * http://huskdoll.tistory.com/347

## CI/CD

### 젠킨스 빌드 파이프라인

* http://blog.kingbbode.com/posts/jenkins-build
* https://jojoldu.tistory.com/355

## Intellij

* copyright

  * simple

    ```text
    Created by njkim $today.year.$today.month.$today.day.
    ```

    