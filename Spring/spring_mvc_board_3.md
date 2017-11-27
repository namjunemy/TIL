# 18. 스프링MVC 게시판 3

DB에 접근하여 데이터를 가져올 때, DBCP를 사용한다. JDBC는 뒤에서 학습한 후 변경 적용해 본다.

## 18-1. 리스트 조회 페이지 만들기

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_3.png?raw=true)

### Server의 context.xml 정의

* tomcat에 대한 설정이 담겨있는 Servers의 context.xml파일에 아래의 내용을 추가한다.
* MySQL DB에 접근하기 위한 정보를 입력한다.
* context.xml

```xml
<Resource name="jdbc/TestDB"
  auth="Container"
  type="javax.sql.DataSource"
  maxTotal="100"
  maxIdle="30"
  maxWaitMillis="10000"
  username="your_name"
  password="your_pw"
  driverClassName="com.mysql.jdbc.Driver"
  url="jdbc:mysql://localhost:3306/javatest" />
```

### 서비스 클래스 작성(커맨드 클래스)

* BListCommand.java

```java
package com.javalec.spring_project_board_command;

import java.util.ArrayList;

import org.springframework.ui.Model;

import com.javalec.spring_project_board_dao.BDao;
import com.javalec.spring_project_board_dto.BDto;

public class BListCommand implements BCommand {

  @Override
  public void execute(Model model) {
    BDao dao = new BDao();
    ArrayList<BDto> dtos = dao.list();

    model.addAttribute("list", dtos);
  }
}
```

### Dao 클래스 작성(list() 메소드)

* BDao.java
  * Server의 context.xml에 정의된 DB정보를 가지고 Connection을 만들기 위하여 Context, Datasource를 사용한다.
  * 이 후, Connection, PreparedStatement, ResultSet객체를 이용하여 DB에 query를 요청하여 게시물 리스트를 저장한다.
  * while문을 통하여 resultSet에 담겨있는 정보들을 ArrayList에 추가한다.
    * 이 때, 데이터를 담기 위해 미리 정의된 DTO객체를 이용한다.
  * 마지막으로 사용해준 자원을 해제 하고, 정보들이 담긴 ArrayList를 반환한다.

```java
package com.javalec.spring_project_board_dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Timestamp;
import java.util.ArrayList;

import javax.naming.Context;
import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.sql.DataSource;

import com.javalec.spring_project_board_dto.BDto;

public class BDao {
  DataSource dataSource;

  public BDao() {
    try {
      Context context = new InitialContext();
      dataSource = (DataSource) context.lookup("java:comp/env/jdbc/mysqlDB");
    } catch (NamingException e) {
      e.printStackTrace();
    }
  }

  public ArrayList<BDto> list() {
    ArrayList<BDto> dtos = new ArrayList<>();
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    ResultSet resultSet = null;

    try {
      connection = dataSource.getConnection();
      String query = "select bId, bName, bTitle, bContent, bDate, bHit, bGroup, bStep, bIndent"
          + " from mvc_board order by bGroup desc, bStep asc";
      preparedStatement = connection.prepareStatement(query);
      resultSet = preparedStatement.executeQuery();

      while (resultSet.next()) {
        int bId = resultSet.getInt("bId");
        String bName = resultSet.getString("bName");
        String bTitle = resultSet.getString("bTitle");
        String bContent = resultSet.getString("bContent");
        Timestamp bDate = resultSet.getTimestamp("bDate");
        int bHit = resultSet.getInt("bHit");
        int bGroup = resultSet.getInt("bGroup");
        int bStep = resultSet.getInt("bStep");
        int bIndent = resultSet.getInt("bIndent");

        BDto dto = new BDto(bId, bName, bTitle, bContent, bDate, bHit, bGroup, bStep, bIndent);
        dtos.add(dto);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        if (resultSet != null) resultSet.close();
        if (preparedStatement != null) preparedStatement.close();
        if (connection != null) connection.close();
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
    return dtos;
  }
}
```

