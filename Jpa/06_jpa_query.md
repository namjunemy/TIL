# JPA 객체지향 쿼리

연관관계 매핑과 영속성 컨텍스트 등 앞의 내용들로 기본기를 다졌고, 지금부터는 활용 단계이다.

> JPA와 객체지향 쿼리
>
> QueryDSL

## JPA는 다양한 쿼리 방법을 지원

* **JPQL**
* JPA Criteria
* **QuertDSL**
* 네이티브 SQL
* JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

## JPQL 소개

* **Java Persistence Query Language**
* 가장 단순한 조회 방법
  * EntityManager.find()
  * 객체 그래프 탐색
    * (a.getB().getC())로 get get get 하면서 계속 찾아다닐 수 없다.
* 나이가 18살 이상인 회원을 모두 검색하고 싶다면?

## JPQL

* JPA를 사용하면 엔티티 객체를 중심으로 개발

* 문제는 검색 쿼리

* 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색을 해야한다.

  * DB를 몰라야 한다. 자바 코드에서 멤버 테이블이 있구나 보다, 멤버 객체가 있구나라고 생각을 가지고 개발해야 한다.

* 근데, 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.

* **애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.**

* 그래서 JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다.

* SQL과 문법이 유사하고, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN등을 지원한다.

* **JPQL은 엔티티 객체를 대상으로 쿼리**를 질의하고

* **SQL은 데이터베이스 테이블을 대상으로 쿼리**를 질의한다.

* ex)

  ```java
  //검색
  String jpql= "select m From Member m where m.name like '%hello%'";
  
  List<Member> result = em.createQuery(jpql, Member.class).getResultList();
  ```

* 위의 JPQL에 의해 변환되어(데이터베이스 방언을 참조해서 DB에 맞는 쿼리로) 실행된 SQL

  ```sql
  select
      m.id as id,
      m.age as age,
      m.USERNAME as USERNAME,
      m.TEAM_ID as TEAM_ID
  from
      Member m
  where
      m.age>18
  ```

* 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리라고 이해하면 되며,
* SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
* JPQL을 한마디로 정의하면 객체 지향 SQL 이다.

## JPQL 문법

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

* 몇가지 유의 사항은 존재 한다.

  * from절에 들어가는 것은 객체다!

    `select m from Member m where m.age > 8`

  * 엔티티와 속성은 대소문자를 구분
    
    * 예를 들면, Member 엔티티와 username 필드
  * JPQL 키워드는 대소문자 구분 안함
    
    * SELECT, FROM, where
  * 엔티티 이름을 사용한다. 테이블 이름이 아니다
    
    * 엔티티명 Member
  * 별칭은 필수이다.
    
    * Member의 별칭 m

### 결과 조회 API

* `query.getResultList()`
  * 결과가 하나 이상인 경우, 리스트를 반환한다.
* `query.getSingleResult()`
  * 결과가 정확히 하나, 단일 객체를 반환한다.(정확히 하나가 아니면 예외 발생)

### 파라미터 바인딩 

* 웬만하면 이름으로 바인딩하자.

* 이름 기준

  ```sql
  SELECT m 
  FROM Member m
  where m.username=:username
  ```

  ```java
  query.setParameter("username", usernameParam);
  ```

* 위치 기준

  ```sql
  SELECT m
  FROM Member m
  where m.username=?1
  ```

  ```java
  query.setParameter(1, usernameParam);
  ```

### 프로젝션

* 엔티티 프로젝션(멤버 조회)
  * `SELECT m FROM Member m ...`
* 엔티티 프로젝션(멤버 안에 있는 팀 조회)
  * `SELECT m.team FROM Member m ...`
* 단순 값 프로젝션
  * hibernate에서 지원을 해서 username, age로 쓸 수 있지만
  * 공식적으로는 m.username, m.age로 접근해야 한다.
  * `SELECT m.username, m.age FROM Member m ...`
* **new** 명령어
  * **단순 값을 DTO로 바로 조회** 한다.
  * new 패키지명, DTO를 넣고 생성자처럼 사용해서 DTO로 바로 반환 받을 수 있다.
  * `SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m ...`
* DISTINCT는 중복을 제거 한다.

### 페이징 API

* JPA는 페이징을 다음 두 API로 추상화 해준다.

* 조회 시작위치(0부터 시작)

  * `setFirstResult(int startPosition)`

* 조회할 데이터 수

  * `setMaxResults(int maxResult)`

* 페이징 쿼리 예시)

  ```ajva
  String jpql = "select m from Member m order by m.name desc";
  
  List<Member> resultList = em.createQuery(jpql, Member.class)
    .setFirstResult(10)
    .setMaxResult(20)
    .getResultList();
  ```

  * 모든 데이터 베이스의 방언이 동작한다.

  * 위 쿼리의 MySQL 방언

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

  * 위 쿼리의 Oracle 방언

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

### 집합과 정렬

* 기본적인 집합 명령어 다 동작 한다.

```sql
select
    COUNT(m),   //회원수
    SUM(m.age), //나이 합
    AVG(m.age), //평균 나이
    MAX(m.age), //최대 나이
    MIN(m.age)  //회소 나이
from Member m
```

* GROUP BY, HAVING
* ORDER BY

### 조인

* 내부 조인

  * 멤버 내부의 팀에 m.team으로 접근

  ```sql
  SELECT m
  FROM Member m
  [INNER] JOIN m.team t
  ```

