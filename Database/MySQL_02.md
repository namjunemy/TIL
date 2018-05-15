# MySQL 02 - CRUD

> 지식 공유자 - Opentutorials의 egoing님

## CRUD

- Create
  - 생성
- Read
  - 조회

- Update
  - 수정
- Delete
  - 삭제
- Create, Read는 데이터베이스라면 반드시 가지고 있어야 하는 개념이다.(input, output)
- Update, Delete는 분야에 따라 없을 수도 있다. 예를 들면, 역사의 이야기, 회계에서 거래내역을 다루는 경우

## Insert

- 테이블의 컬럼 확인

```mysql
DESCRIBE topic;
DESC topic;
```

- insert

```mysql
INSERT INTO topic (title, description, created, author, profile)
VALUES ('MySQL', 'MySQL is ...', NOW(), 'namjune', 'developer');
```

- 확인

```mysql
SELECT * FROM topic;
```

## Select

* 테이블의 모든 row 조회

```mysql
SELECT * FROM topic;
```

* 일부 column만 조회

```Mysql
SELECT id, title, created, author FROM topic;
```

* 작성자가 namjune인 row 조회

```mysql
SELECT id, title, created, author FROM topic WHERE author = 'namjune';
```

* id를 오름차순으로 정렬해서 조회

```mysql
SELECT
  id,
  title,
  created,
  author
FROM topic
WHERE author = 'namjune'
ORDER BY id DESC;
```

* LIMIT 키워드를 사용해서 조회하는 row의 수 제한

```mysql
SELECT
  id,
  title,
  created,
  author
FROM topic
WHERE author = 'namjune'
ORDER BY id DESC
LIMIT 3;
```