### JSP 페이지 구성

* DAO가 수행하는 작업까지 끝나면, Controller에 의해 화면단으로 넘어오게 된다.
* list.jsp
  * model객체에 저장되어있는 list를 사용하여 화면에 출력한다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Insert title here</title>
</head>
<body>

<table width="500" cellpadding="0" cellspacing="0" border="1">
  <tr>
    <td>번호</td>
    <td>이름</td>
    <td>제목</td>
    <td>날짜</td>
    <td>히트</td>
  </tr>
  <c:forEach items="${list}" var="dto">
    <tr>
      <td>${dto.bId}</td>
      <td>${dto.bName}</td>
      <td><c:forEach begin="1" end="${dto.bIndent}">-</c:forEach> <a
          href="content_view?bId=${dto.bId}">${dto.bTitle}</a></td>
      <td>${dto.bDate}</td>
      <td>${dto.bHit}</td>
    </tr>
  </c:forEach>
  <tr>
    <td colspan="5"><a href="write_view">글작성</a></td>
  </tr>
</table>

</body>
</html>
```



## 18-2. 글 작성 페이지 만들기

* 게시물 리스트와 동일한 흐름으로 요청을 처리하는 작업이 이루어 진다.
* 대신, write_view 요청은 DAO작업이 필요하지 않기 때문에 직접 JSP페이지로 리턴한다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_4.png?raw=true)

### 게시물 작성

* write_view.jsp

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
  <form action="write" method="post">
    <tr>
      <td> 이름</td>
      <td><input type="text" name="bName" size="50"></td>
    </tr>
    <tr>
      <td> 제목</td>
      <td><input type="text" name="bTitle" size="50"></td>
    </tr>
    <tr>
      <td> 내용</td>
      <td><textarea name="bContent" rows="10"></textarea></td>
    </tr>
    <tr>
      <td colspan="2"><input type="submit" value="입력"> &nbsp;&nbsp; <a href="list">목록보기</a></td>
    </tr>
  </form>
</table>

</body>
</html>
```

### 게시물 등록

* 서비스객체에서 사용자가 입력한 내용을 DAO의 write() 메소드로 넘겨준다.
* write() 메소드에서는 DB에 접근하여 insert를 수행하고
* list페이지로 redirect한다.
* BWriteCommand.java

```java
package com.javalec.spring_project_board_command;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.ui.Model;

import com.javalec.spring_project_board_dao.BDao;

public class BWriteCommand implements BCommand {

  @Override
  public void execute(Model model) {
    Map<String, Object> map = model.asMap();
    HttpServletRequest request = (HttpServletRequest) map.get("request");

    String bName = request.getParameter("bName");
    String bTitle = request.getParameter("bTitle");
    String bContent = request.getParameter("bContent");

    BDao dao = new BDao();
    dao.write(bName, bTitle, bContent);
  }
}
```

* BDao.java의 write() 메소드
  * 요청받은 데이터를 DB에 insert 하는 작업을 수행한다.

```java
...

  public void write(String bName, String bTitle, String bContent) {
    Connection connection = null;
    PreparedStatement preparedStatement = null;

    try {
      connection = dataSource.getConnection();
      String query = "INSERT INTO mvc_board(bName, bTitle, bContent, bHit, bGroup, bStep, bIndent)"
          + "VALUE (?, ?, ?, 0, 0, 0, 0)";
      preparedStatement = connection.prepareStatement(query);
      preparedStatement.setString(1, bName);
      preparedStatement.setString(2, bTitle);
      preparedStatement.setString(3, bContent);
      
      int rn = preparedStatement.executeUpdate();

    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        if (preparedStatement != null)
          preparedStatement.close();
        if (connection != null)
          connection.close();
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
  }

...
```



## 18-3. 글 내용 페이지 만들기

