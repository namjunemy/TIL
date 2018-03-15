# Java Logging Framework, LOGBack

우리는 자바 애플리케이션을 개발하면서 습관적으로 `System.out.println(" ... ");`을 활용한다. 대부분은 작성한 프로그램의 정확성이나 논리적인 오류를 찾아내기 위한 디버깅 과정에서 자주 사용하고, 일부 경우에만 정보를 전달하거나 에러 메시지를 전달하기 위해 사용한다. 

하지만 이것은 프로그램의 성능 이슈를 발생시키기 때문에 실 서비스하는 시점에서는 개발 단계에서 곳곳에 추가했던 `System.out.println(" ... ");`을 찾아다니면서 제거하는 번거로운 작업을 해야 한다.

여기서 이슈가 한 가지 더 발생한다. 실제 작성한 프로그램을 배포한 후, 추가로 문제가 발생한 상황에서는 다시 디버깅 메시지를 출력해서 해당 로직을 수정하는 작업 등을 해야 한다. 기존에 삭제했던 디버깅 메시지들을 추가해야 하는 상황이 발생할 수도 있다. 이러한 불편함이 있기 때문에 디버깅 메시지를 주석처리 해놓고 개발하기도 하지만, 그다지 매력적이지 않은 방법이다. 

이런 부분을 보완하기 위해 Logging Framework를 사용한다. 가장 유명한 프레임워크인 log4j, log4j의 단점을 보완한 logging facade SLF4J 그리고 가장 최근에 등장하여 log4j를 대체하고 있는 LOGBack이라는 프레임워크가 존재한다.(세 가지 프레임워크에 대해서는 줌인터넷 포털개발팀장이신 BeyondJ2EE님의 [포스팅("logback을 사용해야 하는 이유")](https://beyondj2ee.wordpress.com/2012/11/09/logback-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC-%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0-reasons-to-prefer-logback-over-log4j/)으로 대신한다.)

![](https://github.com/namjunemy/TIL/blob/master/Java/img/logging_framework_01.png?raw=true)

  

### 로그 레벨

```java
ALL < TRACE < DEBUG < INFO < WARN < ERROR < OFF
```

logback에서는 위와 같은 단계로 로그 레벨을 구분한다. **TRACE 레벨**은 TRACE, DEBUG, INFO, WARN, ERROR 레벨을 포함하고, **INFO 레벨**은 INFO, WARN, ERROR 레벨을 포함한다. 가장 많이 사용하게 될 DEBUG 레벨은 개발 단계에서 디버깅을 위한 메시지를 출력하고, 각 레벨 설정에 따라서 확인할 수 있는 로그 메시지의 종류가 달라진다. 다음 단계에서 더 자세히 살펴본다.



### 로그 메시지 형식

설정 파일의 configuration 태그를 사용하여 로그 메시지의 형식을 설정할 수 있다. maven의 기본 resources 폴더의 경로는 src/main/resources 이다. 따라서, 해당 경로에 logback.xml 파일을 생성한다.

* logback.xml
  * 패턴이 포함하고 있는 내용은 순서대로 시간, 스레드, 로그 레벨, 로거를 포함하고 있는 클래스, 디버그 메시지 이다.

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```



### 예제

사용자가 한글을 입력하는 경우를 대비하여 Encoding 설정을 하는 Filter 클래스에 로깅 프레임워크를 적용하는 예제이다.

* LoggerFactory 생성
  * logger를 사용하기 위해서 LoggerFactory를 생성한다.
  * org.slf4j.Logger와 LoggerFactory를 import한다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

...
@WebFilter(urlPatterns = { "/*" })
public class CharacterEncodingFilter implements Filter {
	static final Logger logger = LoggerFactory.getLogger(CharacterEncodingFilter.class);
    ...
```

* CharacterEncodingFilter.java
  * System.out.println()를 대신하여 logger를 통해서 로그를 출력한다.
  * debug() 메소드는 로그 레벨을 DEBUG로 설정한다.

```java
package io.namjune.support;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@WebFilter(urlPatterns = { "/*" })
public class CharacterEncodingFilter implements Filter {
  static final Logger logger = LoggerFactory.getLogger(CharacterEncodingFilter.class);

  @Override
  public void init(FilterConfig filterConfig) throws ServletException {
    logger.debug("character encoding filter init!");
  }

  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    request.setCharacterEncoding("UTF-8");
    chain.doFilter(request, response);
  }

  @Override
  public void destroy() {
  }
}

```

* 로그 출력 결과
  * logback.xml 파일에 설정한 configuration 형식대로 로그가 출력된다.
  * 해당 메시지는 DEBUG 로그 레벨 메시지이다.

```shell
...
23:30:06.266 [localhost-startStop-1] DEBUG i.n.support.CharacterEncodingFilter - character encoding filter init!
...
```

  

### 로그 레벨 추가 설정

개발 단계에서 디버깅을 위해 로그 레벨을 DEBUG로 설정하고 개발을 한다. 그러나 실제 프로그램을 운영하는 단계에서는 DEBUG 레벨의 메시지들이 모두 로그로 찍히면 프로그램의 성능 이슈가 생긴다. 특히, 로그를 파일로 저장한다면 파일 I/O에 대한 비용이 발생하면서 성능에 직접적인 영향을 준다. 이것은 디버그 메시지와 사용자가 많을수록 더 커진다.

따라서, logback 설정파일을 이용하여 로그 레벨을 변경할 수 있다. 위에서 기본으로 설정한 logback.xml 파일을 보면, root 태그에 설정한 로그 레벨이 DEBUG로 되어 있다. 애플리케이션의 default 로그 레벨을 DEBUG가 아닌 INFO 또는 WARN으로 설정하면, 해당 레벨의 메시지만 로그로 남길 수 있다.

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="INFO or WARN">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

실제로 logback 설정파일에서 로그 레벨을 INFO로 설정하면 `logger.debug("debug message");`를 통해 출력하는 DEBUG 레벨의 메시지는 출력하지 않는다. 따라서, 개발단계에서는 DEBUG 레벨로 개발하고, 실제로 서비스를 운영할 때에는 default 로그 레벨을 INFO 레벨이나 WARN 레벨로 변경하여 성능 이슈가 발생하지 않도록 한다.

  

### 패키지 별 Logging Framework 설정

실제로 로깅 프레임워크를 사용하다 보면, 의존하고 있는 라이브러리에 의해서 찍히는 로그들을 상당히 많이 볼 수 있다. Java Bean Validation Check를 하기 위해 Hibernate Validator를 사용하는 경우 해당 라이브러리에서 디버깅을 위해 출력하는 메시지가 자연스레 찍히는 경우가 이에 해당한다. 이렇게 개발자가 원하지 않은 로그 메시지를 출력하지 않기 위해서 패키지별로 Logger를 관리할 수 있다.

아래와 같이 configuration에 logger를 등록하고 관리한다. 다음의 logback 설정은 io.namjune 패키지에 대해서는 DEBUG 로그 레벨의 메시지를 출력하고, 애플리케이션 default 로그 레벨은 WARN으로 설정한다. 이렇게 설정을 할 경우, 개발자가 의도한 패키지의 DEBUG 레벨 로그 메시지만 출력하고, Hibernate Validator 등 의존 라이브러리에서 찍히는 DEBUG 로그 레벨의 메시지는 출력되지 않는다.

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="io.namjune" level="DEBUG" />
  
  <root level="WARN">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

  

### 로깅 메시지 구현 단계의 성능 이슈 해결하기

아래와 같이 해당 패키지의 로그 레벨을 INFO 레벨로 설정했다고 가정하자. io.namjune 패키지에 대해서는 INFO 레벨 이상의 로그 메시지만 출력될 것이다. 그리고 설정된 INFO 레벨 아래의 DEBUG, TRACE 단계의 로그 메시지를 출력하는 로직이 수행될 경우 개발자가 의도하지 않은 성능상 이슈가 발생한다.

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="io.namjune" level="INFO" />
  
  <root level="WARN">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

아래와 같이 작성된 Servlet 코드를 보자. Java 코드가 동작하는 과정을 생각해보면, logger.debug() 메소드를 실행하기 위해서 String과 String을 더하는 연산 즉, 비용이 발생한다. INFO 레벨의 로그 메시지만 찍히길 바라는 개발자로서는 필요하지 않은 성능 이슈이다.

```java
@WebServlet("/users/create")
public class CreateUserServlet extends HttpServlet {
  static final Logger logger = LoggerFactory.getLogger(CreateUserServlet.class);

  @Override
  protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String userId = request.getParameter("userId");
    String password = request.getParameter("password");
    String name = request.getParameter("name");
    String email = request.getParameter("email");

    User user = new User(userId, password, name, email);

    logger.debug("User : " + user);
    
    ...
  }
}
```

logback에서는 다음과 같이 구현할 수 있다. 아래와 같이 구현하면, 먼저 debug() 메소드가 현재 로그 레벨을 판단하고 로깅 메시지의 출력을 결정한다. 로깅 메시지의 출력 여부가 결정된 다음에, 전달되는 두 개의 인자의 formatting 연산을 통해 메시지를 구성한다. 결과적으로 불필요한 연산의 비용이 발생하지 않는다.

```java
@WebServlet("/users/create")
public class CreateUserServlet extends HttpServlet {
  static final Logger logger = LoggerFactory.getLogger(CreateUserServlet.class);

  @Override
  protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String userId = request.getParameter("userId");
    String password = request.getParameter("password");
    String name = request.getParameter("name");
    String email = request.getParameter("email");

    User user = new User(userId, password, name, email);

    logger.debug("User : {}", user);
    
    ...
  }
}
```

개발을 하면서 위와 같은 방법으로 디버깅을 위한 로그 메시지를 출력한다면 분명히 성능상의 이점을 얻을 수 있다. 이런 습관들이 작은 프로젝트에서는 큰 영향이 없을 수도 있겠지만, 나중에 상상할 수 없을 정도로 큰 규모의 프로젝트에서는 이야기가 달라진다.

> 손에 익은 sout + tab 보다는 Logging Framwork을 사용하는 습관을 기르자.

  

### References

* [https://logback.qos.ch/manual/configuration.html](https://logback.qos.ch/manual/configuration.html)

### Contributor

**Namjun Kim**\<namjunemy@gmail.com>

* [github](https://github.com/namjunemy)
* [blog](http://ict-nroo.tistory.com/)