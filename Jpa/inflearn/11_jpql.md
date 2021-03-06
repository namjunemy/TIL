# JPA 객체지향 쿼리

연관관계 매핑과 영속성 컨텍스트 등 앞의 내용들로 기본기를 다졌고, 지금부터는 활용 단계이다.

> JPA와 객체지향 쿼리
>
> QueryDSL

## JPA는 다양한 쿼리 방법을 지원

- **JPQL**
- JPA Criteria
- **QuertDSL**
- 네이티브 SQL
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

## JPQL 소개

- **Java Persistence Query Language**
- 가장 단순한 조회 방법
  - EntityManager.find()
  - 객체 그래프 탐색
    - (a.getB().getC())로 get get get 하면서 계속 찾아다닐 수 없다.
- 나이가 18살 이상인 회원을 모두 검색하고 싶다면?

## JPQL

- JPA를 사용하면 엔티티 객체를 중심으로 개발

- 문제는 검색 쿼리

- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색을 해야한다.

  - DB를 몰라야 한다. 자바 코드에서 멤버 테이블이 있구나 보다, 멤버 객체가 있구나라고 생각을 가지고 개발해야 한다.

- 근데, 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.

- **애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.**

- 그래서 JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다.

- SQL과 문법이 유사하고, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN등을 지원한다.

- **JPQL은 엔티티 객체를 대상으로 쿼리**를 질의하고

- **SQL은 데이터베이스 테이블을 대상으로 쿼리**를 질의한다.

- ex)

  ```java
  //검색
  String jpql= "select m From Member m where m.name like '%hello%'";
  
  List<Member> result = em.createQuery(jpql, Member.class).getResultList();
  ```

- 위의 JPQL에 의해 변환되어(데이터베이스 방언을 참조해서 DB에 맞는 쿼리로) 실행된 SQL

  ```sql
  Hibernate: 
      /* select
          m 
      from
          Member m 
      where
          m.username like '%hello%' */ select
              member0_.id as id1_7_,
              member0_.createdBy as createdB2_7_,
              member0_.createdDate as createdD3_7_,
              member0_.lastModifiedBy as lastModi4_7_,
              member0_.lastModifiedDate as lastModi5_7_,
              member0_.age as age6_7_,
              member0_.description as descript7_7_,
              member0_.city as city8_7_,
              member0_.street as street9_7_,
              member0_.zipcode as zipcode10_7_,
              member0_.locker_id as locker_15_7_,
              member0_.roleType as roleTyp11_7_,
              member0_.team_id as team_id16_7_,
              member0_.name as name12_7_,
              member0_.endDate as endDate13_7_,
              member0_.startDate as startDa14_7_ 
          from
              Member member0_ 
          where
              member0_.name like '%hello%'
  ```

- 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리라고 이해하면 되며,

- SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.

- JPQL을 한마디로 정의하면 객체 지향 SQL 이다.

## Criteria 소개

* JPA 표준 스팩에 들어가 있는 기술이다.
* 동적 쿼리를 자바 코드로 짤 수 있게 해준다.
* 부분 동적 쿼리를 떼어 내는 것이 편리 하다.
* 컴파일 타임에 오류를 발견할 수 있다.
* 단점은 SQL 스럽지 않다. 그래서, 실무에서 사용하지 말자, 유지보수 어렵고 복잡하다.
* 대신에 **QueryDSL(오픈소스 라이브러리) 사용을 권장**한다.

```java
//Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

//루트 클래스 (조회를 시작할 클래스)

Root<Member> m = query.from(Member.class);

//쿼리 생성
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.createQuery(cq).getResultList();

tx.commit();
```

## QueryDSL 소개

* 문자가 아닌 자바코드로 JPQL을 작성할 수 있다.
* JPQL 빌더 역할을 한다.
* 컴파일 시점에 문법의 오류를 찾을 수 있고, 동적 쿼리 작성이 편리하다.
* 단순, 쉬움. **실무 사용 권장**
* JPQL을 문법을 알면 QueryDSL은 쓰기 쉽다. 기승전 JPQL

```java
//JPQL
//select m from Member m where m.age > 18
    
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;

List<Member> list = 
    query.selectFrom(m)
         .where(m.age.gt(18))
         .orderBy(m.name.desc())
         .fetch();
```

## 네이티브 SQL 소개

