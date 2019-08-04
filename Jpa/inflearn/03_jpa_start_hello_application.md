# JPA 시작하기

## Hello JPA

* JPA에서는 크게 중요한게 두가지가 있다.

* 첫번째는 객체와 관계형 데이터베이스를 중간에서 매핑하는 과정, 즉 설계 과정이 있고,

* 두번째는 JPA가 어떤 방식으로 동작하는지(영속성 컨텍스트)이며 이 두가지에 대해 이해 하는 것이 중요하다.

### H2

* [http://www.h2database.com](http://www.h2database.com)

* 최고의 실습용 DB, 가볍다(1.5MB)
* 웹용 쿼리툴 제공
* MySQL, Oracle 데이터베이스 시뮬레이션 기능
* Sequence, AUTO INCREMENT 기능 지원

### persistence.xml

* jpa 설정 파일
* /META-INF/persistence.xml 위치
* javax.persistence로 시작 : JPA 표준 속성
* hibernate로 시작 : 하이버 네이트 전용 속성

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
  <!-- 데이터베이스당 1개 작성 -->
  <persistence-unit name="hello">
    <properties>
      <!-- 필수 속성 -->
      <!-- JPA 표준 -->
      <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
      <property name="javax.persistence.jdbc.user" value="sa"/>
      <property name="javax.persistence.jdbc.password" value=""/>
      <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
      <!-- Hibernate 전용 설정 -->
      <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
      <!-- 옵션 -->
      <property name="hibernate.show_sql" value="true"/>
      <property name="hibernate.format_sql" value="true"/>
      <property name="hibernate.use_sql_comments" value="true"/>
      <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
    </properties>
  </persistence-unit>
</persistence>
```

* dialect는 데이터베이스 방언이라는 뜻인데,

### 데이터베이스 방언

* JPA는 특정 데이터베이스에 종속적이지 않은 기술
* 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다르다
  * 가변 문자 : MySQL은 VARCHAR, Oracle은 VARCHAR2
  * 문자열을 자르는 함수 : SQL 표준은 SUBSTRING(), Oracle은 SUBSTR()
  * 페이징 : MySQL은 LIMIT, Oracle은 ROWNUM

* 방언 : SQL 표준을 지키지 않거나 특정 데이터베이스 만의 고유한 기능
* Dialect를 사용하면 각 DB에 맞는 SQL을 생성해 준다.
* hibernate.dialect 속성에 지정
  * H2 : org.hibernate.dialect.H2Dialect
  * Oracle 10g : org.hibernate.dialect.Oracle10gDialect
  * MySQL : org.hibernate.dialect.MySQL5InnoDBDialect
* 하이버네이트는 45가지 방언을 지원

![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/04_jpa_dialect.PNG?raw=true)

## 애플리케이션 개발

### 엔티티 매니저 팩토리 설정

* 엔티티 매니저 팩토리는 애플리케이션 로딩 시점에 딱 하나만 만들어야 한다.
* Persistence를 통해서 xml설정을 읽고, EntityManagerFactory를 만들어야 한다.
* **트랜잭션 단위로 기능을 만들때 항상 엔티티 매니저를 새로 만들어야 한다**. EntityManagerFactory를 통해서 만든다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/05_jpa_entity_manager.PNG?raw=true)

### 객체와 테이블을 생성하고 매핑하기

- @Entity
    - JPA가 관리할 객체, 엔티티라고 한다.
- @Id
    - DB PK와 매핑할 필드

```
@Entity
 public class Member {
 @Id
 private Long id;
 private String name;
//Getter, Setter ...
}
create table Member (
 id bigint not null,
 name varchar(255),
 primary key (id)
);
```

### 비즈니스 로직(CRUD)

* 생성

    ```java
    public class Main {
        public static void main(String[] args) {
            //엔티티 매니저 팩토리 생성
            EntityManagerFactory entityManagerFactory =
                Persistence.createEntityManagerFactory("hello");
            
            //엔티티 매니저 생성
            EntityManager em = entityManagerFactory.createEntityManager();
            
            //트랜잭션 생성, 시작
            EntityTransaction tx = em.getTransaction();
            tx.begin();
    
            try {
                Member member = new Member();
                member.setId(100L);
                member.setName("name");
    
                em.persist(member);
    
                //트랜잭션 커밋
                tx.commit();
    
            } catch (Exception e) {
                tx.rollback();
            } finally {
                em.close();
            }
            
            entityManagerFactory.close();
        }
    }
    ```

    * 실제 실행된 SQL

        ```sql
        Hibernate: 
            /* insert hello.jpa.Member
                */ 
                insert 
                into
                    Member
                    (name, id) 
                values
                    (?, ?)
        ```

* 삭제

    ```java
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    
    try {
        Member findMember = em.find(Member.class, 1L);
        em.remove(findMember);
    
        tx.commit();
    } catch (Exception e) {
        tx.rollback();
    } finally {
        em.close();
    }
    ```

    * 실제 실행된 SQL

        ```java
        Hibernate: 
            select
                member0_.id as id1_0_0_,
                member0_.name as name2_0_0_ 
            from
                Member member0_ 
            where
                member0_.id=?
        Hibernate: 
            /* delete hello.jpa.Member */ 
        delete 
                from
                    Member 
                where
                    id=?
        ```

* 수정

    * em.persist()로 저장해야 되지 않을까? 아니다. 안해도 된다.
    *  JPA 엔티티 매니저를 통해서 객체를 가져오면, 그 객체는 JPA가 관리를 한다.
    * 트랜잭션 커밋 시점에 해당 객체의 변경을 체크하고, 변경 된 것이 있으면
    * 업데이트 쿼리를 만들어서 날리고 커밋이 이루어 진다.

    ```java
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    
    try {
        Member findMember = em.find(Member.class, 1L);
        findMember.setName("NJKIM");
    
        /*
           em.persist()로 저장해야 되지 않을까? 아니다. 안해도 된다.
        */
    
        tx.commit();
    } catch (Exception e) {
        tx.rollback();
    } finally {
        em.close();
    }
    ```

    * 실제 실행된 SQL

        ```sql
        Hibernate: 
            select
                member0_.id as id1_0_0_,
                member0_.name as name2_0_0_ 
            from
                Member member0_ 
            where
                member0_.id=?
        Hibernate: 
            /* update
                hello.jpa.Member */ 
                update
                    Member 
                set
                    name=? 
                where
                    id=?
        ```

- 주의할 점
    - **엔티티 매니저 팩토리**는 하나만 생성해서 애플리케이션 전체에서 공유
    - **엔티티 매니저**는 스레드간에 공유하면 안된다(사용하고 버려야 한다.)
    - JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

## JPQL 소개

* 가장 단순한 조회 방법은
    * EntityManager.find();
    * 객체 그래프 탐색 가능
        * a.getB().getC()
* 그런데, 나이가 18세 이상인 회원을 모두 **검색**하고 싶다면?
* JPA를 사용하면 엔티티 객체를 중심으로 개발하는데, **현업에서 문제는 검색에 대한 쿼리**이다.
* 그래서 JPA는 SQL을 추상화한 JPQL이라는 **객체 지향 쿼리 언어**를 제공한다.
* **검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색** 한다.
* 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.
* 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다 .
    * 실제 RDB에 있는 물리적인 테이블을 대상으로 쿼리를 날려서 검색을 처리하면, RDB에 종속적인 설계가 된다.
    * 그것을 피하기 위해서 **엔티티를 대상으로 쿼리를 날릴 수 있는 JPQL을 제공** 한다.
* **엔티티를 대상으로 쿼리를 할 수 있는 것의 이점**은 애플리케이션의 DB가 변경되는 일이 발생해도 데이터베이스 방언만 변경해주면 된다는 것이다. 애플리케이션에 작성된 JPQL 코드는 수정할 필요가 없다.
    * SQL을 추상화해서 특정 DB SQL에 의존하지 않는다.
* SQL문법과 유사하며 SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN을 지원한다.
* **JPQL은 엔티티 객체를 대상으로 쿼리**하고
* **SQL은 데이터베이스 테이블을 대상으로 쿼리**한다.
* JPQL은 뒤에서 아주 자세히 학습할 예정

### 실습

* **JPQL로 전체 회원 검색**

    * 쿼리 안의 내용을 잘 보면 약간 다르다.
    * JPA 입장에서는 코드를 짤 때, 테이블을 보지 않는다.
    * Member 객체를 대상으로 쿼리를 만든다고 보면 된다.
    * 멤버 **객체** 를 모두 가져오라는 쿼리이다.
    * **JPQL에서 작성하는 쿼리의 대상이 테이블이 아니고 객체(엔티티)** 다.

    ```java
    EntityManager em = emf.createEntityManager();
    
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    
    try {
        List<Member> members = em.createQuery("select m from Member as m", Member.class)
            .getResultList();
    
        tx.commit();
    } catch (Exception e) {
        tx.rollback();
    } finally {
        em.close();
    }
    
    emf.close();
    ```

    * 실제 실행된 SQL

        ```sql
        Hibernate: 
            /* select
                m 
            from
                Member as m */ 
                select
                    member0_.id as id1_0_,
                    member0_.name as name2_0_ 
                from
                    Member member0_
        ```

* **JPQL로 검색 결과 페이징**

    * 객체를 대상으로 쿼리를 하면 이점이 있을까?

    * 있다. 예를 들면, 페이징도 쉽게 가능하다.

        ```java
        EntityManager em = emf.createEntityManager();
        
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        
        try {
            List<Member> members = em.createQuery("select m from Member as m", Member.class)
                .setFirstResult(5)
                .setMaxResults(5)
                .getResultList();
        
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }
        ```

    * 실행된 SQL

    * 여기서 데이터베이스 방언 설정만 Oracle로 바꿔주면, Oracle에 맞는 페이징 쿼리가 나간다.

        ```sql
        Hibernate: 
            /* select
                m 
            from
                Member as m */ 
                select
                    member0_.id as id1_0_,
                    member0_.name as name2_0_ 
                from
                    Member member0_ limit ? offset ?
        ```

### Reference

- [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)