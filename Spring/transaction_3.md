# 24. 트랜잭션(Transaction) 3

트랜잭션의 상호 포함 관계에 따라서 두개 이상의 트랜잭션 전파 속성을 상수 값의 설정으로 처리할 수 있다.

## 24-2. 트랜잭션 전파 속성

- 속성 뒤에오는 숫자는 servlet-context 설정을 할 때, PropagationBehavior에 값으로 들어갈 상수이다.

| 트랜잭션 속성                      | 설명                                       |
| ---------------------------- | ---------------------------------------- |
| PROPAGATION_REQUIRED(0)      | Default: 전체 처리                           |
| PROPAGATION_SUPPORTS(1)      | 기존 트랜잭션에 의존                              |
| PROPAGATION_MANDATORY(2)     | 트랜잭션에 꼭 포함 되어야 함 - 트랜잭션이 있는 곳에서 호출해야 함   |
| PROPAGATION_REQUIRES_NEW(3)  | 각각 트랜잭션 처리(별도의 트랜잭션 처리)                  |
| PROPAGATION_NOT_SUPPORTED(4) | 트랜잭션에 포함 하지 않음 - 트랜잭션이 없는 것과 동일 함        |
| PROPAGATION_NEVER(5)         | 트랜잭션에 절대 포함 하지 않음 - 트랜잭션이 있는 곳에서 호출하면 에러 발생 |

#### servlet-context.xml 의 트랜잭션 전파 속성 설정 부분

```xml
...
<beans:bean name="transactionTemplate1"
            class="org.springframework.transaction.support.TransactionTemplate">
  <beans:property name="transactionManager" ref="transactionManager"/>
  <beans:property name="propagationBehavior" value="0"></beans:property>
</beans:bean>

<beans:bean name="transactionTemplate2"
            class="org.springframework.transaction.support.TransactionTemplate">
  <beans:property name="transactionManager" ref="transactionManager"/>
  <beans:property name="propagationBehavior" value="0"></beans:property>
</beans:bean>
...
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

#### 