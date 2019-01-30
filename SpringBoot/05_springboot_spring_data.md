# 5. 스프링 데이터

> [백기선 - 스프링 부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)

## 1. 소개

스프링 부트가 지원하는 스프링 데이터 연동 기술에 대해 알아본다.

### SQL DB 지원

* 인메모리 데이터베이스 지원
* DataSource 설정
* DBCP 설정
* JDBC 사용하기
* 스프링 데이터 JPA 사용하기
* jOOQ 사용하기
* 데이터베이스 초기화
* 데이터베이스 마이그레이션 툴 연동하기

### NoSQL 지원

* Redis(Key/Value)
* MongoDB(Document)
* Neo4J(Graph)
* *Gemfire (IMDG)*
* *Solr (Search)*
* *Elasticsearch (Search & Analytics)*
* *Cassandra*
* *Couchbase*
* *LDAP*
* *InfluxDB*

## 2. 인메모리 데이터베이스

* 지원하는 인-메모리 데이터베이스
  * H2(추천, 콘솔 때문에...)
  * HSQL
  * Derby
* Spring-JDBC가 classpath에 있으면 자동 설정이 필요한 빈을 설정해준다.
  * DataSource
  * JdbcTemplate
* 인-메모리 데이터베이스 기본 연결 정보 확인하는 방법
  * URL: "testdb"
  * username: "sa"
  * password: ""
* H2 콘솔 사용하는 방법
  * spring-boot-devtools를 추가하거나
  * spring.h2.console.enabled=true 만 추가
  * /h2-console로 접속(이 path도 바꿀 수 있음)
* 실습 코드
  * https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/b76fb673efed4a77559b7da81c2d6e6c7d886f01
  * CREATE TABLE USER (id INTEGER NOT NULL, name VARCHAR(255), PRIMARY KEY(id))
  * INSERT INTO USER VALUES (1, 'nj')

## 3. MySQL

스프링 부트가 지원하는 DBCP 기능과 MySQL 설정법

* 지원하는 DBCP

  * HikariCP(기본)
    * https://github.com/brettwooldridge/HikariCP
      * default
        * autoCommit
          * true
        * connectionTimeout
          * 30 seconds
        * maximumPoolSize
          * 10
        * 등등
  * Tomcat CP
    * https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html
  * Commons DBCP2
    * https://commons.apache.org/proper/commons-dbcp/

* DBCP 설정

  * 설정은 각 DBCP의 문서를 보는 것이 가장 좋다.
  * spring.datasource.{DBCP이름}.*
    * spring.datasource.hikari.*
      * spring.datasource.hikari.maximum-pool-size=4
    * spring.datasource.tomcat.*
    * spring.datasource.dbcp2.*

* MySQL 커넥터 의존성 추가

  ```groovy
  compile group: 'mysql', name: 'mysql-connector-java', version: '8.0.13'
  ```

* MYSQL 추가(도커 사용)

  * docker run -p 3306:3306 --name mysql_boot -e MYSQL_ROOT_PASSWORD=1 -e MYSQL_DATABASE=springboot -e MYSQL_USER=keesun -e MYSQL_PASSWORD=pass -d mysql
  * docker exec -i -t mysql_boot bash
  * mysql -u root -p

* MySQL용 Datasource 설정

  * spring.datasource.url=jdbc:mysql://localhost:3306/springboot?useSSL=false
  * spring.datasource.username=nj
  * spring.datasource.password=pass

* MySQL 접속시 에러

  * MySQL 5.* 최신 버전 사용할 때
    * 문제
      * Sat Jul 21 11:17:59 PDT 2018 WARN: Establishing SSL connection without server’s identity verification is not recommended. **According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn’t set.** For compliance with existing applications not using SSL the **verifyServerCertificate property is set to ‘false’**. You need either to explicitly disable SSL by setting **useSSL=false**, or set **useSSL=true and provide truststore** for server certificate verification.
    * 해결
      * jdbc:mysql:/localhost:3306/springboot?**useSSL=false**
  * MySQL 8.* 최신 버전 사용할 때
    * 문제
      * com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Public Key Retrieval is not allowed
    * 해결
      * jdbc:mysql:/localhost:3306/springboot?useSSL=false&**allowPublicKeyRetrieval=true**
  * MySQL 라이센스(GPL) 주의
    * MySQL 대신 MariaDB 사용 검토
      * 컨테이너를 mariadb로 띄움 
        * docker run -p 3306:3306 --name mysql_boot -e MYSQL_ROOT_PASSWORD=1 -e MYSQL_DATABASE=springboot -e MYSQL_USER=keesun -e MYSQL_PASSWORD=pass -d mariadb
    * 소스 코드 공개 의무 여부 확인

