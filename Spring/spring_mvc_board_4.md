# 19.스프링 MVC 게시판 4

## 19-1. 글 수정 페이지 만들기

* 같은 흐름으로 글 수정 페이지를 만든다.
  - 컨트롤러에 정의된 BModifyCommand클래스를 만든다.
  - BDao에 modify() 메소드를 만들면서 실제로 DB에서 처리할 작업을 정의한다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_6.png?raw=true)

### 서비스 객체 생성

* BModifyCommand.java
  * request로 넘어온 id, name, title, content를 가지고 Dao의 modify메소드를 실행시킨다.

```java
package com.javalec.spring_project_board_command;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.ui.Model;

import com.javalec.spring_project_board_dao.BDao;

public class BModifyCommand implements BCommand {

  @Override
  public void execute(Model model) {
    Map<String, Object> map = model.asMap();
    HttpServletRequest request = (HttpServletRequest) map.get("request");
    
    String bId = request.getParameter("bId");
    String bName = request.getParameter("bName");
    String bTitle = request.getParameter("bTitle");
    String bContent = request.getParameter("bContent");
    
    BDao dao = new BDao();
    dao.modify(bId, bName, bTitle, bContent);
  }
}
```

### DAO 메소드 추가

* BDao.java
  * 넘어온 인자들을 가지고 update query를 구성한다.
  * preparedStatement의 executeUpdate() 메소드를 통해 데이터를 수정한다.

```java
public class BDao {
  DataSource dataSource;

  public BDao() {
    try {
      Context context = new InitialContext();
      dataSource = (DataSource) context.lookup("java:comp/env/jdbc/springmvcdb");
    } catch (NamingException e) {
      e.printStackTrace();
    }
  }

  public void modify(String bId, String bName, String bTitle, String bContent) {
    Connection connection = null;
    PreparedStatement preparedStatement = null;

    try {
      connection = dataSource.getConnection();
      String query = "update mvc_board set bName = ?, bTitle = ?, bContent = ? where bId = ?";
      preparedStatement = connection.prepareStatement(query);
      preparedStatement.setString(1, bName);
      preparedStatement.setString(2, bTitle);
      preparedStatement.setString(3, bContent);
      preparedStatement.setInt(4, Integer.parseInt(bId));
      
      int rn = preparedStatement.executeUpdate();
      
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        if (preparedStatement != null)
          preparedStatement.close();
        if (connection != null)
          connection.close();
      } catch (Exception e2) {
        e2.printStackTrace();
      }
    }
  }
...
```

### Jsp 페이지

* contentView.jsp
  * 게시물 상세 조회 페이지에서 post메소드를 통해 데이터를 전송한다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Insert title here</title>
</head>
<body>

<table width="500" cellpadding="0" cellspacing="0" border="1">
  <form action="modify" method="post">
    <input type="hidden" name="bId" value="${contentView.bId}">
    <tr>
      <td> 번호</td>
      <td> ${contentView.bId} </td>
    </tr>
    <tr>
      <td> 히트</td>
      <td> ${contentView.bHit} </td>
    </tr>
    <tr>
      <td> 이름</td>
      <td><input type="text" name="bName" value="${contentView.bName}"></td>
    </tr>
    <tr>
      <td> 제목</td>
      <td><input type="text" name="bTitle" value="${contentView.bTitle}"></td>
    </tr>
    <tr>
      <td> 내용</td>
      <td><textarea rows="10" name="bContent">${contentView.bContent}</textarea></td>
    </tr>
    <tr>
      <td colspan="2"><input type="submit" value="수정"> &nbsp;&nbsp; <a href="list">목록보기</a> &nbsp;&nbsp;
        <a href="delete?bId=${contentView.bId}">삭제</a> &nbsp;&nbsp; <a
            href="replyView?bId=${contentView.bId}">답변</a></td>
    </tr>
  </form>
</table>

</body>
</html>
```



## 19-2. 글 삭제 페이지 만들기

같은 흐름으로 게시물 삭제 페이지를 만든다.

- 컨트롤러에 정의된 BDeleteCommand클래스를 만든다.
- BDao에 delete() 메소드를 만들면서 실제로 DB에서 처리할 작업을 정의한다

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_7.png?raw=true)



### 서비스 객체 생성

- BDeleteCommand.java
  - request로 넘어온 id를 가지고 Dao의 delete메소드를 실행시킨다.

```java
package com.javalec.spring_project_board_command;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.ui.Model;

import com.javalec.spring_project_board_dao.BDao;

public class BDeleteCommand implements BCommand {

  @Override
  public void execute(Model model) {
    Map<String, Object> map = model.asMap();
    HttpServletRequest request = (HttpServletRequest) map.get("request");
    
    String bId = request.getParameter("bId");
    
    BDao dao = new BDao();
    dao.delete(bId);
  }
}

```

### DAO 메소드 추가

- BDao.java
  - 넘어온 인자들을 가지고 delete query를 구성한다.
  - preparedStatement의 executeUpdate() 메소드를 통해 데이터를 삭제한다.

```java
public class BDao {
  DataSource dataSource;

  public BDao() {
    try {
      Context context = new InitialContext();
      dataSource = (DataSource) context.lookup("java:comp/env/jdbc/springmvcdb");
    } catch (NamingException e) {
      e.printStackTrace();
    }
  }

  public void delete(String bId) {
    Connection connection = null;
    PreparedStatement preparedStatement = null;

    try {
      connection = dataSource.getConnection();
      String query = "delete from mvc_board where bId = ?";
      preparedStatement = connection.prepareStatement(query);
      preparedStatement.setInt(1, Integer.parseInt(bId));

      int rn = preparedStatement.executeUpdate();

    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        if (preparedStatement != null)
          preparedStatement.close();
        if (connection != null)
          connection.close();
      } catch (Exception e2) {
        e2.printStackTrace();
      }
    }
  }
...
```

### Jsp 페이지

- contentView.jsp
  - 삭제 버튼을 통해서 delete 작업을 요청한다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Insert title here</title>
</head>
<body>

<table width="500" cellpadding="0" cellspacing="0" border="1">
  <form action="modify" method="post">
    <input type="hidden" name="bId" value="${contentView.bId}">
    <tr>
      <td> 번호</td>
      <td> ${contentView.bId} </td>
    </tr>
    <tr>
      <td> 히트</td>
      <td> ${contentView.bHit} </td>
    </tr>
    <tr>
      <td> 이름</td>
      <td><input type="text" name="bName" value="${contentView.bName}"></td>
    </tr>
    <tr>
      <td> 제목</td>
      <td><input type="text" name="bTitle" value="${contentView.bTitle}"></td>
    </tr>
    <tr>
      <td> 내용</td>
      <td><textarea rows="10" name="bContent">${contentView.bContent}</textarea></td>
    </tr>
    <tr>
      <td colspan="2"><input type="submit" value="수정"> &nbsp;&nbsp; <a href="list">목록보기</a> &nbsp;&nbsp;
        <a href="delete?bId=${contentView.bId}">삭제</a> &nbsp;&nbsp; <a
            href="replyView?bId=${contentView.bId}">답변</a></td>
    </tr>
  </form>
</table>

</body>
</html>
```