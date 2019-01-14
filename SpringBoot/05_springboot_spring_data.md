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

* 지원하는 DBCP

  * HikariCP(기본)
    * https://github.com/brettwooldridge/HikariCP
  * Tomcat CP
    * https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html
  * Commons DBCP2
    * https://commons.apache.org/proper/commons-dbcp/

* DBCP 설정

  * spring.datasource.hikari.*
  * spring.datasource.tomcat.*
  * spring.datasource.dbcp2.*

* MySQL 커넥터 의존성 추가

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
    * 소스 코드 공개 의무 여부 확인

  