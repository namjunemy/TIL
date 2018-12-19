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