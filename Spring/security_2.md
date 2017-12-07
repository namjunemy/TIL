# 26. Security-2

## 26-1 로그인 페이지 만들기

스프링이 제공하는 html페이지가 아닌 사용을 위한 로그인 페이지를 만든다.

* 직접 만드는 로그인 페이지에 연결하기 위해 security-context.xml에 로그인 페이지 설정을 한다.
* security-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:security="http://www.springframework.org/schema/security"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-4.2.xsd">


  <security:http auto-config="true">
    <security:form-login login-page="/loginForm.html"/>
    <security:intercept-url pattern="/login.html" access="ROLE_USER"/>
    <security:intercept-url pattern="/welcome.html" access="ROLE_ADMIN"/>
  </security:http>

  <security:authentication-manager>
    <security:authentication-provider>
      <security:user-service>
        <security:user name="user" password="123" authorities="ROLE_USER"/>
        <security:user name="admin" password="123" authorities="ROLE_ADMIN, ROLE_USER"/>
      </security:user-service>
    </security:authentication-provider>
  </security:authentication-manager>
</beans>

```

* 다음으로 로그인 페이지의 코드이다.
  * 여기서 c:url의 value로 j_spring_security_check를 입력해야 스프링 보안 설정 파일의 검증을 거칠 수 있다.
  * 마찬가지로 ID와 PW에 j_ 라는 약속된 키워드를 사용해야 스프링 보안 설정 파일을 참조할 수 있다.
  * loginForm.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>loginForm.jsp</h1>

<form action="<c:url value="j_spring_security_check" />" method="post">
	ID : <input type="text" name="j_username"> <br />
	PW : <input type="text" name="j_password"> <br />
	<input type="submit" value="LOGIN"> <br />
</form>

</body>
</html>
```



## 26-2 로그인, 로그아웃 상태 표시

[소스코드 repo](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_26_1_ex1_springex)

* 로그인에 실패 했을 경우 실패 메시지를 출력한다.
  * loginForm.jsp
    * jstl core 라이브러리를 사용하여 param의 ng값이 null이 아니면 Login NG와 메세지를 출력한다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>

  <c:url value="j_spring_security_check" var="loginUrl" />

  <form action="${loginUrl}" method="post">
    <c:if test="${param.ng != null}">
      <p>
        Login NG ! <br>
        <c:if test="${SPRING_SECURITY_LAST_EXCEPTION != NULL}">
          message : <c:out value="${SPRING_SECURITY_LAST_EXCEPTION.message }" />
        </c:if>
      </p>
    </c:if>
    ID : <input type="text" name="j_username"> <br />
    PW : <input type="text" name="j_password"> <br />
    <input type="submit" value="LOGIN"> <br />
  </form>

</body>
</html>
```

* 로그아웃 기능과 로그인 상태를 출력한다.
  * login.jsp
    * 마찬가지로 jstl core 라이브러리를 사용하여 pageContext의 request.userPrincipal이 비어있을 경우와 존재할 경우를 판단하여 로그인 상태를 출력한다.
    * 추가적으로 pageContext의 request.userPrincipal.name을 뽑아내서 로그인 아이디를 출력한다.
    * 로그아웃 기능을 추가하기 위해 ${pageContext.request.contextPath}/j_spring_security_logout로 이동시킨다.
      * contextPath를 가져오고, 약속된 키워드인 j_spring_security_logout를 통해 세션을 로그아웃 시킬 수 있다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
  <h1>login.jsp</h1>

  <c:if test="${not empty pageContext.request.userPrincipal }">
    <p>is Log-In</p>
  </c:if>

  <c:if test="${empty pageContext.request.userPrincipal }">
    <p>is Log-Out</p>
  </c:if>

  USER ID : ${pageContext.request.userPrincipal.name}
  <br />
  <a href="${pageContext.request.contextPath}/j_spring_security_logout">Log Out</a>
  <br />


</body>
</html>
```

