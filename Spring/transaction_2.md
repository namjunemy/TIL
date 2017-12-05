# 23. 트랜잭션(Transaction) 2

## 23-1. Transaction Template

기본적으로 사용한 PlatformTransactionManager 인터페이스 보다 더욱 많이 사용되는 TransactionTemplate에 대해 학습한다. 많이 사용된다는 것은 기존의 방법보다 개발자의 수고가 덜 할 수 있다는 것이다.

[소스코드 repo](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_23_1_ex1_springex)

### TransactionManager 적용된 DAO

* TicketDao.java

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



### TransactionTemplate 적용된 DAO

* TicketDao.java
  * Servlet-context설정 파일에 의해 만들어진 TransactionTemplate의 execute() 메소드와 TransactionCallbackWithoutResult 객체를 통해서 트랜잭션을 처리한다.

```java
package com.javalec.ex.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallbackWithoutResult;
import org.springframework.transaction.support.TransactionTemplate;

import com.javalec.ex.dto.TicketDto;

public class TicketDao {
  JdbcTemplate template;
  TransactionTemplate transactionTemplate;

  public TicketDao() {
    System.out.println(template);
  }

  public void setTransactionTemplate(TransactionTemplate TransactionTemplate) {
    this.transactionTemplate = TransactionTemplate;
  }

  public void setTemplate(JdbcTemplate template) {
    this.template = template;
  }

  public void buyTicket(final TicketDto dto) {
    System.out.println("buyTicket");
    System.out.println("dto.getConsumerId() : " + dto.getConsumerId());
    System.out.println("dto.getAmount() : " + dto.getAmount());

    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      
      @Override
      protected void doInTransactionWithoutResult(TransactionStatus arg0) {
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
      }
    });
  }
}
```

* 변경된 servlet-context.xml

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
  
  <beans:bean name="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
    <beans:property name="transactionManager" ref="transactionManager"/>
  </beans:bean>

  <beans:bean name="dao" class="com.javalec.ex.dao.TicketDao">
    <beans:property name="template" ref="template"/>
    <beans:property name="transactionTemplate" ref="transactionTemplate"/>
  </beans:bean>

</beans:beans>
```

  

## 23-2. 트랜잭션 전파 속성(1)

2개 이상의 트랜잭션이 작동할 때, 기존의 트랜잭션에 참여하는 방법을 결정하는 속성이다.

* 속성 뒤에오는 숫자는 servlet-context 설정을 할 때, PropagationBehavior에 값으로 들어갈 상수이다.

| 트랜잭션 속성                      | 설명                                       |
| ---------------------------- | ---------------------------------------- |
| PROPAGATION_REQUIRED(0)      | Default: 전체 처리                           |
| PROPAGATION_SUPPORTS(1)      | 기존 트랜잭션에 의존                              |
| PROPAGATION_MANDATORY(2)     | 트랜잭션에 꼭 포함 되어야 함 - 트랜잭션이 있는 곳에서 호출해야 함   |
| PROPAGATION_REQUIRES_NEW(3)  | 각각 트랜잭션 처리(별도의 트랜잭션 처리)                  |
| PROPAGATION_NOT_SUPPORTED(4) | 트랜잭션에 포함 하지 않음 - 트랜잭션이 없는 것과 동일 함        |
| PROPAGATION_NEVER(5)         | 트랜잭션에 절대 포함 하지 않음 - 트랜잭션이 있는 곳에서 호출하면 에러 발생 |

### 실습 흐름

* 계속해서 실습하던 Ticket 발권 실습 코드를 두 개의 트랜잭션을 적용시키는 실습을 하기 위해 커맨드객체를 구현하여 컨트롤러에서 커맨드객체로 Dao메소드 실행을 위임하는 MVC형태로 변경시킨다.
* 전체적인 흐름은 다음과 같다.
* Controller에서 기존에 Dao를 @Autowired하던 코드를 제거하고 ITicketCommand 인터페이스를 구현상속한 TicketCommand 클래스를 set한다.
* Controller에서 ticketCommand객체의 execute() 메소드를 실행 한다.
* 실제로 execute() 메소드가 구현되어 있는 TicketCommand에서는 Dao와 TransactionTemplate객체를 내부 필드에 유지하고 트랜잭션처리를 하는 메소드 내에서 Dao의 buyTicket메소드를 실행하게 된다.
* TicketCommand에서 처리하는 트랜잭션이 에러가 나면 전체 트랜잭션이 처리되는 것을 실습하기 위해 buyTicket() 메소드를 두번 실행 시키고, 두번째 메소드 실행시 하드코딩으로 DB exception을 유도한다.
* Dao클래스에는 TransactionTemplate을 통한 트랜잭션 처리 메소드 안에서 실제 Query를 수행하게 된다.
* 결과적으로 Command클래스에서 메소드 호출하는 과정에 트랜잭션 한개, 그리고 실제 DAO의 메소드를 수행할 때 처리되는 트랜잭션 한개, 총 두개의 트랜잭션이 처리되고 있다.
* servlet-context.xml에서는 두 개의 트랜잭션을 생성하는 bean설정과 bean생성에 관한 설정들이 들어가 있다.
  * 여기서, transactionTemplate 빈의 property로 propagationBehavior와 value를 설정한다.
  * propagationBehavior의 기본값은 트랜잭션 속성 정의 표에 있는 상수0 이며, 전체 트랜잭션을 처리하게 된다.
* 따라서, 실습결과 두번째 트랜잭션에서 오류가 발생하면 전체 트랜잭션이 처리가 되서 DB에 정보가 반영되지 않는다.

#### Controller.java

```java
package com.javalec.ex;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import com.javalec.ex.com.ITicketCommand;
import com.javalec.ex.dao.TicketDao;
import com.javalec.ex.dto.TicketDto;

