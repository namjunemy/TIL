# MySQL CURRENT_TIMESTAMP

> **아래와 같은 차이점이 있었지만, 패치로 인해서 현 시점에는 무리 없이 now()와 CURRENT_TIMESTAMP, CURRENT_TIMESTAMP()가 동일하게 동작한다. 따라서, 현재 시간을 기본 값으로 설정하고 싶을 때 선택해서 사용하면 된다.** 

* ~~Oracle에서 날짜와 시간을 저장하는 DATE 타입에 현재 시간을 기본 값으로 설정하고 싶을 때, 아래와 같이 SYSDATE를 사용한다.~~

  ```sql
  CREATE TABLE ORACLE_TABLE (
    DATE_CREATED DATE DEFAULT SYSDATE
  )
  ```

* ~~MySQL에서는 Oracle의 DATE 타입에 대응되는 DATETIME이 있다. 여기에 현재 시간을 기본 값으로 설정하려면 now() 함수를 사용하면 안된다.~~

  * ~~MySQL v5.6.5 이상의 경우 CURRENT_TIMESTAMP를 사용한다.~~

  ```sql
  CREATE TABLE MYSQL_TABLE( 
    DATE_CREATED DATETIME DEFAULT CURRENT_TIMESTAMP 
  )
  ```

  * ~~MySQL v5.6.5 미만일 경우 해당 테이블에 트리거를 작성해야 한다.~~

  ```Sql
  CREATE TABLE MYSQL_TABLE( 
    DATE_CREATED DATETIME 
  )
  
  CREATE 
    TRIGGER MYSQL_TABLE_OnInsert BEFORE INSERT ON MYSQL_TABLE 
    FOR EACH ROW 
    SET NEW.DATE_CREATED = NOW() ;
  
  ```

> 출처: http://jsonobject.tistory.com/122 [지단로보트의 블로그]

