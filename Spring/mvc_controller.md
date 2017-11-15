# 12. MVC Controller

최초 클라이언트로부터 요청이 들어왔을 때, 컨트롤러로 진입하게 된다. 그리고 컨트롤러는 요청에 대한 작업을 한 후 뷰쪽으로 데이터를 전달한다.

### 컨트롤러 클래스 제작 순서

* @Controller를 이용한 클래스 생성

  ```java
  @Controller
  public class MyController {
    ...
  ```

  * DispatcherServlet이 받은 요청을 처리할 패키지로 servlet-context.xml에 등록된 패키지 안에서 @Controller 어노테이션을 사용하여 컨트롤러를 정의한다.

* 요청 처리 메소드 구현

  ```java
  @RequestMapping("/content/contentView")
    public String contentView(Model model) {
      model.addAttribute("id", "namjunemy");
      
      return "/content/contentView";
  }
  ```

  * @RequestMapping을 이용한 요청 경로 지정
    * @RequestMapping("/board/view")
      * 요청 경로를 지정하는 어노테이션이다.
  * 뷰 이름 리턴
    * return "board/view"
      * 뷰 페이지 이름이다.
    * 뷰 페이지 이름 생성 방법
      * 뷰의 이름은 servlet-context.xml에서 매핑된다.
      * 아래의 servlet-context.xml에서 ViewResolver 빈이 생성되고 prefix로 경로를 suffix로 jsp확장자를 등록함으로써 실제 뷰파일이 서비스 될 수 있다.

  ```xml
  ... servlet-context.xml 생략

    <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <beans:property name="prefix" value="/WEB-INF/views/" />
      <beans:property name="suffix" value=".jsp" />
    </beans:bean>
  ...
  ```

* 뷰에 데이터 전달

  * 컨트롤러에서 로직 수행 후 뷰페이지를 반환한다. 이때, 뷰에서 사용하게 될 데이터를 객체로 전달 할 수 있다.

  * @RequestMapping이 정의 되어있는 메소드는 Model객체를 파라미터로 받을 수 있다.

  * 받은 model객체의 addAttribute를 이용하여 model 객체에 데이터를 담을 수 있다.

    ```java
    @RequestMapping("/content/contentView")
      public String contentView(Model model) {
        model.addAttribute("id", "namjunemy");
        
        return "/content/contentView";
      }
    ```


  * 후에 뷰에서는 전달받은 Model객체의 속성을 이용할 수 있다.

    ```jsp
    <%@ page language="java" contentType="text/html; charset=EUC-KR"
      pageEncoding="EUC-KR"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
    <title>Insert title here</title>
    </head>
    <body>
      contentView.jsp 입니다.
      <br /> id : ${id}
    </body>
    </html>
    ```

  * ModelAndVIew 클래스를 이용한 데이터 전달법도 있다.

    * ModelAndView 클래스는 Model과 View를 구성하는 클래스로 인스턴스를 생성한 뒤,
    * Object로 데이터를 등록하고
    * viewName을 설정함으로써 위의 Model객체를 인자로 받는 방법과 동일하게 사용할 수 있다.
    * 대신, return type도 ModelAndView 타입이다.

    ```java
    @RequestMapping("/board/reply")
      public ModelAndView reply() {

        ModelAndView mv = new ModelAndView();
        mv.addObject("id", 30);
        mv.setViewName("board/reply");

        return mv;
    }
    ```

* 클래스에 @RequestMapping 적용

  * 이제까지는 컨트롤러클래스에 정의된 메소드에 @RequestMapping을 적용하여 요청 경로를 얻었다.

  * @RequestMapping 어노테이션을 클래스에 적용하여 요청경로를 얻는 방법에 대해서 살펴본다.

  * 컨트롤러 클래스의 모든 메소드에 공통경로가 존재한다면, 클래스 공통으로 @RequestMapping을 적용 시킬 수 있다.

  * 이렇게 되면 컨트롤러 클래스의 @RequestMapping과 메소드의 @RequestMapping을 합쳐서 경로를 찾게 된다.

    * **아래의 contentView()메소드에 적용되는 매핑 경로는 /board/contentView이다.**

    ```java
    @Controller
    @RequestMapping("/board")
    public class MyController {
      @RequestMapping("/contentView")
      public String contentView() {
        ...
          
        return "/board/contentView"
      }
    }
    ```

    ​