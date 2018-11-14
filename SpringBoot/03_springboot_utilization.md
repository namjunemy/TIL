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

### 

