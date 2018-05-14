# MySQL 01

## 데이터베이스의 목적

* 데이터베이스와 스프레드시트의 가장 큰 중요한 차이점은 코딩(프로그래밍 언어)을 통해 접근할 수 있는가이다.
* RDB는 SQL(Structured Query Language)이라는 언어를 통해서 데이터를 제어할 수 있다.
* 프로그래밍 언어와 SQL을 통해서 데이터베이스에 저장된 데이터를 공유할 수 있고, 분석할 수 있다.
  * 웹과 앱, 빅데이터와 머신러닝 등의 기술을 통해서
* 웹사이트의 정보를 데이터베이스에 담아서 전세계의 모든 사람들과 공유할 수 있다. 이것은 데이터베이스를 사용하는 수 많은 방법 중 하나에 불과하다.

## 설치

* 설치에 대한 정리는 링크로 대신한다.
  * https://opentutorials.org/course/3161/19532

## MySQL의 구조

* database server
  * database(=schema)들을 저장하는 서버
* database(=schema)
  * 연관된 table을 grouping 하는 폴더의 개념
* table
  * 실제 데이터가 저장되는 표
* 정리하자면 MySQL을 설치한 것은 Database 서버를 설치한 것이고, 이 프로그램이 가지고 있는 기능을 이용해서 데이터와 관련된 여러가지 작업을 한다.

## 서버 접속

* db는 자체적인 보안 체계를 갖추고 있기 때문에 안전하게 데이터를 저장할 수 있다.
* 그리고 권한을 통해서 사용자가 차등적으로 데이터를 관리할 수 있는 권한과 범위를 지정할 수 있다.
* `$ mysql -u root -p`

## 스키마(schema)의 사용

* `$ CREATE DATEABASE testdb;`
* `$ DROP DATEBASE testdb;`
* `$ show databases;` 또는 `$ show schemas;`
* `$ USE testdb;`

## SQL과 테이블의 구조

* Structured
  * 관계형 DB가 기본적으로 table의 형식으로 데이터를 정리 정돈하는데, 이를 구조화 되었다라고 표현한다.
* Query
  * DB에 데이터를 관리하기 위해 요청하는 것을 질의 한다라고 표현한다.
* Language
  * DB 서버와 Client가 서로 알아 듣기 위한 언어이다.
* table
  * row, record, 행
    * 데이터 자체가 저장된다
  * column, 열
    * 데이터의 타입 또는 구조

## 테이블의 생성

```mysql
CREATE TABLE topic (
  id          INT(11)      NOT NULL AUTO_INCREMENT,
  title       VARCHAR(100) NOT NULL,
  description TEXT         NULL,
  created     DATETIME     NOT NULL,
  author      VARCHAR(15)  NULL,
  profile     VARCHAR(200) NULL,
  PRIMARY KEY (id)
);
```

* 경우에 따라서, 아래의 에러가 발생하는 경우가 있다.
  * 해결방안
  * `$ SET PASSWORD = PASSWORD('password');`

```shell
ERROR 1820(HY000): You must reset your password using ALTER USER statement before executing this statement.
```

## CRUD

* Create
  * 생성
* Read
  * 조회

* Update
  * 수정
* Delete
  * 삭제
* Create, Read는 데이터베이스라면 반드시 가지고 있어야 하는 개념이다.(input, output)
* Update, Delete는 분야에 따라 없을 수도 있다. 예를 들면, 역사의 이야기, 회계에서 거래내역을 다루는 경우