@Controller
public class HomeController {  
  private ITicketCommand ticketCommand;

  @Autowired
  public void setTicketCommand(ITicketCommand ticketCommand) {
    this.ticketCommand = ticketCommand;
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

    ticketCommand.execute(ticketDto);
    model.addAttribute("ticketInfo", ticketDto);

    return "buyTicketEnd";
  }
}
```

#### ticketCommand.java

```java
package com.javalec.ex.com;

import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallback;
import org.springframework.transaction.support.TransactionCallbackWithoutResult;
import org.springframework.transaction.support.TransactionTemplate;

import com.javalec.ex.dao.TicketDao;
import com.javalec.ex.dto.TicketDto;

public class TicketCommand implements ITicketCommand {
  private TicketDao ticketDao;
  private TransactionTemplate transactionTemplate2;
  
  public void setTicketDao(TicketDao ticketDao) {
    this.ticketDao = ticketDao;
  }
  
  public void setTransactionTemplate2(TransactionTemplate transactionTemplate2) {
    this.transactionTemplate2 = transactionTemplate2;
  }
  
  @Override
  public void execute(final TicketDto ticketDto) {
    transactionTemplate2.execute(new TransactionCallbackWithoutResult() {
      @Override
      protected void doInTransactionWithoutResult(TransactionStatus arg0) {
        ticketDao.buyTicket(ticketDto);
        
        ticketDto.setConsumerId("22222222222222222222222222222222222");
        ticketDao.buyTicket(ticketDto);        
      }
    });
  }
}
```

### TicketDao.java

```java
package com.javalec.ex.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallbackWithoutResult;
import org.springframework.transaction.support.TransactionTemplate;

import com.javalec.ex.dto.TicketDto;

public class TicketDao {
  JdbcTemplate template;
  TransactionTemplate transactionTemplate1;

  public TicketDao() {
    System.out.println(template);
  }

  public void setTransactionTemplate1(TransactionTemplate TransactionTemplate) {
    this.transactionTemplate1 = TransactionTemplate;
  }

  public void setTemplate(JdbcTemplate template) {
    this.template = template;
  }

  public void buyTicket(final TicketDto dto) {
    System.out.println("buyTicket");
    System.out.println("dto.getConsumerId() : " + dto.getConsumerId());
    System.out.println("dto.getAmount() : " + dto.getAmount());

    transactionTemplate1.execute(new TransactionCallbackWithoutResult() {

      @Override
      protected void doInTransactionWithoutResult(TransactionStatus arg0) {
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
      }
    });
  }
}
```

#### servlet-context.xml

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
    <beans:property name="password" value="youtPW"/>
  </beans:bean>

  <beans:bean name="template" class="org.springframework.jdbc.core.JdbcTemplate">
    <beans:property name="dataSource" ref="dataSource"/>
  </beans:bean>

  <beans:bean name="transactionManager"
              class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <beans:property name="dataSource" ref="dataSource"/>
  </beans:bean>

  <beans:bean name="transactionTemplate1"
              class="org.springframework.transaction.support.TransactionTemplate">
    <beans:property name="transactionManager" ref="transactionManager"/>
    <beans:property name="propagationBehavior" value="0"></beans:property>
  </beans:bean>
  
  <beans:bean name="transactionTemplate2"
              class="org.springframework.transaction.support.TransactionTemplate">
    <beans:property name="transactionManager" ref="transactionManager"/>
  </beans:bean>

  <beans:bean name="dao" class="com.javalec.ex.dao.TicketDao">
    <beans:property name="template" ref="template"/>
    <beans:property name="transactionTemplate1" ref="transactionTemplate1"/>
  </beans:bean>
  
  <beans:bean name="ticketCommand" class="com.javalec.ex.com.TicketCommand">
    <beans:property name="ticketDao" ref="dao"/>
    <beans:property name="transactionTemplate2" ref="transactionTemplate2"/>
  </beans:bean>

</beans:beans>
```



