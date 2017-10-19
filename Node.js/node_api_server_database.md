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