## 4. PostgreSQL

* 의존성 추가

  ```groovy
  compile group: 'org.postgresql', name: 'postgresql', version: '42.2.5'
  ```

* PostgreSQL 설치 및 서버 실행 (docker)

  ```groovy
  docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=keesun -e POSTGRES_DB=springboot --name postgres_boot -d postgres
  
  docker exec -i -t postgres_boot bash
  
  su - postgres
  
  psql springboot
  
  데이터베이스 조회
  \list
  
  테이블 조회
  \dt
  
  쿼리
  SELECT * FROM account;
  ```

* PostgreSQL 연동시 경고 메세지 해결
  * driver가 createClob() 메서드를 지원하지 않아서 나는 에러
  * 경고
    * org.postgresql.jdbcPgConnection.createClob() is not yet implemented
  * 해결(application.yml)
    * spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true

## 5. Spring-Data-JPA

### ORM(Object-Relational Mapping)과 JPA(JAVA Persistence API)

* 객체와 릴레이션을 맵핑할 때 발생하는 개념적 불일치를 해결하는 프레임워크
* http://hibernate.org/orm/what-is-an-orm/

* JPA: ORM을 위한 자바(EE) 표준

### 스프링 데이터 JPA

*  Repository 빈 자동 생성
*  쿼리 메소드 자동 구현
*  @EnableJpaRespositories(스프링 부트가 자동으로 설정 해줌)
*  우리가 스프링 데이터 JPA를 쓰지만 그 밑단에는
   *  Spring Data JPA -> JPA -> Hibernate -> Datasource 의 흐름이 존재한다.
   *  따라서 우리는 spring data jdbc의 기능을 전부 다 사용할 수 있고,
   *  추가로 Spring Data JPA, JPA, Hibernate의 기능들도 사용할 수 있다.
*  추가된 의존성을 확인해보면 쉽게 이해할 수 있다.
   *  spring-data-jpa가 추가되고, hibernate도 추가된다.
   *  spring-data-jpa안에 추가된 spring-orm으로 jpa를 쓰는 걸 확인할 수 있고,
   *  JPA 구현체인 hibernate-core를 의존성을 통해서 JPA 2.2 구현체인 것을알 수 있다.