* JPA가 제공하는 SQL을 직접 사용하는 기능
* JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
* 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트
* 하이버네이트가 데이터베이스 방언에 설정해서 사용할 수 있도록 지원한다.

```sql
em.createNativeQuery("SELECT ID, TEAM_ID, NAME FROM MEMBER WHERE NAME = `KIM`", Member.class)
  .getResultList();
```

## JDBC 직접사용, SpringJdbcTemplate 등

* JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스등을 함꼐 사용할 수 있다.
* 단, 영속성 컨텍스트를 적절한 시점에 강제로 플러시 하는 것이 필요하다.
  * DB 커넥션을 직접 가져와서 쿼리를 날릴때, em.persist() 해줘봤자. 영속성 컨텍스트에만 담겨있고 DB에는 반영되어있지 않다. 그래서 커넥션에서 날라가는 select 쿼리는 정상적이지 않다. 그 이후에 commit시점에 플러시가 동작하게 된다.
* 예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시

## JPQL 기본 문법과 기능

*  Java Persistence Query Language

* JPQL은 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리하는 것이 아니라 **엔티티 객체를 대상으로 쿼리** 한다.

  * **JPQL은 엔티티 객체를 대상으로 쿼리** 를 질의하고
  * **SQL은 데이터베이스 테이블을 대상으로 쿼리** 를 질의한다.

* JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.

* JPQL은 결국 SQL로 변환된다.

* 학습에 사용할 예제 객체 모델과 DB 모델

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/39_jpql.PNG?raw=true)

### JPQL 문법

```sql
select_문 :: = 
    select_절
    from_절
    [where_절]
    [groupby_절]
    [having_절]
    [orderby_절]
```

```sql
update_문 :: = update_절 [where_절]
delete_문 :: = delete_절 [where_절]
```

- 몇가지 유의 사항은 존재 한다.

  - from절에 들어가는 것은 객체다!

    `select m from Member m where m.age > 8`

  - 엔티티와 속성은 대소문자를 구분

    - 예를 들면, Member 엔티티와 username 필드

  - JPQL 키워드는 대소문자 구분 안함

    - SELECT, FROM, where

  - 엔티티 이름을 사용한다. 테이블 이름이 아니다

    - 엔티티명 Member

  - 별칭은 필수이다.

    - Member의 별칭 m

### 집합과 정렬

- 기본적인 집합 명령어 다 동작 한다.

  ```sql
  select
      COUNT(m),   //회원수
      SUM(m.age), //나이 합
      AVG(m.age), //평균 나이
      MAX(m.age), //최대 나이
      MIN(m.age)  //회소 나이
  from Member m
  ```

- GROUP BY, HAVING
- ORDER BY

### TypeQuery, Query

* TypeQuery

  * 반환 타입이 명확할 때 사용한다.

    ```sql
    TypedQuery<Member> query = 
      em.createQuery("SELECT m FROM Member m", Member.class);
    ```

* Query

  * 반환 타입이 명확하지 않을 때 사용

    ```sql
    Query query = 
      em.createQuery("SELECT m.username, m.age FROM Member m");
    ```

### 결과 조회 API

- `query.getResultList()`
  - 결과가 하나 이상인 경우, 리스트를 반환한다.
- `query.getSingleResult()`
  - 결과가 정확히 하나, 단일 객체를 반환한다.(정확히 하나가 아니면 예외 발생)

### 파라미터 바인딩 

- 웬만하면 이름으로 바인딩하자.

- 이름 기준

  ```java
  TypedQuery<Member> query = em.createQuery("SELECT m 
                                            FROM Member m
                                            where m.username=:username:, Member.class);
  query.setParameter("username", "member1");
  Member member = query.getSingleResult();
  ```

  * 보통은 위의 과정을 메서드 체이닝으로 처리함.

    ```java
    Member member = 
      em.createQuery("SELECT m FROM Member m where m.username=:username:, Member.class)
                   .query.setParameter("username", "member1")
                   .getSingleResult();
    ```

- 위치 기준

  ```sql
  SELECT m
  FROM Member m
  where m.username=?1
  ```

  ```java
  query.setParameter(1, usernameParam);
  ```

## 프로젝션

* 프로젝션은 SELECT 절에 조회할 대상을 지정하는 것을 말한다.

* 프로젝션의 대상은 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입)

  * RDB 쿼리에서는 SELECT 절에 스칼라 타입만 선택할 수 있지만, JPQL에서는 다양하게 가능하다.

