# 스프링 웹 MVC

> [백기선 - 스프링 부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)

## 1. 소개

간단한 컨트롤러와 테스트 코드를 작성한다. @WebMvcTest 애노테이션을 사용하면 MockMvc를 주입받아서 사용할 수 있다.

아래의 테스트에서 우리는 아무런 설정파일을 작성하지 않았지만 스프링 MVC의 기능을 사용할 수 있었다. 이것이 가능한 것은 스프링 부트가 제공해주는 기본설정 때문이다.

자세히 말해서 spring-boot-starter의존성을 추가하면서 같이 따라온 spring-boot-autoconfigure의존성의 속을 까보면 spring.factories 파일 안에 WebMvcAutoConfiguration이라는 클래스가 존재하고, 이 클래스에 정의된 설정들 때문에 우리는 스프링 MVC의 기능을 바로 쓸 수 있다.

```java
@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
            .andExpect(status().isOk())
            .andExpect(content().string("hello"));
    }
}
```

```java
@RestController
public class UserController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

* 스프링 부트가 제공해주는 기본설정을 사용하면서, 추가적으로 확장해서 쓰고 싶은경우
  * **@Configuration + WebMvcConfigurer** 로 추가 설정파일을 만들 수 있다.
* **주의할 점**은 **@Configuration + @EnableWebMvc** 애노테이션을 클래스에 붙이게 되면, 기본설정을 사용할 수 없고 Web MVC에 대한 모든 설정을 해당 클래스에서 정의해야 한다.

## 2. HttpMessageConverters

HttpMessageConverters는 스프링 프레임워크에서 제공하는 인터페이스이다.

HTTP 요청 본문을 객체로 변경하거나, 객체를 HTTP 응답 본문으로 변경할 때 사용한다. 사용하는 HttpmessageConverter는 여러가지가 있고, 우리가 어떤 요청을 받았는지, 응답을 보내는지에 따라서 메세지컨버터가 달라진다.

```java
{"username":"keesun", "password":"123"} <-> User
```

* @RequestBody

* @ResponseBody

  * 아래에서 User(객체)를 리턴할 때는 기본적으로 JsonMessageConverter가 사용이되고, String타입을 이턴할 때는 StringMessageConverter가 사용이 된다. int도 마찬가지로 StringMessageConverter이다.
  * **@RestController면 @ResponseBody는 생략해도 된다.** 
    * MessageConverter를 타고 객체를 응답 본문으로 바꾼다.
  * **그냥 @Controller를 사용할 경우에는 @ResponseBody를 넣어야 MessageConverter가 적용이된다.**
    * @Controller에서 @ResponseBody를 선언하지 않으면 BeanNameViewResolver에 의해서 ViewName에 해당하는 뷰를 찾으려고 시도한다.

  ```java
  @PostMapping("/user")
  public @ResponseBody User create(@RequestBody User user) {
    ...
    return new User( ... );
  }
  ```

* 테스트
  * MockMvc를 사용해서 post요청에 대한 응답을 확인하기 위한 테스트코드이다.
  * 요청을 @RestController에서 @RequestBody가 json을 파싱해서 User 객체로 만들고,  JsonMessageConverter에 의해 User가 JSON 형태로 만들어져서 return 된다.
  * 테스트코드에서는 jsonPath를 사용해서 결과로 받은 status와 JSON의 프로퍼티를 검증한다.
  * POST요청의 요청 타입은 JSON이고, 응답을 원하는 타입을 의미하는 accept 헤더도 JSON으로 세팅한다.

```java
package io.namjune.springbootconceptandutilization.user;

import static org.hamcrest.Matchers.equalTo;
import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void createUser_JSON() throws Exception {
        String userJson = "{\"username\":\"namjune\", \"password\":\"123\"}";
        mockMvc.perform(post("/users/create")
            .contentType(MediaType.APPLICATION_JSON_UTF8)
            .accept(MediaType.APPLICATION_JSON_UTF8)
            .content(userJson)
        )
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.username", is(equalTo("namjune"))))
            .andExpect(jsonPath("$.password", is(equalTo("123"))));
    }
}
```

```java
@RestController
public class UserController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @PostMapping("/users/create")
    public User create(@RequestBody User user) {
        return user;
    }
}
```

```java
@Getter
@Setter
public class User {

