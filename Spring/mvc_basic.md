# 11. 스프링 MVC 기초

## 11-1. 스프링 MVC 개요

지금까지 스프링의 전반적이고 기본적인 내용에 대해서 살펴 보았다. 사실 스프링이 유명하게 된 계기는 아마도 웹 어플리케이션 제작에 적용되면서 웹 프레임워크로서 우수성이 인정되었기 때문일 것이다. 이제부터 웹 어플리케이션 제작을 위한 MVC에 대해서 하나씩 살펴보도록 한다.

* 먼저 클라이언트의 최초 요청을 받아서 여러 부분에 전달해주는 DispatcherServlet이 존재한다.
* 각 부분에 대한 상세 설명은 뒤에서 알아보고, 전체적인 흐름은 다음과 같다.
* 클라이언트의 요청이 오면 DispatcherServlet이 요청을 받고 Controller에게 요청을 하면 Controller에서 요청을 처리하고 다시 DispatcherServlet에 응답한다.
* 그러면 DispatcherServlet은 ViewResolver를 통하여 사용자에게 View(JSP)를 응답하게 된다.
* 뒤에서 더 자세히 살펴보도록 한다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/mvc_01.png?raw=true)

  

## 11-2. 스프링 MVC 구조 살펴보기

스프링 MVC의 전체적인 구조만 잘 정리를 해도, 반 이상을 학습한거라 생각해도 좋다. 그만큼 중요하다.

* New -> Project -> Spring Legacy Project -> Spring MVC Project 선택 해서 프로젝트 만들기

* 기본 패키지 구조는 2단계 이상 설정 권장(예를 들면, com.java.ex에서 ex가 context이름이 됨)

* 아래와 같은 구조의 프로젝트 생성 완료

  ![](https://github.com/namjunemy/TIL/blob/master/Spring/img/mvc_02.png?raw=true)

  * 프로젝트 구조에서는 보이지 않지만 DispatcherServler은 클라이언트의 요청을 최초로 받아서 컨드롤러에게 전달한다.

  * web.xml에서는 DispatcherServlet을 매핑하고, 스프링 컨테이너 설정 파일 servlet-context.xml의 위치를 정한다.

    - web.xml
      - 여기서 보면 / 경로로 요청이 들어오면 appServlet으로 매핑시키고 있고,
      - 이 appServlet은 org.springframework.web.servlet.DispatcherServlet로 명시적으로 매핑되어있다.
      - 그리고, contextConfigLocation이 /WEB-INF/spring/appServlet/servlet-context.xml이 경로에 설정되어 있기 때문에,
      - servlet-context.xml을 참조하여 해당 컨트롤러로 요청을 전달할 수 있다.

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

      <!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
      <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/root-context.xml</param-value>
      </context-param>

      <!-- Creates the Spring Container shared by all Servlets and Filters -->
      <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>

      <!-- Processes application requests -->
      <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
      </servlet>

      <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
      </servlet-mapping>

    </web-app>
    ```

  * src/main/java 밑의 HomeController.java는 DispatcherServlet에서 전달된 요청을 처리한다.

    * HomeController.java
      * HomeController에서 DispatcherServlet의 요청을 받을 수 있는 이유는 @Controller 어노테이션이 선언되어있기 때문이다.

    ```java
    @Controller
    public class HomeController {
      private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

      @RequestMapping(value = "/", method = RequestMethod.GET)
      public String home(Locale locale, Model model) {
        logger.info("Welcome home! The client locale is {}.", locale);

        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);

        String formattedDate = dateFormat.format(date);

        model.addAttribute("serverTime", formattedDate);

        return "home";
      }
    }
    ```

  * servlet-context.xml은 스프링 컨테이너의 설정파일이다.

    * 스프링 컨테이너 설정파일에서 conponent-scan설정에 패키지를 등록하면 비로소 해당 패키지 안의 @Controller 어노테이션이 선언된 파일이 컨트롤러의 역할을 할 수 있게 된다.
    * servlet-context.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans:beans xmlns="http://www.springframework.org/schema/mvc"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:beans="http://www.springframework.org/schema/beans"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
    		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

      <!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->

      <!-- Enables the Spring MVC @Controller programming model -->
      <annotation-driven />

      <!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the
        ${webappRoot}/resources directory -->
      <resources mapping="/resources/**" location="/resources/" />

      <!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views
        directory -->
      <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <beans:property name="prefix" value="/WEB-INF/views/" />
        <beans:property name="suffix" value=".jsp" />
      </beans:bean>

      <context:component-scan base-package="com.javalec.ex" />
      <context:component-scan base-package="추가 패키지 등록 가능" />

    </beans:beans>
    ```

  * 그리고 src/webapp/views에는 실제 뷰인 jsp파일들이 위치한다.

    * 실제 jsp파일이 호출되는 로직은 다음과 같다.
    * 컨트롤러에서 뷰의 이름이 return되는 것을 HomeController.java에서 볼 수 있다.
    * 뷰의 이름은 바로 위에 있는 servlet-context.xml에서 매핑된다.
    * 아래의 servlet-context.xml에서 ViewResolver 빈이 생성되고 prefix로 경로를 suffix로 jsp확장자를 등록함으로써 실제 뷰파일이 서비스 될 수 있다.

    ```xml
    ... servlet-context.xml 생략

      <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <beans:property name="prefix" value="/WEB-INF/views/" />
        <beans:property name="suffix" value=".jsp" />
      </beans:bean>
    ...
    ```

    ​

## 11.3 resources 폴더

뷰 페이지를 구성하기위한 static 파일들(img, css 등)은 servlet-context.xml인 스프링 컨테이너 설정 파일에 resources폴더를 등록해야 서비스 할 수 있다. 정적파일 서비스를 위한 요청과 폴더 경로를 정해놓지 않으면 DispatcherServlet이 모든 요청을 바로 Controller로 보내버리기 때문에 정적파일을 서비스 할 수 없게 된다.

- servlet-context.xml의 내용 중 resources 태그를 본다.

  - /resources/**로 mapping정보가 설정되어있고, location은 /resources/이다.
  - 따라서, 정적파일을 서비스 하기 위해 jsp에서 http://localhost:8080/ex/resources/mvc_01.png라는 경로로 이미지를 찾는 코드가 있다면, resources 매핑을 하지않은 상태에서는 서비스를 할 수 없다.
  - 폴더명은 사용자 정의가 가능하다 resources가 아니라 myResources에 정적파일이 있다면, 해당 location으로 등록을 해주면 된다.

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans:beans xmlns="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
  		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->

    <!-- Enables the Spring MVC @Controller programming model -->
    <annotation-driven />

    <!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the
      ${webappRoot}/resources directory -->
    <resources mapping="/resources/**" location="/resources/" />

    <!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views
      directory -->
    <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <beans:property name="prefix" value="/WEB-INF/views/" />
      <beans:property name="suffix" value=".jsp" />
    </beans:bean>

    <context:component-scan base-package="com.javalec.ex" />
    <context:component-scan base-package="추가 패키지 등록 가능" />

  </beans:beans>
  ```