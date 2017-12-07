# 27. Security-3

## 27-1. 보안 관련 taglibs 사용 방법

pom.xml에 dependency로 spring-security-taglibs를 추가하여 사용할 수 있다.

pageContext.request.userPrincipal.name을 통해 얻어오던 사용자의 name을 설정된 taglibs의 prefix의 authentication의 property를 통해 가져올 수 있다. 

마찬가지로 로그인 상태를 표시하던 ${not empty pageContext.request.userPrincipal } 표현도, \<s:authorize ifAnyGranted="ROLE_USER"> 로 대체하여 표현할 수 있다.

* login.jsp

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

* 변경된 login.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://www.springframework.org/security/tags" prefix="s" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Insert title here</title>
</head>
<body>
<h1>login.jsp</h1>

<s:authorize ifAnyGranted="ROLE_USER">
  <p>is Log-In</p>
</s:authorize>

<s:authorize ifNotGranted="ROLE_USER">
  <p>is Log-out</p>
</s:authorize>

USER ID : <s:authentication property="name"/>
<br/>
<a href="${pageContext.request.contextPath}/j_spring_security_logout">Log Out</a>
<br/>

</body>
</html>
```