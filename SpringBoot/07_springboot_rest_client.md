# 7. 스프링 REST 클라이언트

스프링 부트가 REST 클라이언트 관련해서 직접적으로 기능을 제공하는 것은 아니다. REST 클라이언트는 Spring Framework에서 제공하는 것이고, 부트는 그걸 쉽게 사용할 수 있도록 빈을 등록해준다.

주의 할 것은 RestTemplate과 WebClient 두 타입의 빈을 등록해주는 것이 아니라, 빌더를 등록해준다. 그래서 우리는 빌더를 주입받아서 필요할 때마다 REST 클라이언트를 빌드해서 사용해야한다.

## 7-1. RestTemplate과 WebClient

REST 클라이언트를 사용하는데에 두가지 선택사항이 있다. 차이점은 아래와 같다.

* **RestTemplate**

    * **Blocking I/O 기반의 Synchronous API**

        * Blocking I/O 기반의 Synchronous 개념을 이해하기 위한 예제 코드

            * https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/d2297945ffd022c912118af8aabca2738c894d0f
            * RestRunner에서 hi API를 호출한 뒤, Blocking 되므로 해당 메서드가 끝나기 전까지 다음 라인이 실행되지 않는다.
            * 따라서 두개의 API를 호출하고 시간을 찍어보면 약 8초가 나온다.

            ```java
            @RestController
            public class RestClientController {
            
                @GetMapping("/hi")
                public String hi() throws InterruptedException {
                    Thread.sleep(5000L);
                    return "hi";
                }
            
                @GetMapping("/bye")
                public String bye() throws InterruptedException {
                    Thread.sleep(3000L);
                    return "bye";
                }
            }
            ```

            ```java
            @Slf4j
            @Component
            public class RestRunner implements ApplicationRunner {
            
                @Autowired
                RestTemplateBuilder restTemplateBuilder;
            
                @Override
                public void run(ApplicationArguments args) throws Exception {
                    RestTemplate restTemplate = restTemplateBuilder.build();
            
                    StopWatch stopWatch = new StopWatch();
                    stopWatch.start();
            
                    String hiResult = restTemplate.getForObject("http://localhost:8080/hi", String.class);
                    log.info("/hi API Result : " + hiResult);
            
                    String byeResult = restTemplate.getForObject("http://localhost:8080/bye", String.class);
                    log.info("/bye API Result : " + byeResult);
            
                    stopWatch.stop();
                    log.info(stopWatch.prettyPrint());
                }
            }
            
            ```

            ```shell
            2019-06-30 17:17:52.271  INFO 37526 --- [           main] i.n.s.restclient.RestRunner              : /hi API Result : hi
            2019-06-30 17:17:55.280  INFO 37526 --- [           main] i.n.s.restclient.RestRunner              : /bye API Result : bye
            2019-06-30 17:17:55.281  INFO 37526 --- [           main] i.n.s.restclient.RestRunner              : StopWatch '': running time (millis) = 8425
            -----------------------------------------
            ms     %     Task name
            -----------------------------------------
            08425  100%  
            ```

    * RestTemplateAutoConfiguration.class 설정 클래스 내부

        * 프로젝트에 spring-web 모듈이 있다면 RestTemplate**Builder**를 빈으로 등록해준다.

        ```java
        @Configuration
        @AutoConfigureAfter(HttpMessageConvertersAutoConfiguration.class)
        @ConditionalOnClass(RestTemplate.class)
        public class RestTemplateAutoConfiguration {
        
            private final ObjectProvider<HttpMessageConverters> messageConverters;
        
            private final ObjectProvider<RestTemplateCustomizer> restTemplateCustomizers;
        
            public RestTemplateAutoConfiguration(
                ObjectProvider<HttpMessageConverters> messageConverters,
                ObjectProvider<RestTemplateCustomizer> restTemplateCustomizers) {
                this.messageConverters = messageConverters;
                this.restTemplateCustomizers = restTemplateCustomizers;
            }
        
            @Bean
            @ConditionalOnMissingBean
            public RestTemplateBuilder restTemplateBuilder() {
                RestTemplateBuilder builder = new RestTemplateBuilder();
                HttpMessageConverters converters = this.messageConverters.getIfUnique();
                if (converters != null) {
                    builder = builder.messageConverters(converters.getConverters());
                }
        
                List<RestTemplateCustomizer> customizers = this.restTemplateCustomizers
                    .orderedStream().collect(Collectors.toList());
                if (!CollectionUtils.isEmpty(customizers)) {
                    builder = builder.customizers(customizers);
                }
                return builder;
            }
        }
        ```

    * https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#rest-client-access

