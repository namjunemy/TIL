# 21.스프링 JDBC

16강 ~ 20강을 통해 스프링 MVC 게시판을 만든 것을 토대로 JDBC로 적용하는 작업을 한다.

## 21-1.JDBC를 이용한 반복코드 줄이기

DAO 객체를 이용해서 Database의 데이터를 이용한다. 이때, 매번 같은 동작을 반복하는 부분이 있다.(드라이버 로드, 커넥션 생성 및 DB연결, SQL 실행, 자원 해제) 이런 반복적인 작업을 스프링에서는 간단하게 처리 할 수 있다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_9.png?raw=true)

  

## 21-2.Spring 빈을 이용한 코드 간소화

Spring 빈을 이용하여 Database관련 클래스들을 생성하고 조립해 본다. 

* JdbcTemplate 빈을 만들고, 안에 포함된 Datasource 빈을 Java파일에서 가져다 쓴다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_10.png?raw=true)



### 실습

* pom.xml에 Spring-jdbc 의존성 추가

```xml
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>4.1.4.RELEASE</version>
    </dependency>
```

* 컨트롤러에 JdbcTemplate 추가

  * JdbcTemplate.java
  * Autowired 어노테이션 선언을 통해 Auto Scan시 setter가 자동으로 빈을 설정하게 해준다.

  ```java
  @Controller
  public class BController {
    BCommand command;

    @Autowired
    public void setTemplate(JdbcTemplate template) {
      Constant.template = template;
    }

    ...	
  ```

  * Constant.java
    * 전체 클래스에서 JDBC템플릿을 쓰기 위한 Constant 클래스

  ```java
  package com.javalec.spring_project_board_util;

  import org.springframework.jdbc.core.JdbcTemplate;

  public class Constant {
    public static JdbcTemplate template;
  }
  ```


* 스프링 설정파일에서 빈 생성하기
  * WEB-INF/spring/appServlet/servlet-context.xml
  * dataSource 빈을 생성하고 DB에 관련된 property 값을 넣는다.
  * JdbcTemplate 빈을 생성하고 필드값으로 dataSource를 참조하게 한다.

```xml
...

  <beans:bean name="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <beans:property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <beans:property name="url" value="jdbc:mysql://localhost:3306/springmvcdb"/>
    <beans:property name="username" value="yourID"/>
    <beans:property name="password" value="yourPW"/>
  </beans:bean>
  
  <beans:bean name="template" class="org.springframework.jdbc.core.JdbcTemplate">
    <beans:property name="dataSource" ref="dataSource"></beans:property>
  </beans:bean>
...
```

  

## 21-3.JDBC를 이용한 리스트 목록 만들기

JdbcTemplate의 메소드를 이용하여 코드를 간단하게 변경할 수 있다.

* JdbcTemplate 적용 하기 전 BDao.java

```java
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
      if (resultSet != null)
        resultSet.close();
      if (preparedStatement != null)
        preparedStatement.close();
      if (connection != null)
        connection.close();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
  return dtos;
}
```

* JdbcTemplate 적용 후 BDao.java
  * 기본 생성자에 위의 과정을 통해 생성한 JdbcTemplate를 넣어준다.
  * template의 query메소드를 통해서 query를 날린다. 여기서 사용할 객체를 넘겨주기 위해 BenPropertyRowMapper를 통해서 BDto객체를 만들어서 넘긴다.
  * ArrayList로 형변환해서 return 한다.

```java
public class BDao {
  DataSource dataSource;
  JdbcTemplate template = null;

  public BDao() {
    template = Constant.template;
  }
  
  public ArrayList<BDto> list() {
    String query = "select bId, bName, bTitle, bContent, bDate, bHit, bGroup, bStep, bIndent"
        + " from mvc_board order by bGroup desc, bStep asc";
    return (ArrayList<BDto>) template.query(query, new BeanPropertyRowMapper<BDto>(BDto.class));
  }
```



## 21-4.insert, update, delete 처리하기

