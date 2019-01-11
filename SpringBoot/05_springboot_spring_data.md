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