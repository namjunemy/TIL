# Tips

> 개발하면서 마주한 팁들

## Enum

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

## 일급컬렉션

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
  * jdbc:mysql://localhost:3306/customersdb?useSSL=false&amp;useUnicode=yes&characterEncoding=UTF-8
  * 내일 다시 정리 필요

## 테스트

### @Spy, @Mock, @SpyBean, @MockBean, @InjectMock

* https://jojoldu.tistory.com/226?category=635883
* https://jojoldu.tistory.com/239?category=635883

* http://blog.devenjoy.com/?p=529

* [https://velog.io/@june0313/Mockito-Mock-%EB%A6%AC%EC%8A%A4%ED%8A%B8%EB%A5%BC-%EC%A3%BC%EC%9E%85%ED%95%98%EA%B3%A0-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%98%EA%B8%B0](https://velog.io/@june0313/Mockito-Mock-리스트를-주입하고-테스트-하기)

## CI/CD

### 젠킨스 빌드 파이프라인

* http://blog.kingbbode.com/posts/jenkins-build
* https://jojoldu.tistory.com/355