# 데이터베이스

* SQL
  * 테이블 형식의 데이터베이스
  * MySQL, PostgreSQL, Aurora, Sqlite
  * 학습에서는 Sqlite를 사용한다.
* NoSQL
  * MongoDB, DynamoDB
* In Memory DB
  * 말 그대로 메모리 안에 데이터가 저장이 된다. 즉, 휘발성을 가지고 있다. 그런데도 퍼포먼스가 좋기 때문에 사용한다.
  * Redis, Memcashed 

  

## SQL 쿼리 기초

* 삽입
  * insert users ('name') values ('alice');
* 조회
  * select * from users; 
* 수정
  * update users set name = 'bek' where id = 1;
* 삭제
  * delete from users where id = 1;

  

## ORM

* 데이터베이스를 객체로 추상화해 논것을 ORM(Object Relational Mapping)이라고 한다.
* 쿼리를 직접 작성하는 대신 ORM의 메소드로 데이터를 관리할 수 있는 것이 장점이다.
* 노드에서 SQL ORM은 시퀄라이져(Sequelize)가 있다.

  

## Node.js의 ORM 시퀄라이져

node의 ORM시퀄라이져를 사용하면 다음과 같이 쿼리문을 메소드형태로 바꾸어서 사용할 수 있다.

* insert users ('name') values ('alice');
  * User.create({name: 'alice'});
* select * from users;
  * User.findAll();
* update users set name = 'bek' where id = 1;
  * User.update({name: 'bek'}, {where: {id: 1}});
* delete from users where id = 1;
  * User.destory({where: {id: 1}});

메소드 앞의 User는 ORM에서 모델이라고 한다.

### 모델

* 데이터베이스 테이블을 ORM으로 추상화한것을 모델이라고 한다.

* 우리가 사용할 유저의 모델을 만들어보자

  * sequelize ORM에서 제공하는 메소드를 사용한다.
  * sequelize.define(): 모델 정의
  * sequelize.sync(): 데이터베이스 연동

* 필요 모듈 설치

  * ORM sequelize와 DB sqlite3을 설치한다.
  * `npm i sequelize sqlite3 --save`

* models.js 파일 생성

  * sequelize를 이용하여 모델을 정의한다.
  * sequelize모듈을 추출하고 new를 통하여 객체를 생성한다.
  * 이 때, 생성자로 dialect 속성으로 sqlite를 파일형태로 저장을 할 것이므로 db로 파일의 경로를 넘겨준다.
  * 다음으로 sequelize의 define을 통해서 User모델을 만든다. 속성은 name을 가지게 하고 데이터 타입은 Sequelize.STRING 타입으로 사용한다.
  * Sequelize.STRING 타입으로 생성하면 기본으로 varchar에 length 255로 기본 설정 된다.

  ```javascript
  const Sequelize = require('sequelize');
  const sequelize = new Sequelize({
    dialect: 'sqlite',
    db: './db.sqlite'
  });

  const User = sequelize.define('User', {
    name: Sequelize.STRING
  });

  module.exports = {Sequelize, sequelize, User};
  ```

  ​