* list 페이지에서 각 컨텐츠의 타이틀을 누르면 게시물의 id를 전달하여 해당 게시물을 보는 페이지를 요청한다.

  * list.jsp 코드의 일부분
    * \<a href="contenView?bId=${dto.bId}"\>${dto.bTitle}

  ```jsp
  <table width="500" cellpadding="0" cellspacing="0" border="1">
    <tr>
      <td>번호</td>
      <td>이름</td>
      <td>제목</td>
      <td>날짜</td>
      <td>히트</td>
    </tr>
    <c:forEach items="${list}" var="dto">
      <tr>
        <td>${dto.bId}</td>
        <td>${dto.bName}</td>
        <td>
          <c:forEach begin="1" end="${dto.bIndent}">-</c:forEach>
          <a href="contenView?bId=${dto.bId}">${dto.bTitle}</a></td>
        <td>${dto.bDate}</td>
        <td>${dto.bHit}</td>
      </tr>
    </c:forEach>
    <tr>
      <td colspan="5"><a href="writeView">글작성</a></td>
    </tr>
  </table>
  ```



![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_5.png?raw=true)



### contentView 서비스 객체

* BContentView.java
  * 요청받은 ID를 DAO객체의 메소드로 넘기는 역할을 한다.
  * 작업이 끝나면 DTO객체를 model객체에 추가한다.

```java
package com.javalec.spring_project_board_command;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.ui.Model;

import com.javalec.spring_project_board_dao.BDao;
import com.javalec.spring_project_board_dto.BDto;

public class BContentCommand implements BCommand {

  @Override
  public void execute(Model model) {
    Map<String, Object> map = model.asMap();
    HttpServletRequest request = (HttpServletRequest) map.get("request");
    String bId = request.getParameter("bId");

    BDao dao = new BDao();
    BDto dto = dao.contentView(bId);

    model.addAttribute("contentView", dto);
  }
}
```

  

### DAO contentView() 메소드

* BDao.java의 일부
  * DB에 게시물 ID를 바탕으로 데이터를 가져오기 위한 작업을 수행하고,
  * upHit() 메소드를 통해서 게시물 조회수를 올린다.

```java
...
  
public BDto contentView(String strId) {
    upHit(strId);

    BDto dto = null;
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    ResultSet resultSet = null;

    try {
      connection = dataSource.getConnection();

      String query = "select * from mvc_board where bId = ?";
      preparedStatement = connection.prepareStatement(query);
      preparedStatement.setInt(1, Integer.parseInt(strId));
      resultSet = preparedStatement.executeQuery();

      if (resultSet.next()) {
        int bId = resultSet.getInt("bId");
        String bName = resultSet.getString("bName");
        String bTitle = resultSet.getString("bTitle");
        String bContent = resultSet.getString("bContent");
        Timestamp bDate = resultSet.getTimestamp("bDate");
        int bHit = resultSet.getInt("bHit");
        int bGroup = resultSet.getInt("bGroup");
        int bStep = resultSet.getInt("bStep");
        int bIndent = resultSet.getInt("bIndent");

        dto = new BDto(bId, bName, bTitle, bContent, bDate, bHit, bGroup, bStep, bIndent);
      }

    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        if (resultSet != null)
          resultSet.close();
        if (preparedStatement != null)
          preparedStatement.close();
        if (connection != null)
          connection.close();
      } catch (Exception e2) {
        e2.printStackTrace();
      }
    }
    return dto;
  }

  private void upHit(String bId) {
    Connection connection = null;
    PreparedStatement preparedStatement = null;

    try {
      connection = dataSource.getConnection();
      String query = "update mvc_board set bHit = bHit + 1 where bId = ?";
      preparedStatement = connection.prepareStatement(query);
      preparedStatement.setString(1, bId);

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

  

### contentView JSP페이지

* contentView.jsp
  * 데이터를 바탕으로  jsp페이지를 구성한다.

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

