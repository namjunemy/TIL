# 28. Mybatis-1

Mybatis 라이브러리는 DB에 접근하기 위해 존재하는 java 코드들을 xml코드로 분리 해낸다.

## 28.1 Mybatis 사용을 위한 기본 설정

Mybatis 실습을 위해 간단한 게시판 예제를 작성한다. 전체 리스트 화면, 글 작성 기능, 글 삭제 기능을 포함하고 있다.

* Database table 생성

```sql
create table board ()
  mId integer,
  mWriter varchar(100),
  mContent varchar(200),
  primary key(mId)
);

alter table board modify column mId integer auto_increment;
```

* pom.xml dependency 추가
  * spring-jdbc
  * mybatis
  * mybatis-spring

```xml
...
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.1.4.RELEASE</version>
  </dependency>

  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.2.8</version>
  </dependency>
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.2.2</version>
  </dependency>
...
```

* ContentDao.java

```java
package com.javalec.mybatis.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.ArrayList;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.core.PreparedStatementSetter;

import com.javalec.mybatis.dto.ContentDto;

public class ContentDao implements IDao {
  JdbcTemplate template;

  @Autowired
  public void setTemplate(JdbcTemplate template) {
    this.template = template;
  }

  @Override
  public ArrayList<ContentDto> listDao() {
    String query = "select * from board order by mId desc";
    ArrayList<ContentDto> dtos = (ArrayList<ContentDto>) template.query(query,
        new BeanPropertyRowMapper<ContentDto>(ContentDto.class));

    return dtos;
  }

  @Override
  public void writeDao(final String mWriter, final String mContent) {
    System.out.println("writeDao()");
    this.template.update(new PreparedStatementCreator() {
      @Override
      public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
        String query = "insert into board (mWriter, mContent) values (?,?)";
        PreparedStatement pstmt = con.prepareStatement(query);
        pstmt.setString(1, mWriter);
        pstmt.setString(2, mContent);

        return pstmt;
      }
    });
  }

  @Override
  public ContentDto viewDao(String strId) {
    System.out.println("viewDao()");

    String query = "select * from board where mId = " + strId;

    return template.queryForObject(query, new BeanPropertyRowMapper<ContentDto>(ContentDto.class));
  }

  @Override
  public void deleteDao(final String bId) {
    System.out.println("delete()");

    String query = "delete from board where mId = ?";
    this.template.update(query, new PreparedStatementSetter() {
      @Override
      public void setValues(PreparedStatement ps) throws SQLException {
        ps.setInt(1, Integer.parseInt(bId));
      }
    });
  }
}
```

