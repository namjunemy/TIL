# 25강. Security-1

## 25-1 보안 관련 프로젝트 생성

1. 프로젝트 생성
2. 한글 처리
3. 보안 관련 라이브러리 추가
4. 스프링 설정 파일 분리(web.xml)
5. 보안 관련 설정 파일 만들기

## 25-2 보안 관련 라이브러리 추가

 [https://projects.spring.io/spring-security/](https://projects.spring.io/spring-security/)

* 위의 페이지 방문하여 dependency를 추가하는 방법과, 이클립스 IDE에서 pom.xml에 add하는 방법이 있다.

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>5.0.0.RELEASE</version>
    </dependency>
</dependencies>
```

## 25-3 보안 관련 설정 파일 만들기

* src/main/webapp/WEB-INF/spring/appServlet 폴더에

* security-context.xml 파일 생성.

  * 파일 생성하는 방법은 new -> Spring Bean Configuration File 선택 -> next하면, xml파일에서 사용할 Bean Definition을 선택할 수 있다. 
  * 여기서 우리는 security dependency를 추가 했기 때문에, 선택 후 아래에서 최신 버전으로 선택한다.
  * 그러면 xml파일의 상단에 빈 설정에 사용할 수 있는 네임스페이스가 정의된다.

* src/main/webapp/WEB-INF/web.xml에 security-context.xml 설정파일을 추가한다.

  * security-context.xml안에 있는 설정 정보를 사용할 수 있다.

  ```xml
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
    <param-value>/WEB-INF/spring/security-context.xml</param-value>
  </context-param>
  ```

## 25-4 In-Memory 인증

