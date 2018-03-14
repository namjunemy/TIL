# Java Logging Framework 개념 및 기본설정

우리는 자바 애플리케이션을 개발하면서 습관적으로 `System.out.println(" ... ");`을 활용한다. 대부분은 작성한 프로그램의 정확성이나 논리적인 오류를 찾아내기 위한 디버깅 과정에서 자주 사용하고, 일부 경우에만 정보를 전달하거나 에러 메시지를 전달하기 위해 사용한다. 하지만 이것은 프로그램의 성능 이슈를 발생시킨다.

성능적으로 이슈가 발생하기 때문에 실 서비스하는 시점에서는 개발 단계에서 곳곳에 추가했던 `System.out.println(" ... ");`을 찾아다니면서 제거하는 번거로운 작업을 해야 한다.

여기서 이슈가 한 가지 더 발생한다. 실제 작성한 프로그램을 배포한 후, 추가로 문제가 발생한 상황에서는 다시 디버깅 메시지를 출력해서 해당 로직을 수정하는 작업 등을 해야 한다. 기존에 삭제했던 디버깅 메시지들을 추가해야 하는 상황이 발생할 수도 있다. 이러한 불편함이 있기 때문에 디버깅 메시지를 주석처리 해놓고 개발하기도 하지만, 그다지 매력적이지 않은 방법이다. 

이런 부분을 보완하기 위해 Logging Framework을 사용한다. 가장 유명한 프레임워크인 log4j, log4j의 단점을 보완한 SLF4J 그리고 가장 최근에 등장하여 log4j를 대체하고 있는 logback이라는 프레임워크가 존재한다.(세가지 프레임워크에 대해서는 줌인터넷 포털개발팀장이신 BeyondJ2EE님의 [포스팅("logback을 사용해야 하는 이유")](https://beyondj2ee.wordpress.com/2012/11/09/logback-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC-%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0-reasons-to-prefer-logback-over-log4j/)으로 대신한다.)

![](https://github.com/namjunemy/TIL/blob/master/Java/img/logging_framework_01.png?raw=true)

  

### 로그 레벨

```java
ALL < TRACE < DEBUG < INFO < WARN < ERROR < OFF
```

logback에서는 위와 같은 단계로 로그 레벨을 구분한다. **TRACE 레벨**은 TRACE, DEBUG, INFO, WARN, ERROR 레벨을 포함하고, **INFO 레벨**은 INFO, WARN, ERROR 레벨을 포함한다. 가장 많이 사용하게 될 DEBUG 레벨은 개발 단계에서 디버깅을 위한 메시지를 출력하고, 각 레벨 설정에 따라서 확인할 수 있는 로그 메시지의 종류가 달라진다. 아래에서 진행할 실습을 통해 더 자세히 살펴본다.



### 로그 메시지 형식

설정 파일의 configuration태그를 사용하여 로그 메시지의 형식을 설정할 수 있다. maven의 기본 resources 폴더의 경로는 src/main/resources 이다. 따라서, 해당 경로에 logback.xml 파일을 생성한다.

* logback.xml
  * 패턴이 포함하고 있는 내용은 순서대로 시간, 스레드, 로그 레벨, 로거를 포함하고 있는 클래스, 디버그 메시지 이다.

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```



### 예제

사용자가 한글을 입력하는 경우를 대비하여 Encoding설정을 하는 Filter 클래스에 로깅 프레임워크를 적용하는 예제이다.

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



### References

* [https://logback.qos.ch/manual/configuration.html](https://logback.qos.ch/manual/configuration.html)