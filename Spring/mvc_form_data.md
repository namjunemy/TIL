# 13. Form 데이터

Spring에서 사용자가 어떻게 Form에 있는 데이터를 서버쪽으로 보내고, 어떻게 이 데이터를 서버에서 처리히는지 알아본다.

  

## 13-1. HttpServletRequest 클래스

HttpServletRequest클래스를 이용해서 데이터를 전송하는 방법에 대해서 살펴본다.

* @RequestMapping("board/confirmId")

  * 해당 url로 들어오는 데이터를 처리하기 위한 어노테이션
  * Model 객체는 데이터를 담아서 Jsp로 넘기기 위한 객체이다.
  * HttpServletRequest객체에 담겨있는 요청 데이터를 getParameter로 추출하여 저장한다.
  * model객체에 Attribute를 추가한다. 추가하면서 atttibute의 이름을 재정의 할 수 있다.
  * Return 하는 jsp 파일에서 model 객체에 들어있는 attribute를 사용할 수 있다.

  ```java
  @RequestMapping("board/confirmId")
  public String confirmId(HttpServletRequest httpServletRequest, Model model) {
    String id = httpServletRequest.getParameter("id");
    String pw = httpServletRequest.getParameter("id");
    
    model.addAttribute("id", id);
    model.addAttribute("pw", pw);
    
    return "board/confirmId";
  }
  ```

* board/confirmId.jsp

  ```html
  ...
  <body>
    ID : ${id} <br />
    PW : ${pw}
  </body>
  ```

#### 실습 코드

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_13_1_ex1_springex)

  

## 13-2. @RequestParam 어노테이션

@RequestParam 어노테이션을 이용해서 데이터를 전송하는 방법에 대해서 살펴 본다.

* @RequestParam을 사용 예시

  ```java
  @RequestMapping("board/checkId")
  public String checkId(@RequestParam("id") String id, @RequestParam("pw") String pw, Model model) {
    model.addAttribute("identify", id);
    model.addAttribute("password", pw);
    
    return "board/checkId";
  }
  ```

* 같은 방법으로 model 객체에 담아서 Jsp페이지로 넘겨준다.

* **중요한 것은 @RequestParam 어노테이션으로 선언을 하면, 선언된 Param들이 요청으로 들어오지 않을 경우 BadRequest 400 에러가 발생한다.**

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_13_1_ex1_springex)

  

## 13-3. 데이터(커맨드) 객체

데이터(커맨드)객체를 이용하여 데이터가 많을 경우 간단하게 사용할 수 있다. **실제로 많이 사용되는 방법이다.**

* 기존의 HttpServletRequest 객체에서 Parameter를 얻는 방법이나, @RequestParam 어노테이션을 이용하는 경우 처리해야할 데이터가 많아질수록 경우 코드의 양이 많아진다.

  ```java
  @RequestMapping("member/join")
  public String joinData(@RequestParam("name") String pw, @RequestParam("id") String id, @RequestParam("pw") String pw, @RequestParam("email") String email, Model model) {
    Member member = new Member();
    member.setName(name);
    member.setId(id);
    member.setPw(pw);
    member.setEmail(email);
    
    model.addAttribute("memberInfo", member);
    
    return "member/join";
  }
  ```

* 이러 경우 다음과 같이 데이터 객체를 사용하면 코드의 양이 적어진다.

  * 아래와 같이 Member객체만 받아서 넘기는 코드를 작성하면, **spring 프레임워크에서 알아서 Member객체의 필드에 setter를 이용하여 set해준다.** 그대로 jsp페이지에서 사용하면 된다.

  ```java
  @RequestMapping("member/join")
  public String joinData(Member member) {
    
    return "member/join";
  }
  ```

#### 실습코드

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_13_3_ex1_springex)

  

## 13-4. @PathVariable

@PathVariable 어노테이션을 이용하면 경로(Path)에 변수를 넣어 요청 메소드에서 파라미터로 이용할 수 있다.

* @PathVariable 사용 예시
  * RequestMapping 경로에 아래와 같이 변수를 선언한다

```java
@RequestMapping("/student/{studentId}")
public String getStudent(@PathVariable String studentId, Model model) {
  model.addAttribute("studentId", studentId);
  
  return "student/studentView";
}
```

* Jsp 페이지에서 사용

```jsp
<%@ page language="java" contentType="text/html; charset=EUC-KR"
	pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>Insert title here</title>
</head>
<body>student : ${studentId}

</body>
</html>
```

#### 실습코드

- 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_13_4_ex1_springex)