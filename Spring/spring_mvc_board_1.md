# 16. 스프링MVC 게시판 1

## 16-1. 프로젝트 설계

프로젝트의 전체적인 설계를 한다.

* 클라이언트의 요청을 Dispatcher에서 받아서 컨트롤러로 넘겨준다.
*  auto scan을 통해 요청을 처리할 컨트롤러가 정해지면 해당 로직을 수행 할 command를 결정한다.
* 그리고 나서 DAO(Data Access Object)를 통해서 DB에 접근한 후, DTO(Data Transfer Object)를 통해 정보를 가져온다.
* 다음으로 컨트롤러에서 화면 UI를 통해 응답한다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_1.png?raw=true)

  

## 16-2. Database 구축

데이터베이스를 구축한다.

* SQL 작성 (MySQL)
  * Group은 하나의 원 댓글 아래 달린 모든 전체 답글을 뜻한다.
  * Step은 Group에서 몇번째 위치 할 것인가를 의미한다.
  * Indent는 들여쓰기의 횟수를 의미한다.

```sql
CREATE TABLE mvc_board (
  bId		INTEGER NOT NULL,
  bName		VARCHAR(20),
  bTitle 	VARCHAR(100),
  bContent	VARCHAR(300),
  bDate		DATETIME DEFAULT CURRENT_TIMESTAMP,
  bHit 		INTEGER DEFAULT '0',
  bGroup	INTEGER,
  bStep		INTEGER,
  bIndent	INTEGER,
  PRIMARY KEY(bId)
);

ALTER TABLE mvc_board MODIFY COLUMN bId INTEGER NOT NULL AUTO_INCREMENT COMMENT 'm_no';

INSERT INTO mvc_board(bName, bTitle, bContent, bHit, bGroup, bStep, bIndent)
VALUE ('abcd', 'is title', 'is content', 0, 0, 0, 0);

select * from mvc_board;

drop table mvc_board;
```

  

## 16-3. 프로젝트 생성

- new -> project -> spring legacy project 선택

- project name 입력 후, Template항목에서 Spring MVC 템플릿 선택

- 프로젝트 진행을 위해 한글 필터 처리를 web.xml에 해준다.

  - web.xml

  ```xml
  ...
    <filter>
      <filter-name>encodingFilter</filter-name>
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

      <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
      </init-param>
    </filter>

    <filter-mapping>
      <filter-name>encodingFilter</filter-name>
      <url-pattern>/*</url-pattern>
    </filter-mapping>
  ...
  ```

  ​