    private Long id;
    private String username;
    private String password;
}
```

## 3. ViewResolve

스프링부트에 등록 되어있는 스프링 웹 MVC의 [**ContentNegotiatingViewResolver**](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/web/servlet/view/ContentNegotiatingViewResolver.html) 가 어떤 contentType일 때 어떤 응답을 보내고, accept header 요청에 의해서 해당 요청에 맞는 응답을 보내는 작업을 알아서 해준다.

* https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-multiple-representations

그래서 Accept header를 XML 타입으로 설정하고 xpath를 이용해서 XML로 받는 응답을 검증하는 테스트코드를 작성하고 실행시켜보면 406 HttpMediaTypeNotAcceptableException이 떨어진다.

```java
package io.namjune.springbootconceptandutilization.user;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.xpath;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void createUser_JSON() throws Exception {
        String userJson = "{\"username\":\"namjune\", \"password\":\"123\"}";
        mockMvc.perform(post("/users/create")
            .contentType(MediaType.APPLICATION_JSON_UTF8)
            .accept(MediaType.APPLICATION_XML)
            .content(userJson)
        )
            .andExpect(status().isOk())
            .andExpect(xpath("/User/username").string("namjune"))
            .andExpect(xpath("/User/password").string("123"));
    }
}

```

스프링 부트의 HttpMessageConverters는 HttpMessageConvertersAutoConfiguration 클래스로 인해서 적용이 된다. 해당 클래스를 찾아보면 spring-boot-autoconfigure의존성 아래에 http/ 안에 존재한다. 이 클래스에 @Import 로 선언되어있는 **JacksonHttpMessageConvertersConfiguration** 클래스를 보면 MappingJackson2XmlHttpMessageConverterConfiguration 클래스가 정의 되어있고, MappingJackson2XmlHttpMessageConverterConfiguration  클래스에는 @ConditionalOnClass(XmlMapper.class)으로 XmlMapper클래스가 classpath에 존재해야만 등록되도록 정의되어 있다. 406 에러가 떨어진 이유는 xml로 변환할 수 있는 컨버터가 등록되어있지 않기 때문이다.

```java
@Configuration
class JacksonHttpMessageConvertersConfiguration {
  
  ...
    
  @Configuration
	@ConditionalOnClass(XmlMapper.class)
	@ConditionalOnBean(Jackson2ObjectMapperBuilder.class)
	protected static class MappingJackson2XmlHttpMessageConverterConfiguration {

		@Bean
		@ConditionalOnMissingBean
		public MappingJackson2XmlHttpMessageConverter mappingJackson2XmlHttpMessageConverter(
				Jackson2ObjectMapperBuilder builder) {
			return new MappingJackson2XmlHttpMessageConverter(
					builder.createXmlMapper(true).build());
		}

	}
 
