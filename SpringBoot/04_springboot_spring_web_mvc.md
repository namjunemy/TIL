# 스프링 웹 MVC

> [백기선 - 스프링 부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)

## 1. 소개

간단한 컨트롤러와 테스트 코드를 작성한다. @WebMvcTest 애노테이션을 사용하면 MockMvc를 주입받아서 사용할 수 있다.

아래의 테스트에서 우리는 아무런 설정파일을 작성하지 않았지만 스프링 MVC의 기능을 사용할 수 있었다. 이것이 가능한 것은 스프링 부트가 제공해주는 기본설정 때문이다.

자세히 말해서 spring-boot-stater의존성을 추가하면서 같이 따라온 spring-boot-autoconfigure의존성의 속을 까보면 spring.factories 파일 안에 WebMvcAutoConfiguration이라는 클래스가 존재하고, 이 클래스에 정의된 설정들 때문에 우리는 스프링 MVC의 기능을 바로 쓸 수 있다.

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

















