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
  ...
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
    <param-value>/WEB-INF/spring/security-context.xml</param-value>
  </context-param>
  ...
  ```

## 25-4 In-Memory 인증

[소스코드 repo](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_25_1_ex1_springex)

* Spring security를 인증 테스트 하기 위한 실습이다.
* 프로젝트 구성에는 보이지 않지만, 스프링 프레임워크에서 제공하는 login.html을 이용해서 인증테스트를 할 수 있다.
* security-context.xml 파일 작성
  *  name이 user이고 패스워드가 123인 유저는 ROLE_USER 권한을 갖고 login.html 페이지에 접근 할 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:security="http://www.springframework.org/schema/security"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-4.2.xsd">


  <security:http auto-config="true">
    <security:intercept-url pattern="/login.html" access="ROLE_USER" />
    <security:intercept-url pattern="/welcome.html" access="ROLE_ADMIN" />
  </security:http>

  <security:authentication-manager>
    <security:authentication-provider>
      <security:user-service>
        <security:user name="user" password="123" authorities="ROLE_USER" />
        <security:user name="admin" password="123" authorities="ROLE_ADMIN, ROLE_USER" />
      </security:user-service>
    </security:authentication-provider>
  </security:authentication-manager>
</beans>
```

* 보안 관련 설정인 security-context.xml에 있는 설정이 프로젝트에 적용되게 하기 위하여 web.xml파일에 Security필터 설정을 한다.
* web.xml
  * DelegatingFilterProxy를 이용하여 접근하는 모든 요청에 대해 필터를 거치게 한다.
  * 이 과정에서 spring-context.xml을 참조하여 권한이 있는 사용자가 페이지에 접근할 수 있도록 한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
    /WEB-INF/spring/root-context.xml
    /WEB-INF/spring/security-context.xml
    </param-value> 
  </context-param>

  <filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  
  ...
```