* SELECT **m** FROM Member m

  * 엔티티 프로젝션

    ```java
    // 엔티티 프로젝션의 대상인 Member 리스트는 영속성 컨텍스트에서 관리 된다.
    List<Member> members =
      em.createQuery("select m from Member m", Member.class)
      .getResultList();
    
    //update 쿼리 나간다.
    Member findMember = members.get(0);
    findMember.setAge(27);
    ```

* SELECT **m.team** FROM MEMBER m

  * 엔티티 프로젝션
  * 이 경우에는 멤버와 팀의 조인 쿼리가 나간다. 뒤에서 배우겠지만, 이럴경우 JPQL에 조인을 명시하는 것이 좋다.
  * SELECT t FROM Member m JOIN m.team t

* SELECT **m.address** FROM Member m

  * 임베디드 타입 프로젝션

  * 임베디드 타입 자체로 엔티티 프로젝션처럼 사용할 수는 없다. 엔티티에 속해 있기 때문에. 엔티티에서 시작해야 한다.

    ```java
    em.createQuery("select o.address from Order o", Address.class)
      .getResultList();
    ```

* SELECT **m.username, m.age** FROM Member m

  * 스칼라 타입 프로젝션

    ```java
    em.createQuery("select m.username, m.age from Member m")
      .getResultList();
    ```

* **DISTINCT** 로 중복 제거 가능

  ```java
  em.createQuery("select distinct m.username, m.age from Member m")
    .getResultList();
  ```

### 프로젝션 - 여러 값 조회

* SELECT m.username, m.age FROM Member m

  * 스칼라 타입이 string, int인데 어떻게 가져오지?

* Query 타입으로 조회

  * 반환 타입이 명확하지 않을 때 사용

  * 타입이 명확하지 않아서 Object[] 타입으로 넣어준다. 꺼내서 써야 한다...

    ```java
    List resultList = em.createQuery("select m.username, m.age from Member m")
      .getResultList();
    
    Object o = resultList.get(0);
    Object[] result = (Object[]) o;
    log.info(result[0]);
    log.info(result[1]);
    ```

* Object[] 으로 조회

  * 한단계가 사라졌지만 그닥..

    ```java
    List<Object[]> resultList = em.createQuery("select m.username, m.age from Member m")
      .getResultList();
    
    Object[] result = resultList.get(0);
    log.info(result[0]);
    log.info(result[1]);
    ```

* **new 명령어로 조회**

  * 가장 깔끔한 방법

  * 단순 값을 DTO로 바로 조회

  * SELECT **new** UserDTO(m.username, m,age) FROM Member m

  * 순서와 타입이 일치하는 생성자가 필요하고,

  * DTO의 패키지명을 다 적어줘야 하는데, 실제로는 QueryDSL로 극복할 수 있다.

    * `Projections.bean(UserDTO.class, user.firstName, user.lastName));`

    ```java
    // 스칼라 타입 다수를 DTO로 바로 주입 받는 방법
    List<MemberDTO> members =
      em.createQuery("select new jpql.dto.MemberDTO(m.name, m.age) from Member m", MemberDTO.class)
      .getResultList();
    System.out.println(members.get(0).getName());
    System.out.println(members.get(0).getAge());
    ```

## 페이징 API

- JPA는 페이징을 다음 두 API로 추상화 해준다.

- 조회 시작위치(0부터 시작)

  ```java
  setFirstResult(int startPosition)
  ```

- 조회할 데이터 수

  ```java
  setMaxResults(int maxResult)
  ```