* **WebClient**

    * WebClientAutoConfiguration

    * 프로젝트에 spring-webflux 모듈이 있다면 WebClient.**Builder**를 빈으로 등록해준다.

        - 가장 쉬운 방법은 spring-boot-starter-webflux를 의존성에 추가하는 방법이다.

    * **Non-Blocking I/O 기반의 Asynchronous API**

        * 개념을 이해하기 위한 예제 코드(컨트롤러는 위와 동일)

            * https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/08fcec6bfcaaa0cb812d3d45e6a36297e6a6a136
            * WebClient로 부터 Mono를 만드는 라인은 Blocking되지 않는다. 실제로 subscribe가 일어날 때 요청해서 String객체를 파싱하는 동작을 한다. 이 동작도 Non-Blocking으로 동작한다. 단지, Asynchronous하게 응답이 오면 subscribe의 콜백함수가 동작을 하게 된다. 호출만 되고 밑라인을 수행한다.
            * 그렇기 때문에, 늦게 호출한 /bye 에 대한 동작이 3초밖에 소요되지 않으므로 먼저 끝나서 로그로 찍히고, 뒤에 5초가 소요되는 /hi 의 로그가 찍힌다.
            * 총 소요시간은 5초밖에 되지 않는다.

            ```java
            @Slf4j
            @Component
            public class RestRunner implements ApplicationRunner {
            
                @Autowired
                WebClient.Builder builder;
            
                @Override
                public void run(ApplicationArguments args) throws Exception {
                    WebClient webClient = builder.build();
            
                    StopWatch stopWatch = new StopWatch();
                    stopWatch.start();
            
                    // 이 라인이 실행되도 아무런 동작을 하지 않는다.
                    Mono<String> hiMono = webClient.get().uri("http://localhost:8080/hi")
                        .retrieve()
                        .bodyToMono(String.class);
            
                    // subscribe를 해줘야 스트리밍이 일어난다.
                    hiMono.subscribe(hiResult -> {
                        log.info("/hi API Result : " + hiResult);
            
                        // Asynchronus 이므로 어느것이 먼저 끝날지 모른다. 스탑워치가 돌고있으면 멈추고,
                        if (stopWatch.isRunning()) {
                            stopWatch.stop();
                        }
            
                        // 로그를 찍고, 스탑워치를 다시 돌려줘야 한다. Asynchronus 이므로 순서보장이 안되기 때문에.
                        log.info(stopWatch.prettyPrint());
                        stopWatch.start();
                    });
            
                    Mono<String> byeMono = webClient.get().uri("http://localhost:8080/bye")
                        .retrieve()
                        .bodyToMono(String.class);
            
                    byeMono.subscribe(byeResult -> {
                        log.info("/bye API Result : " + byeResult);
            
                        if (stopWatch.isRunning()) {
                            stopWatch.stop();
                        }
            
                        log.info(stopWatch.prettyPrint());
                        stopWatch.start();
                    });
                }
            }
            ```

            ```shell
            2019-06-30 17:39:27.124  INFO 39877 --- [ctor-http-nio-2] i.n.s.restclient.RestRunner              : /bye API Result : bye
            2019-06-30 17:39:27.125  INFO 39877 --- [ctor-http-nio-2] i.n.s.restclient.RestRunner              : StopWatch '': running time (millis) = 3375
            -----------------------------------------
            ms     %     Task name
            -----------------------------------------
            03375  100%  
            
            2019-06-30 17:39:29.064  INFO 39877 --- [ctor-http-nio-4] i.n.s.restclient.RestRunner              : /hi API Result : hi
            2019-06-30 17:39:29.065  INFO 39877 --- [ctor-http-nio-4] i.n.s.restclient.RestRunner              : StopWatch '': running time (millis) = 5314
            -----------------------------------------
            ms     %     Task name
            -----------------------------------------
            03375  064%  
            01939  036%  
            ```

    * https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client

## 7-2. REST 클라이언트 커스터마이징

* **RestTemplate**

  * 기본으로 java.net.HttpURLConnection 사용

  * 커스터마이징

    * 로컬 커스터마이징

      * 로컬에서 정의 후 안에서만 사용

    * 글로벌 커스터마이징

      * RestTemplate이 기본적으로 사용하는 java.net.HttpURLConnection를 HttpClient로 사용해야 하는 상황이 있을 수 있다.

      * 먼저 HttpClient를 빈으로 만들기 위해서 의존성에 추가한다.

      * 그리고 RestTemplateCustomizer를 빈으로 등록해서 전역 사용한다.

      * 커스터마이저 안에서 restTemplate.setRequestFactory()로 맞는 구현체를 등록한다.(이것 역시 Spring의 PSA(Portable Service Abstraction)가 잘 적용 되어있다.)

      * 이렇게 되면 더이상 기본 커넥션이 아닌, apache에서 제공하는 org.apache.http.client.HttpClient를 RestTemplate이 사용하게 된다.

        ```java
        @Bean
        public RestTemplateCustomizer restTemplateCustomizer() {
          return new RestTemplateCustomizer() {
            @Override
            public void customize(RestTemplate restTemplate) {
              restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
            }
          };
        }
        ```

      * 빈 재정의

* **WebClient**

  * 기본으로 Reactor Netty의 HTTP 클라이언트 사용

  * 커스터마이징

    * 로컬 커스터마이징

      ```java
      WebClient webClient = builder
      	.baseUrl("http://localhost:8080")
        .build();
      
      ...
        
      Mono<String> hiMono = webClient.get().uri("/hi")
        .retrieve()
        .bodyToMono(String.class);
      
      ...
      
      Mono<String> byeMono = webClient.get().uri("/bye")
        .retrieve()
        .bodyToMono(String.class);
              
      ```

    * 글로벌 커스터마이징

      * WebClientCustomizer를 빈으로 등록해서 전역 사용

        * 빈으로 등록해 놓고, 사용하는 쪽에서는 build()만 해서 사용

          ```java
          @Bean
          public WebClientCustomizer webClientCustomizer() {
            return webClientBuilder -> webClientBuilder.baseUrl("http://localhost:8080");
          }
          ```

      * 또는 빈 재정의 해서 사용해도 됨.

## Reference

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html
- [백기선 - 스프링 부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)