  ...
    
}
```

XML 메시지 컨버터를 classpath에 추가해주면 문제를 해결할 수 있다. 의존성을 추가해주면 된다. 이제 위에서 작성한 XML 검증 테스트를 통과시킬 수 있다.

보통 JSON을 많이 사용하기 때문에 추가설정을 아무것도 하지 않아도 되지만, XML로 내보내고 싶은 경우 의존성을 추가해서 필요한 메세지 컨버터를 테스트 할 수 있다.

```groovy
dependencies {
    ...
      
    implementation('com.fasterxml.jackson.dataformat:jackson-dataformat-xml:2.9.6')
  
    ...
}
```

## 4. 정적 리소스 지원

정적 리소스 맵핑 "/**". 루트로 맵핑된다.

* 기본 리소스 위치

  * classpath:/static

  * classpath:/public

  * classpath:/resources/

  * classpath:/META-INF/resources

  * 예) "/hello.html" 접근시 /static/hello.html 응답

  * spring.mvc.static-path-pattern: 맵핑 설정 변경 가능

    * application.yml에서 `spring.mvc.static-path-pattern: /static/**` 으로 설정 변경시
    * localhost:8080/hello.html => localhost:8080/static/hello.html로 접근

  * spring.mvc.static-locations: 리소스 찾을 위치 변경 가능

    * 기존의 기본 리소스 위치를 사용하지 않고 변경한 위치만 사용하므로 권장하지 않는 방법이다.
    * 이방법 보다는 WebMvcConfigurer를 구현상속받아서 addResourceHandlers로 커스터마이징 하는 방법이 더 좋다. 기본 리소스 위치를 사용하면서 추가로 필요한 리소스위치만 정의해서 사용할 수 있다.
    * localhost:8080/m/hello.html 접근시 resources/m/hello.html 리턴
    * 여기서 주의할점은 캐시 설정을 따로 해야 한다는 것이다. **기본 리소스들은 기존의 캐싱 전략이 적용되어 있다.**

    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
    
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/m/**")
                .addResourceLocations("classpath:/m/")
                .setCachePeriod(20);
        }
    }
    ```

  * 기본 리소스 위치에 있는 리소스들은 **ResourceHttpRequestHandler**가 처리하는데, 브라우저에서 **304** status을 내려주는 경우가 있다.

    * html 파일이 변경되는 순간 파일에 Last-Modified 라는 최종 변경시간이 기록되는데,
    * 브라우저에서는 처음에 html파일을 요청하고 200 status를 받으면 해당 시간을 If-Modified-Since에 기록해 놓는다.
    * 그리고 브라우저에서 다시 html요청을 하고 응답을 받을때, resopnse 헤더에 넘어오는 Last-Modified 시간이 request 헤더에 보낸 If-Modified-Since와 같다면, html 파일의 변경이 없었다는 의미이므로 다시 리소스를 받아오지 않고 304 status와 캐시된 파일을 내려준다.
    * 하지만 html이 변경됐을 경우 response 헤더에 넘어오는 Last-Modified 시간이 request 헤더에 넘긴  If-Modified-Since 시간 이후이므로 새로 리소스를 받아서 200 status를 반환한다.

## 5. 웹 JAR

자바스크립트 라이브러리를 webjar형태로 dependency를 추가해서 사용할 수 있다.

스프링 부트에서 추가로 제공하는 기능이있는데, jquery의 버전이 올라갈 때마다 버전을 일일히 바꿔주지 않아도 된다. **이 기능을 사용하려면 webjars-locator-core 의존성을 추가해야 한다.**

이것의 내부적인 동작은 springframework의 resource chaining에 의해서 이루어진다. 필요하다면 더 자세히 공부하자.

```groovy
dependencies {
    ...
    compile group: 'org.webjars.bower', name: 'jquery', version: '3.3.1'
    compile group: 'org.webjars', name: 'webjars-locator-core', version: '0.36'
    ...
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
Hello static resource!

<script src="/webjars/jquery/dist/jquery.min.js"></script>
<script>
    $(function() {
        alert('ready!');
    });
</script>
</body>
</html>
```

## 6. Index 페이지와 파비콘

* 스프링 부트의 정적 리소스 4가지 기본 위치중 아무 곳이나 index.html을 두면된다.
  * classpath:/static
  * classpath:/public
  * classpath:/resources/
  * classpath:/META-INF/resources

* 그러면 스프링부트는
  * index.html을 찾아보고 있으면 제공
  * index.템플릿 찾아보고 있으면 제공
  * 둘 다 없으면 에러페이지를 내보낸다.
* favicon.io
  * 기본리소스 위치에 위의 파일명으로 위치시킨다.
  * 파이콘 만들기: <https://favicon.io/>
  * 파비콘은 캐시가 되어있으므로, 크롬에서 캐시비우고 새로고침을 하면 확인할 수 있다.