- 페이징 쿼리 예시([커밋로그](https://github.com/namjunemy/orm-jpa-basic/commit/150fe2e9e74cce1e4249a3d8ab4063cec8f1f166))

  ```java
  String jpql = "select m from Member m order by m.age desc";
  
  List<Member> resultList = em.createQuery(jpql, Member.class)
    .setFirstResult(1)
    .setMaxResults(20)
    .getResultList();
  ```

  - 모든 데이터 베이스의 방언이 동작한다.

  - 위 쿼리의 MySQL 방언

    ```SQL
    SELECT
        M.ID AS ID,
        M.AGE AS AGE,
        M.TEAM_ID AS TEAM_ID,
        M.NAME AS NAME
    FROM
        MEMBER M
    ORDER BY
        M.name DESC LIMIT ?, ?
    ```

  - 위 쿼리의 Oracle 방언

    ```sql
    SELECT * FROM
        ( SELECT ROW_.*, ROWNUM ROWNUM_
        FROM
            ( SELECT
                M.ID AS ID,
                M.AGE AS AGE,
                M.TEAM_ID AS TEAM_ID,
                M.NAME AS NAME
              FROM MEMBER M
              ORDER BY M.NAME
             ) ROW_
         WHERE ROWNUM <= ?
         )
    WHERE ROWNUM_ > ?
    ```

* Spring Data JPA를 쓰다보면 생각보다 페이징이 쉽게 처리되는데, 내부적으로 JPA가 이런 방식으로 다 지원해 준다.

## 조인

- 엔티티를 중심으로 동작한다. 객체 스타일로 쿼리가 작성된다.

- 내부 조인

  - 멤버 내부의 팀에 m.team으로 접근

  ```sql
  SELECT m
  FROM Member m
  [INNER] JOIN m.team t
  ```

- 외부 조인

  ```sql
  SELECT m
  FROM MEMBER m
  LEFT [OUTER] JOIN m.team t
  ```

- 세타 조인

  - 일명 막(?) 조인이다. 연관관계 상관 없이 유저명과 팀의이름이 같은 경우 찾아라 라는 쿼리 날릴 수 있다. 이런 조인을 세타 조인이라고 한다.

  ```java
  SELECT COUNT(m)
  FROM Member m, Team t
  WHERE m.username = t.name
  ```

### 조인 - ON 절

- JPA2.1 부터 ON절을 활용한 조인이 가능하다.

  - 조인시 조인 대상을 미리 필터링 할 수 있음
  - 하이버네이트 5.1부터 **연관관계가 없는 엔티티도 외부 조인**이 가능!

- **1. 조인 대상 필터링**

  - 예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인

  - JPQL

    ```sql
    SELECT m,t 
    FROM Member m 
    LEFT JOIN m.team t
      on t.name = 'A'
    ```

  - SQL

    ```sql
    SELECT m.*, t.*
    FROM Member m
    LEFT JOIN TEAM t
      ON m.TEAM_ID = t.id AND t.name = 'A'
    ```

- **2. 연관관계 없는 엔티티 외부 조인**

  - 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인. 연관관계가 없다.

  - 서로 아무 연관관계가 없어도 LEFT JOIN 가능하다.

  - JPQL

    ```sql
    SELECT m,t 
    FROM Member m 
    LEFT JOIN Team t
      on m.username = t.name
    ```

  - SQL

    ```sql
    SELECT m.*, t.*
    FROM Member m
    LEFT JOIN TEAM t
      ON m.username = t.name
    ```

## 서브쿼리

* SQL에서 말하는 서브쿼리의 개념과 동일하다

  * 나이가 평균보다 많은 회원

    * 메인쿼리와 서브쿼리 사이에 연관관계가 없다. 각각 m, m2로 분리되어 있음.

    ```sql
    select m 
    from Member m
    where m.age > (select avg(m2.age) from Member m2)
    ```

  * 한 건이라도 주문한 고객 

    ```sql
    select m
    from Member m
    where (select count(o) from Order o where m = o.member) > 0
    ```

### 서브 쿼리 지원 함수

* [NOT] EXISTS (subquery) : 서브쿼리에 결과가 존재하면 참이다.

* {ALL | ANY | SOME} (subquery)

  * ALL : 모두 만족하면 참
  * ANY, SOME : 같은 의미, 조건을 하나라도 만족하면 참

* [NOT] IN (subquery) : 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

* 예제)

  * 팀A 소속인 회원

    ```sql
    select m
    from Member m
    where exists (select t from m.team t where t.name = '팀A')
    ```

  * 전체 상품 각각의 재고보다 주문량이 많은 주문들

    ```sql
    select o
    from Order o
    where o.orderAmount > ALL (select p.stockAmount from product p)
    ```

  * 어떤 팀이든 팀에 소속된 회원

    ```sql
    select m
    from Member m
    where m.team = ANY (select t from Team t)
    ```

### JPA 서브 쿼리의 한계

* JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용이 가능하다.
* 하이버네이트에서는 SELECT절에서도 서브쿼리가 가능하다.
* **FROM 절의 서브 쿼리는 현재 JPQL에서 불가능하다.**
  * **조인으로 풀 수 있으면 풀어서 해결한다.**
  * 조인으로 해결하지 못하면
    * 네이티브 쿼리로 해결하거나,
    * 어플리케이션 비즈니스 로직에서 풀어내거나
    * 쿼리를 두번 날려서 해결할 수 있다.

## JPQL 타입 표현

* 문자

    * 'HELLO', 'She''s'

* 숫자

    * 10L(Long), 10D(Double), 10F(Float)

* Boolean

    * TRUE, FALSE

* ENUM

    * Jpabook.MemberType.Admin(풀 패키지명 포함)
    * 보통은 파라미터 바인딩으로 처리
    * QueryDSL에서는 복잡하지 않게 자바 코드로 사용한다.

* 엔티티 타입

    * TYPE(m) = Member (상속 관계에서 사용)

    * Item을 상속받은 여러개의 클래스 중 Book만 조회하고 싶은 경우

        ```java
        em.createQuery("select i from Item i where type(i) = Book", Item.class)
            .getResultList();
        ```

* 코드

    ```java
    // JPQL 타입 표현 : 문자열, boolean, ENUM
    String query = "select m.name, 'HELLO', TRUE From Member m " +
        "where m.memberType = :memberType";
    
    List<Object[]> resultList = em.createQuery(query)
        .setParameter("memberType", MemberType.ADMIN)
        .getResultList();
    
    for(Object[] objects : resultList) {
        System.out.println(objects[0]);
        System.out.println(objects[1]);
        System.out.println(objects[2]);
    }
    ```

## JPQL 기타

* SQL과 문법이 같은  식
* EXISTS, IN
* AND, OR, NOT
* =, >, >=, <, <=, <>
* BETWEEN, LIKE, IS NULL

## CASE 식

- 기본 CASE 식

    ```sql
    select
        case when m.age <= 10 then '학생요금'
             when m.age >= 60 then '경로요금'
             else '일반요금'
        end 
    from Member m
    ```

    ```java
    String query =
                    "select " +
                        "case when m.age <= 10 then '학생요금'" +
                        "     when m.age >= 60 then '경로요금'" +
                        "     else '일반요금'" +
                        "end " +
                    "from Member m";
    List<String> result = em.createQuery(query, String.class)
                    .getResultList();
    ```

    ```java
    Hibernate: 
        /* select
            case 
                when m.age <= 10 then '학생요금'     
                when m.age >= 60 then '경로요금'     
                else '일반요금'
            end 
        from
            Member m */ select
                case 
                    when member0_.age<=10 then '학생요금' 
                    when member0_.age>=60 then '경로요금' 
                    else '일반요금' 
                end as col_0_0_ 
            from
                Member member0_
    ```

- 단순 CASE 식

    ```sql
    select
        case t.name
          when '팀A' then '인센티브110%'
          when '팀B' then '인센티브120%'
          else '인센티브105%'
        end
    from Team t
    ```

* COALSESCE 

    * 하씩 조회해서 null이 아니면 반환

        ```sql
        String query =
            "select coalesce(m.name, '이름 없는 회원') from Member m";
        ```

* NULLIF

    * 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

    * 사용자 이름이 '관리자'면 null을 반환하고 나머지는 본인의 이름을 반환함.

    * 특정 이름을 숨겨야 할 경우

        ```java
        String query =
            "select nullif(m.name, '관리자') from Member m";
        ```

## JPQL 기본 함수

* JPQL이 제공하는 표준 함수

- CONCAT

- SUBSTRING

- TRIM

- LOWER, UPPER

- LENGTH

- LOCATE -> indexOf와 같이 자리수 반환

- ABS, SQRT, MOD

- SIZE, INDEX(JPA 용도)

    * 값 타입 컬렉션에 컬렉션의 위치 값을 구할 때 @OrderColumn 사용. INDEX로 위치 참조 가능

    ```java
    String query = "select size(t.members) from Team t";
    String query = "select index(t.members) from Team t";
    ```

## 사용자 정의 함수 호출

- 하이버네이트는 사용전 방언에 추가해야 한다.

- DB에 커스텀 펑션 만든 후 사용

    ```java
    select function ('group_concat', i.name) from Item i
    select 'group_concat'(i.name) from Item i //하이버네이트 사용시 function처럼 사용 가능
    ```

* [커밋로그](https://github.com/namjunemy/orm-jpa-basic/commit/f606aa3bd8fa0be0c9b79b47ec9ffb3541731061)

### Reference

- [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)