# MySQL 03

> 지식 공유자 - Opentutorials의 egoing님

## 관계형 데이터베이스의 필요성

```Text
+----+------------+-------------------+---------------------+---------+-----------+
| id | title      | description       | created             | author  | profile   |
+----+------------+-------------------+---------------------+---------+-----------+
|  1 | MySQL      | MySQL is ...      | 2018-05-16 01:12:41 | namjune | developer |
|  2 | Oracle     | Oracle is ...     | 2018-05-16 01:18:46 | admin   | admin     |
|  3 | SQL Server | SQL Server is ... | 2018-05-16 01:19:54 | namjune | developer |
|  4 | PostgreSQL | PostgreSQL is ... | 2018-05-16 01:19:54 | namjune | developer |
+----+------------+-------------------+---------------------+---------+-----------+
```

* 위의 표에서 보면 author와 profile 즉, 작성자에 대한 정보가 중복되는 현상을 볼 수 있다.
* 데이터가 중복되고 있다면, 개선할 것이 있다는 강력한 증거이다.
* 위의 데이터가 1억개의 row를 가지고 있고, 2천만개 쯤 작성자에 대한 정보가 중복되고 있다고 가정하면 여러가지 문제점을 가져올 수 있다.
* 2천만개 쯤 되는 복잡한 그리고 중복되는 데이터를 수정한다고 했을 때, 기술적으로 경제적으로 엄청난 손실이다.
* 위의 문제를 해결하기 위해서 author라는 작성자에 대한 정보를 저장하는 테이블을 새로 만들고 topic 테이블에는 해당 author 테이블의 식별자인 author_id를 저장하게 되면 중복을 해결 할 수 있다. 또한, 작성자 정보를 수정해야 할 경우 author 테이블의 작성자 정보만 수정하면, 테이블을 분리하기 전 topic 테이블의 여러 row를 모두 수정해주는 작업을 한 효과를 얻을 수 있다.
* 하지만, 여기에도 trade off가 존재한다. topic 테이블을 조회했을 때, 예전처럼 작성자의 정보를 한 눈에 식별할 수 없게 된다.(이는 join으로 해결 할 수 있다.)

## 테이블 분리하기

* 기존 테이블 RENAME

```mysql
RENAME TABLE topic TO topic_backup;
```

* author 테이블 생성

```mysql
CREATE TABLE author (
  id      int(11)     NOT NULL AUTO_INCREMENT,
  name    varchar(20) NOT NULL,
  profile varchar(200)         DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

```text
+----+---------+----------------+
| id | name    | profile        |
+----+---------+----------------+
|  1 | namjune | developer      |
|  2 | admin   | DBA            |
|  3 | user1   | data scientist |
+----+---------+----------------+
```

* 새로운 topic 테이블 생성

```mysql
CREATE TABLE topic (
  id          int(11)     NOT NULL AUTO_INCREMENT,
  title       varchar(30) NOT NULL,
  description text,
  created     datetime    NOT NULL,
  author_id   int(11)              DEFAULT NULL,
  PRIMARY KEY (id)
);
```

```text
+----+------------+-------------------+---------------------+-----------+
| id | title      | description       | created             | author_id |
+----+------------+-------------------+---------------------+-----------+
|  1 | MySQL      | MySQL is...       | 2018-05-16 02:18:16 |         1 |
|  2 | Oracle     | Oracle is ...     | 2018-05-16 02:18:17 |         1 |
|  3 | SQL Server | SQL Server is ... | 2018-05-16 02:18:19 |         2 |
|  4 | PostgreSQL | PostgreSQL is ... | 2018-05-16 02:18:20 |         3 |
|  5 | MongoDB    | MongoDB is ...    | 2018-05-16 02:18:21 |         1 |
+----+------------+-------------------+---------------------+-----------+
```