![](https://github.com/namjunemy/TIL/blob/master/SpringBoot/img/04_jpa_dependency.PNG?raw=true)

#### 스프링 데이터 JPA 의존성 추가

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

#### 스프링 데이터 JPA 사용하기

* @Entity 클래스 만들기
  * lombok을 사용해도 된다.

```java
package io.namjune.springbootconceptandutilization.account;

import java.util.Objects;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Account {

    @Id
    @GeneratedValue
    private Long id;
    private String username;
    private String password;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Account account = (Account) o;
        return Objects.equals(id, account.id) &&
            Objects.equals(username, account.username) &&
            Objects.equals(password, account.password);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, username, password);
    }
}
```

* Repository 만들기

```java
package io.namjune.springbootconceptandutilization.account;

import java.util.Optional;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface AccountRepository extends JpaRepository<Account, Long> {

    Optional<Account> findByUsername(String username);
}
```

### 스프링 데이터 Repository 테스트 만들기

* H2 DB를 테스트 scope 의존성에 추가하기

* @DataJpaTest (**슬라이스 테스트**) 작성

  * @DataJpaTest가 아닌 @SpringBootTest를 사용하면 integration 테스트이다. 그 의미는 @SpringBootApplication을 찾아서 애플리케이션 코드의 모든 빈을 다 등록시킨다.
  * @SpringBootTest로 아래의 테스트를 동작시키면, 실제 Application에서 사용중인 데이터베이스 커넥션이 연결된 결과를 확인할 수 있다.
  * 하지만, repository 테스트 할 때는 embedded H2를 사용하는 것을 추천한다. @SpringBootTest로 실제 DB로 테스트하는 것보다 빠르고, 실제 DB를 사용한다고 했을때 테스트 코드에서 데이터를 변경하면 실제 DB의 데이터가 변경되기 때문에  별도의 Test DB 설정을 가져가야 한다.(test용 application.yml을 별도로 가져가거나, @SpringBootTest(properties = "spring.datasource.url='~~~'")의 형태로 별도의 Test DB 설정을 유지)
    * 이 방법보다는 테스트시 embedded H2를 사용하는 것(슬라이스 테스트를 하는 것)이 쉽고 안전하다.
  * 테스트시 H2 db를 사용하는지 확인
    * **try resource를 사용하면 finally 블록에서 자원 해제 동작을 구현하지 않아도, 자원을 종료해준다.**

  ```java
  @RunWith(SpringRunner.class)
  @DataJpaTest
  public class AccountRepositoryTest {
  
      @Autowired
      DataSource dataSource;
  
      @Autowired
      JdbcTemplate jdbcTemplate;
  
      @Autowired
      AccountRepository accountRepository;
  
      @Test
      public void dependencyInjection() throws SQLException {
          try(Connection connection = dataSource.getConnection()) {
              DatabaseMetaData metaData = dataSource.getConnection().getMetaData();
              System.out.println(metaData.getURL());
              System.out.println(metaData.getDriverName());
              System.out.println(metaData.getUserName());
          }
      }
  }
  ```

  ```text
  jdbc:h2:mem:8ac79464-2386-47d1-98d6-e48aca1e7ae9
  H2 JDBC Driver
  SA
  ```

  * Account Repository 테스트
    * Optional<>로 받으면 결과값 검증을 isEmpty(), isNotEmpty()로 해야한다.

  ```java
  package io.namjune.springbootconceptandutilization.account;
  
  import java.util.Optional;
  import org.springframework.data.jpa.repository.JpaRepository;
  import org.springframework.stereotype.Repository;
  
  @Repository
  public interface AccountRepository extends JpaRepository<Account, Long> {
  
      Optional<Account> findByUsername(String username);
  }
  ```

  ```java
  @Test
      public void account() {
          Account account = new Account();
          account.setUsername("nj");
          account.setPassword("1234");
  
          Account newAccount = accountRepository.save(account);
          assertThat(newAccount).isNotNull();
  
          Optional<Account> existingAccount = accountRepository.findByUsername(newAccount.getUsername());
          assertThat(existingAccount).isNotEmpty();
  
          Optional<Account> notExistingAccount = accountRepository.findByUsername("test");
          assertThat(notExistingAccount).isEmpty();
      }
  ```

### 데이터베이스 초기화

* JPA를 사용한 데이터베이스 초기화

  * spring.jpa.hibernate.ddl-auto

    * : create
      * SessionFactory 시작시 테이블 drop -> create
    * : create-drop
      * create 의 동작 + SessionFactory 종료시 테이블 삭제
    * : update
      * SessionFactory 시작시 컬럼 추가/삭제 수행. 데이터 삭제 안함
    * : validate
      * SessionFactory 시작시 테이블 컬럼 확인해서 @Entity와 매핑 다르면 예외 발생
      * @Entity에 추가된 컬럼은 데이터베이스에 추가 적용 되지만, 기존의 커럼이 바뀐 경우는 적용 안된다. 예를 들어 name -> username으로 바뀐 경우 hibernate에서는 알 수 없다.
    * 보통 개발단에서는 update를 사용하고, 운영단에서는 validate로 사용한다.

  * spring.jpa.generate-ddl=true로 설정 해줘야 동작함.

    * 시작시 스키마 초기화 여부 설정으로 default false로 설정 되어 있음

  * spring.jpa.show-sql

    * default: false

    * SQL 문장의 로깅 활성화 여부

* SQL 스크립트를 사용한 데이터베이스 초기화

  * JPA를 사용하지 않고 DB 초기화를 진행할 수 있다.
  * ddl-auto: validation와 generate-ddl: false를 설정하면 초기화가 진행되지 않지만, sql파일을 resources 아래에 위치 시키면 초기화를 진행할 수 있다. 순서는 schema.sql -> data.sql 이다.
    * schema.sql 또는 schema-${platform}.sql
    * data.sql 또는 data-${platform}.sql
    * ${platform} 값은 spring.datasource.platform 으로 설정 가능