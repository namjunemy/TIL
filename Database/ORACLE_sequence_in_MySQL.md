# ORACLE sequence in MySQL

MySQL에서는 컬럼의 AUTO INCREMENT 설정을 통해서 쉽게 ORACLE의 sequence.nextval를 대신한다.

MySQL에서 ORACLE의 sequence.currval을 대신하기 위한 여러 시도를 해본 결과를 정리한다.

* ORACLE에서 bId에 sequence.nextval로 값을 넣기 위한 설정이 되어있고, bGroup에 sequence.currval을 넣으려고 하는 상황이라고 가정한다.

  ```sql
  create sequence mvc_board_seq;

  INSERT INTO mvc_board(bId, bName, bTitle, bContent, bHit, bGroup, bStep, bIndent)
  VALUES (mvc_board_seq.nextval, 'is title', 'is content', 0, mvc_board_seq.currval, 0, 0);
  ```

* 이 것을 MySQL로 바꾸면, sequence.nextval는 auto increment로 대신하면서, insert에서 bId를 제외시키는 방법이 있다.

  ```sql
  create table mvc_board (
    bId INTEGER NOT NULL AUTO INCREMENT,
    ...
  )
  ```

* sequence.currval을 대신하려면, 현재 테이블의 max(bId)를 찾아서 넣어주는 방법이 있다.

  * 하지만, MySQL에서는 insert, update, delete에서 서브쿼리로 동일한 테이블의 조건을 사용시 에러가 발생한다.
  * 아래와 같은 경우이다.

  ```sql
  INSERT INTO mvc_board(bName, bTitle, bContent, bHit, bGroup, bStep, bIndent)
  VALUES ('abcd', 'is title', 'is content', 0, (select max(bId) from mvc_board), 0, 0);
  ```

* 해결 방법은 서브쿼리 내부의 테이블에 별칭을 넣어주면 된다.

  ```sql
  INSERT INTO mvc_board(bName, bTitle, bContent, bHit, bGroup, bStep, bIndent)
  VALUES ('abcd', 'is title', 'is content', 0, (select max(bId) from mvc_board board), 0, 0);
  ```

  ​