* 외부 조인

  ```sql
  SELECT m
  FROM MEMBER m
  LEFT [OUTER] JOIN m.team t
  ```

* 세타 조인

  * 일명 막(?) 조인이다. 연관관계 상관 없이 유저명과 팀의이름이 같은 경우 찾아라 라는 쿼리 날릴 수 있다. 이런 조인을 세타 조인이라고 한다.

  ```java
  SELECT COUNT(m)
  FROM Member m, Team t
  WHERE m.username = t.name
  ```

* 참고:

  * **하이버네이트 5.1부터 세타 조인도 외부 조인이 가능!**

### 페치 조인

* 현업에서 굉장히 많이 쓰인다. fetchType을 LAZY로 다 세팅 해놓고, 쿼리 튜닝할때 한꺼 번에 조회가 필요한 경우 페치 조인을 사용한다.

* 엔티티 객체 그래프를 한번에 조회하는 방법이다.

* 별칭을 사용할 수 없다.

* JPQL

  * 멤버를 조회할 때, 팀까지 같이 조회한다.

  ```select m from Member m join fetch m.team
  select m from Member m join fetch m.team
  ```

* SQL

  ```sql
  SELECT M.*, T.*
  FROM MEMBER T
  INNER JOIN TEAM T ON M.TEAM_ID = T.ID
  ```

* 최근에(jpa2.1)는 페치 조인 말고 엔티티 그래프라는 기능이 있다. 공부해보자.

* 페치 조인 예시)

  ```java
  String jpql = "select m from Member m join fetch m.team";
  
  List<Member> members = em.createQuery(jpql, Member.class).getResultList();
  
  for (Member member : members) {
      //페치 조인으로 회원과 팀을 함께 조회해서 지연 로딩이 발생하지 않는다.
      System.out.println("username = " + member.getUsername() + ","
                      + "teamname = " + member.getTeam().name());
  }
  ```

* 현업에서 많이 쓰이는 이유는, 리스트 쭉 뿌릴때. LAZY로 가게 되면 리스트에서 반복문으로 정보 받아올 때마다 DB에 쿼리가 나간다. 이게 JPA N+1 문제이다. 성능상 좋지 않다.
  * 리스트가 10명이다. 10명의 리스트를 가져오는 쿼리 한방 나가는데, 세부 조회를 할 때마다(10번) Lazy 로딩 되므로 쿼리가 총 11번 나가게 된다.
  * [JPA N+1 문제와 해결방안](<https://jojoldu.tistory.com/165>)

### 기타

* 서브 쿼리 지원
* EXISTS, IN
* BETWEEN, LIKE, IS NULL

### JPQL 기본 함수

* CONCAT
* SUBSTRING
* TRIM
* LOWER, UPPER
* LENGTH
* LOCATE
* ABS, SQRT, MOD
* SIZE, INDEX(JPA 용도)

### CASE 식

* 기본 CASE 식

  ```sql
  select
      case when m.age <= 10 then '학생요금'
           when m.age >= 60 then '경로요금'
           else '일반요금'
      end
  from Member m
  ```

* 단순 CASE 식

  ```sql
  select
      case t.name
        when '팀A' then '인센티브110%'
        when '팀B' then '인센티브120%'
        else '인센티브105%'
      end
  from Team t
  ```

### 사용자 정의 함수 호출

* 하이버네이트는 사용전 방언에 추가해야 한다.

* DB에 커스텀 펑션 만든 후 사용

  ```java
  select function ('group_concat', i.name) from Item i
  ```

### Named 쿼리 - 정적 쿼리

* **미리 정의**해서 이름을 부여해두고 사용하는 JPQL

* 어노테이션, XML에 정의

* 애플리케이션 로딩 시점에 초기화 후 재사용

* 애플리케이션 로딩 시점에 쿼리를 검증

* 어노테이션 예시)

  ```java
  @Entity
  @NamedQuery(
      name = "Member.findByUsername",
      query = "select m from Member m where m.username = :username")
  public vlass Member {
    ...
  }
  ```

  ```java
  List<Member> resultList = 
      em.createNamedQuery("Member.findByUsername", Member.class)
          .setParameter("username", "회원1")
          .getResultList();
  ```

* 왜 굳이 엔티티에 네임드 쿼리 쓰나? 그냥 찾을 때 쓰는거랑 똑같지 않을까?

  * em.createQuery()안에 넣어서 그냥 사용할 수 있다. 하지만 큰 위험이 따른다.
  * 문자열 자체이기 때문에 에러 포인트가 많다. 근데, 사용자 요청 오기 전까지 쿼리가 실행되지 않으니까 모른다. 그제서야 안다.
  * 근데 Named쿼리는 애플리케이션 로딩 시점에 쿼리를 파싱한다. 그래서 배포하기 전에 문제를 잡을 수 있다. 아니, 개발하면서 띄워 보기만해도 잡을 수 있다.
  * Spring-Data-JPA를 사용할 때 @Query 어노테이션이 이 Named쿼리로 동작하게 된다. 그래서 스프링 애플리케이션 올라갈때 바로 에러를 잡을 수 있다.
  * 런타임 에러는 정말 위험하다.

* XML에도 정의를 할 수 있다. 필요하면 알아보자.

* Named 쿼리 환경에 따른 설정

  * XML이 항상 우선권을 가진다.
  * 애플리케이션 운영 환경에 따라 다른 XML을 배포할 수 있다.