* 위에 JdbcTemplate설정을 했다면, Dao의 나머지 메소드도 변경한다.
* 변경하기 이전의 Dao 클래스

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
      dataSource = (DataSource) context.lookup("java:comp/env/jdbc/springmvcdb");
    } catch (NamingException e) {
      e.printStackTrace();
    }
  }

  public void reply(String bId, String bName, String bTitle, String bContent, String bGroup, String bStep,
                    String bIndent) {
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
        if (resultSet != null)
          resultSet.close();
        if (preparedStatement != null)
          preparedStatement.close();
        if (connection != null)
          connection.close();
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
    return dtos;
  }
}
```

* JdbcTemplate의 메소드를 통해서 아래와 같이 반복작업이 없어지고 간결해진 코드를 만들 수 있다.

```java
package com.javalec.spring_project_board_dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.ArrayList;

import javax.sql.DataSource;

import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.core.PreparedStatementSetter;

import com.javalec.spring_project_board_dto.BDto;
import com.javalec.spring_project_board_util.Constant;

public class BDao {
  DataSource dataSource;
  JdbcTemplate template = null;

  public BDao() {
    template = Constant.template;
  }

  public void reply(final String bId, final String bName, final String bTitle, final String bContent,
      final String bGroup, final String bStep, final String bIndent) {
    replyShape(bGroup, bStep);
    String query = "insert into mvc_board (bName, bTitle, bContent, bGroup, bStep, bIndent) " + "values (?,?,?,?,?,?)";
    template.update(query, new PreparedStatementSetter() {
      @Override
      public void setValues(PreparedStatement ps) throws SQLException {
        ps.setString(1, bName);
        ps.setString(2, bTitle);
        ps.setString(3, bContent);
        ps.setInt(4, Integer.parseInt(bGroup));
        ps.setInt(5, Integer.parseInt(bStep) + 1);
        ps.setInt(6, Integer.parseInt(bIndent) + 1);
      }
    });
  }

  private void replyShape(final String strGroup, final String strStep) {
    String query = "update mvc_board set bStep = bStep + 1 where bGroup = ? and bStep > ?";
    template.update(query, new PreparedStatementSetter() {
      @Override
      public void setValues(PreparedStatement ps) throws SQLException {
        ps.setInt(1, Integer.parseInt(strGroup));
        ps.setInt(2, Integer.parseInt(strStep));
      }
    });
  }

  public BDto replyView(String strId) {
    String query = "select * from mvc_board where bId = " + strId;
    return template.queryForObject(query, new BeanPropertyRowMapper<BDto>(BDto.class));
  }

  public void delete(final String bId) {
    String query = "delete from mvc_board where bId = ?";
    template.update(query, new PreparedStatementSetter() {
      @Override
      public void setValues(PreparedStatement ps) throws SQLException {
        ps.setString(1, bId);
      }
    });
  }

  public void modify(final String bId, final String bName, final String bTitle, final String bContent) {
    String query = "update mvc_board set bName = ?, bTitle = ?, bContent = ? where bId = ?";
    template.update(query, new PreparedStatementSetter() {
      @Override
      public void setValues(PreparedStatement ps) throws SQLException {
        ps.setString(1, bName);
        ps.setString(2, bTitle);
        ps.setString(3, bContent);
        ps.setInt(4, Integer.parseInt(bId));
      }
    });
  }

  public BDto contentView(String strId) {
    upHit(strId);

    String query = "select * from mvc_board where bId = " + strId;
    return template.queryForObject(query, new BeanPropertyRowMapper<BDto>(BDto.class));
  }

  private void upHit(final String bId) {
    String query = "update mvc_board set bHit = bHit + 1 where bId = ?";
    template.update(query, new PreparedStatementSetter() {
      @Override
      public void setValues(PreparedStatement ps) throws SQLException {
        ps.setInt(1, Integer.parseInt(bId));
      }
    });
  }

  public void write(final String bName, final String bTitle, final String bContent) {
    template.update(new PreparedStatementCreator() {
      @Override
      public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
        String query = "INSERT INTO mvc_board(bName, bTitle, bContent, bHit, bGroup, bStep, bIndent)"
            + "VALUE (?, ?, ?, 0, 0, 0, 0)";
        PreparedStatement pstmt = con.prepareStatement(query);
        pstmt.setString(1, bName);
        pstmt.setString(2, bTitle);
        pstmt.setString(3, bContent);
        return pstmt;
      }
    });
  }

  public ArrayList<BDto> list() {
    String query = "select bId, bName, bTitle, bContent, bDate, bHit, bGroup, bStep, bIndent"
        + " from mvc_board order by bGroup desc, bStep asc";
    return (ArrayList<BDto>) template.query(query, new BeanPropertyRowMapper<BDto>(BDto.class));
  }
}

```

