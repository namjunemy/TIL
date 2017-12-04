# 22. 트랜잭션(Transaction) 1

## 22-1. 트랜잭션의 개념

논리적 단위로 어떤 한 부분의 작업이 완료되었다 하더라도, 다은 부분의 작업이 완료되지 않을 경우 전체 취소되는 것이다. 이때, 작업이 완료되는 것을 커밋(commit)이라고 하고, 작업이 취소되는 것을 롤백(rollback)이라고 한다.

일상생활에 트랜잭션의 예는 많이 볼 수 있다. 영화 예매를 할 경우 카드 결제 작업과 마일리지 적립 작업은 트랜잭션으로 작동해야 한다. 또한 은행 ATM기도 마찬가지 이다.

## 22-2. 스프링 트랜잭션 사용방법

[소스코드 repo](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_22_2_ex1_springex) 

우선 트랜잭션이 적용되지 않은 상황을 설명한다.

* Card테이블과 Ticket테이블이 DB에 생성되어 있고, Card 테이블은 카드 결제를 기록을 남기기 위한 테이블이고 제약조건이 없다. Ticket 테이블은 티켓 발권 기록을 남기기 위한 테이블이고 발권은 1인당 4장밖에 안되는 DB제약조건이 걸려 있다.
* JSP페이지를 통해 사용자 ID를 입력받고, 티켓 매수를 입력한다.
* Dao의 buyTicket 메소드는 JdbcTemplate의 update메소드를 이용하여 card 테이블과 ticket 테이블에 insert 쿼리를 업데이트 하기 위한 내용이 정의되어 있다.
* 이 상황에서 사용자가 4장 이상에 해당하는 5장, 10장을 사게되면 card 테이블에는 적용이 되지만 ticket 테이블에는 insert 오류가 떨어지고 종료된다.
* 트랜잭션이 적용되지 않아서 전체 결제 프로세스에 치명적인 오류가 발생하는 상황이다.

> 참고로 MySQL에서 CHECK 제약조건은 구문 분석은 되지만, 모든 스토리지 엔진에서 무시된다.
>
> 따라서, MySQL에서 CHECK 제약조건을 구현하려면 Trigger를 이용하자!

  

### 트랜잭션 적용하기

[소스코드 repo](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_22_2_ex2_springex)

* DAO 클래스에 트랜잭션을 적용하기 위해 PlatformTransactionManager 타입의 변수를 선언한다.
* JdbcTemplate과 마찬가지로 setter를 선언한다. Controller 클래스에서 @Autowired 어노테이션으로 DAO 설정에 관련된 Auto Scan을 할 때, servlet-context.xml에 의해 빈이 설정된다.
* 아래의 메소드로 이동하여 TranactionDefinition을 생성하고, TransactionStatus를 저장한다.
* Try-catch문을 통해서 작업을 수행하고, 문제없이 끝난 경우 commit(status);처리를 해주고
* 문제가 있을 경우 rollback(status);처리 해준다.
* TicketDao.java 파일

```java
package com.javalec.ex.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import com.javalec.ex.dto.TicketDto;

public class TicketDao {
  JdbcTemplate template;
  PlatformTransactionManager transactionManager;

  public TicketDao() {
    System.out.println(template);
  }

  public void setTransactionManager(PlatformTransactionManager transactionManager) {
    this.transactionManager = transactionManager;
  }

  public void setTemplate(JdbcTemplate template) {
    this.template = template;
  }

  public void buyTicket(final TicketDto dto) {
    System.out.println("buyTicket");
    System.out.println("dto.getConsumerId() : " + dto.getConsumerId());
    System.out.println("dto.getAmount() : " + dto.getAmount());

    TransactionDefinition definition = new DefaultTransactionDefinition();
    TransactionStatus status = transactionManager.getTransaction(definition);

    try {
      template.update(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
          String query = "insert into card (consumerId, amount) values (?, ?)";
          PreparedStatement pstmt = con.prepareStatement(query);
          pstmt.setString(1, dto.getConsumerId());
          pstmt.setString(2, dto.getAmount());

          return pstmt;
        }
      });

      template.update(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
          String query = "insert into ticket (consumerId, count) values (?, ?)";
          PreparedStatement pstmt = con.prepareStatement(query);
          pstmt.setString(1, dto.getConsumerId());
          pstmt.setString(2, dto.getAmount());

          return pstmt;
        }
      });
      transactionManager.commit(status);
      
    } catch (Exception e) {
      e.printStackTrace();

      transactionManager.rollback(status);
    }
  }
}
```

* Controller.java

```java
package com.javalec.ex;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import com.javalec.ex.dao.TicketDao;
import com.javalec.ex.dto.TicketDto;

@Controller
public class HomeController {
  private TicketDao dao;

  @Autowired
  public void setDao(TicketDao dao) {
    this.dao = dao;
  }

  @RequestMapping("/buyTicket")
  public String buyTicket() {
    return "buyTicket";
  }

  @RequestMapping("/buyTicketCard")
  public String buyTicketCard(TicketDto ticketDto, Model model) {
    System.out.println("buyTicketCard");
    System.out.println("ticketDto : " + ticketDto.getConsumerId());
    System.out.println("ticketDto : " + ticketDto.getAmount());

    dao.buyTicket(ticketDto);
    model.addAttribute("ticketInfo", ticketDto);

    return "buyTicketEnd";
  }
}
```

* servlet-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:beans="http://www.springframework.org/schema/beans"
             xmlns:context="http://www.springframework.org/schema/context"
             xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

  <!-- DispatcherServlet Context: defines this servlet's request-processing 
    infrastructure -->

  <!-- Enables the Spring MVC @Controller programming model -->
  <annotation-driven/>

  <!-- Handles HTTP GET requests for /resources/** by efficiently serving 
    up static resources in the ${webappRoot}/resources directory -->
  <resources mapping="/resources/**" location="/resources/"/>

  <!-- Resolves views selected for rendering by @Controllers to .jsp resources 
    in the /WEB-INF/views directory -->
  <beans:bean
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <beans:property name="prefix" value="/WEB-INF/views/"/>
    <beans:property name="suffix" value=".jsp"/>
  </beans:bean>

  <context:component-scan base-package="com.javalec.ex"/>

  <beans:bean name="dataSource"
              class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <beans:property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <beans:property name="url" value="jdbc:mysql://localhost:3306/springmvcdb"/>
    <beans:property name="username" value="yourID"/>
    <beans:property name="password" value="yourPW"/>
  </beans:bean>

  <beans:bean name="template" class="org.springframework.jdbc.core.JdbcTemplate">
    <beans:property name="dataSource" ref="dataSource"/>
  </beans:bean>
  
  <beans:bean name="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <beans:property name="dataSource" ref="dataSource"/>
  </beans:bean>

  <beans:bean name="dao" class="com.javalec.ex.dao.TicketDao">
    <beans:property name="template" ref="template"/>
    <beans:property name="transactionManager" ref="transactionManager"/>
  </beans:bean>

</beans:beans>
```

