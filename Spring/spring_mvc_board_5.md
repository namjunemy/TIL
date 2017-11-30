# 20. 스프링MVC 게시판 5

현재는 스프링의 전체적인 구조들을 만들고 있다, 추가로 진행되는 학습에서 구조에 살을 붙여 나가는 학습을 한다.

## 20-1. 글 답변 페이지 만들기

해당 게시글에 답변을 다는 기능을 추가 한다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_8.png?raw=true)

* 컨트롤러에 정의된 BReplyCommand, BReplyViewCommand클래스를 만든다.
* BDao에 reply(), replyView() 메소드를 만들면서 실제로 DB에서 처리할 작업을 정의한다.

### 서비스 객체 생성

* BReplyViewCommand.java
  * 게시물의 id를 바탕으로 답변을 작성할 수 있는 화면을 제공하기 위한 클래스

```java
package com.javalec.spring_project_board_command;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.ui.Model;

import com.javalec.spring_project_board_dao.BDao;
import com.javalec.spring_project_board_dto.BDto;

public class BReplyViewCommand implements BCommand {

  @Override
  public void execute(Model model) {
    Map<String, Object> map = model.asMap();
    HttpServletRequest request = (HttpServletRequest) map.get("request");

    String bId = request.getParameter("bId");

    BDao dao = new BDao();
    BDto dto = dao.replyView(bId);

    model.addAttribute("replyView", dto);
  }
}
```

* BReplyCommand.java
  * 실제로 답변의 내용을 DB에 넣어주기위한 역할을 하는 클래스

```java
package com.javalec.spring_project_board_command;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.ui.Model;

import com.javalec.spring_project_board_dao.BDao;
import com.javalec.spring_project_board_dto.BDto;

public class BReplyCommand implements BCommand {

  @Override
  public void execute(Model model) {
    Map<String, Object> map = model.asMap();
    HttpServletRequest request = (HttpServletRequest) map.get("request");

    String bId = request.getParameter("bId");
    String bName = request.getParameter("bName");
    String bTitle = request.getParameter("bTitle");
    String bContent = request.getParameter("bContent");
    String bGroup = request.getParameter("bGroup");
    String bStep = request.getParameter("bStep");
    String bIndent = request.getParameter("bIndent");

    BDao dao = new BDao();
    dao.reply(bId, bName, bTitle, bContent, bGroup, bStep, bIndent);
  }
}
```

### DAO 메소드 추가

* replyView() 메소드
  * 실제로 답변을 입력하기 위한 뷰를 출력하기 위한 화면
  * select문을 수행한 뒤, 해당 내용을 dto에 담는다.

```java
...
  public BDto replyView(String strId) {
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
```

* reply() 메소드와, replyShape() 메소드
  * 실제로 댓글을 등록하는 insert문과, 댓글의 깊이를 표시하기 위한 replyShape 메소드
  * 해당 게시글의 답변이 추가되는 경우 stop과 indent를 +1 시켜줌으로써, 리스트 출력시 댓글을 보기 좋게 한다.

```java
...
  public void reply(String bId, String bName, String bTitle, String bContent, String bGroup, String bStep, String bIndent) {
    replyShape(bGroup, bStep);

    Connection connection = null;
    PreparedStatement preparedStatement = null;

    try {
      connection = dataSource.getConnection();
      String query = "insert into mvc_board (bName, bTitle, bContent, bGroup, bStep, bIndent) " +
          "values (?,?,?,?,?,?)";
      preparedStatement = connection.prepareStatement(query);
      preparedStatement.setString(1, bName);
      preparedStatement.setString(2, bTitle);
      preparedStatement.setString(3, bContent);
      preparedStatement.setInt(4, Integer.parseInt(bGroup));
      preparedStatement.setInt(5, Integer.parseInt(bStep) + 1);
      preparedStatement.setInt(6, Integer.parseInt(bIndent) + 1);

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

  private void replyShape(String strGroup, String strStep) {
    Connection connection = null;
    PreparedStatement preparedStatement = null;

    try {
      connection = dataSource.getConnection();
      String query = "update mvc_board set bStep = bStep + 1 where bGroup = ? and bStep > ?";
      preparedStatement = connection.prepareStatement(query);
      preparedStatement.setInt(1, Integer.parseInt(strGroup));
      preparedStatement.setInt(2, Integer.parseInt(strStep));

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
```

### Jsp 페이지

* replyView.jsp
  * 답변을 등록하기 위한 화면

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
  <form action="reply" method="post">
    <input type="hidden" name="bId" value="${replyView.bId}">
    <input type="hidden" name="bGroup" value="${replyView.bGroup}">
    <input type="hidden" name="bStep" value="${replyView.bStep}">
    <input type="hidden" name="bIndent" value="${replyView.bIndent}">
    <tr>
      <td> 번호</td>
      <td> ${replyView.bId} </td>
    </tr>
    <tr>
      <td> 히트</td>
      <td> ${replyView.bHit} </td>
    </tr>
    <tr>
      <td> 이름</td>
      <td><input type="text" name="bName" value="${replyView.bName}"></td>
    </tr>
    <tr>
      <td> 제목</td>
      <td><input type="text" name="bTitle" value="${replyView.bTitle}"></td>
    </tr>
    <tr>
      <td> 내용</td>
      <td><textarea rows="10" name="bContent">${replyView.bContent}</textarea></td>
    </tr>
    <tr>
      <td colspan="2"><input type="submit" value="답변"> <a href="list">목록</a></td>
    </tr>
  </form>
</table>

</body>
</html>
```

* list.jsp
  * 게시물들을 표시하고, indent를 표시함.

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
      <td>
        <c:forEach begin="1" end="${dto.bIndent}">-</c:forEach>
        <a href="contentView?bId=${dto.bId}">${dto.bTitle}</a></td>
      <td>${dto.bDate}</td>
      <td>${dto.bHit}</td>
    </tr>
  </c:forEach>
  <tr>
    <td colspan="5"><a href="writeView">글작성</a></td>
  </tr>
</table>

</body>
</html>
```



