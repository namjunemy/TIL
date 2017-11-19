# 14. @RequestMapping 파라미터

## 14-1. @RequestMapping에서 Get방식과 Post방식

@RequestMapping에서 요청을 받을 때 Get방식과 Post방식을 구분 할 수 있다.

* HTML Form 태그를 통해 아래와 같이 get 방식으로 데이터를 전송한다.

  ```jsp
  ...
  <form action="student" method="get">
    student id : <input type="text" name="id"> <br />
    <input type="submit" value="전송">
  </form>
  ...
  ```

* Controller에서는 @RequestMapping을 통해 method를 정의하고, path(value)를 설정한다. 

  * 그리고 해당 path와 get Method로 들어온 데이터를 HttpServletRequest 객체를 통해

  ```java
  @RequestMapping(method = RequestMethod.GET, value = "/student")
  public String goStudent(HttpServletRequest httpServletRequest, Model model) {
    System.out.println("RequestMethod.GET");
    
    String id = httpServletRequest.getParameter("id");
    System.out.println("id : " + id);
    model.addAttribute("studentId", id);
    
    return "student/studentId";
  }
  ```

* 만약 HTML Form 태그의 method가 post라면 get Method를 처리하는 Controller에서는 처리할 수 없다. 405 에러가 발생한다.

  * Form 태그의 메소드가 post라면 controller의 @RequestMapping의 처리 메소드 또한 post로 변경하여 데이터를 처리할 수 있도록 한다.

  ```HTML
  ...
  <form action="student" method="post">
    student id : <input type="text" name="id"> <br />
    <input type="submit" value="전송">
  </form>
  ...
  ```

  #### 실습코드

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_14_1_ex1_springex)

  

## 14-2. @ModelAttribute

@ModelAttribute 어노테이션을 이용하면 커맨드 객체(데이터 객체)의 이름을 개발자가 변경 할 수 있다.

* 아래와 같이 데이터 객체를 사용할 때, 그 클래스의 이름 그대로 Jsp페이지에서 사용하기 불편한 경우가 있다.

  * StudentInformation라는 클래스가 데이터 객체로 사용될 경우 JSP페이지에서 저 클래스 이름 그대로 사용해야 한다.

  ```java
  @RequestMapping("/studentView")
  public String studentView(StudentInformation studentInformation) {
    return "studentView";
  }
  ```

* 따라서, 아래와 같이 @ModelAttribute 어노테이션을 설정하면 설정한 이름으로 jsp페이지에서 사용 가능하다.

  ```java
  @RequestMapping("/studentView")
  public String studentView(@ModelAttribute("studentInfo") StudentInformation studentInformation) {
    return "studentView";
  }
  ```

* @ModelAttribute 어노테이션 설정 값을 사용하는 jsp페이지

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8"
  	pageEncoding="EUC-KR"%>
  <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
  <html>
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
  <title>Insert title here</title>
  </head>
  <body>
  	이름 : ${studentInfo.name}
  	<br /> 나이 : ${studentInfo.age}
  	<br /> 학년 : ${studentInfo.classNum}
  	<br /> 반 : ${studentInfo.gradeNum}
  </body>
  </html>
  ```

#### 실습코드

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_14_2_ex1_springex)

  

## 14-3. 리다이렉트(redirect) 키워드

다른 페이지로 이동할 때 사용한다.

* Controller 코드에서 필요에 따라 다른 페이지로 이동이 필요할 때 사용한다.

  * 아래의 키워드로 redirect 설정을 하면, 같은 컨트롤러에 해당 요청을 처리하는 컨트롤러가 페이지 이동을 담당하게 된다.
  * 예를 들면, 로그인 실패와 로그인 성공을 처리하는데 사용된다.

  ```java
  ...
  @RequestMapping("/studentConfirms")
  public String studentRedirect(HttpServletRequest httpServletRequest, Model model) {
    String id = httpServletRequest.getParameter("id");
    if(id.equals("abc")) {
      return "redirect:studentOk";
    }
    
    return "redirect:stidentNg"
  }
  ...
  ```

* studentOk경로의 리다이렉트 요청을 처리하는 컨트롤러

  ``` java
  ...
  @RequestMapping("/studentOk")
  public String studentOk(Model model) {
    return "student/studentOk";
  }
  ...
  ```

* studentNg경로의 리다이렉트 요청을 처리하는 컨트롤러

  ```java
  ...
  @RequestMapping("/studentNg")
  public String studentOk(Model model) {
    return "student/studentNg";
  }
  ...
  ```

#### 실습코드

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_14_3_ex1_springex)