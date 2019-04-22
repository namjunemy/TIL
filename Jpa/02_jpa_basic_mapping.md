# JPA 기초와 매핑

1. Hello JPA

JPA에서는 크게 중요한게 두가지가 있다. 첫번째는 객체와 관계형 데이터베이스를 중간에서 매핑하는 과정, 즉 설계 과정이 있고, 두번째는 JPA가 어떤 방식으로 동작하는지(영속성 컨텍스트)이며 이 두가지에 대해 이해 하는 것이 중요하다.

## H2

* [http://www.h2database.com](http://www.h2database.com)

* 최고의 실습용 DB, 가볍다(1.5MB)
* 웹용 쿼리툴 제공
* MySQL, Oracle 데이터베이스 시뮬레이션 기능
* Sequence, AUTO INCREMENT 기능 지원

## 객체 매핑하기

* @Entity
  * JPA가 관리할 객체, 엔티티라고 한다.
* @Id
  * DB PK와 매핑할 필드

## persistence.xml

* jpa 설정 파일
* /META-INF/persistence.xml 위치
* javax.persistence로 시작 : JPA 표준 속성
* hibername로 시작 : 하이버 네이트 전용 속성

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.2">
  <persistence-unit name="hello">
    <properties>
      
      <!-- 필수 속성-->
      <property name="javax.persistence.jdbc.driver" value="org.h2.driver"/>
      <property name="javax.persistence.jdbc.user" value="sa"/>
      <property name="javax.persistence.jdbc.password" value=""/>
      <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
      <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

      <!-- 옵션 -->
      <property name="hibernate.show_sql" value="true"/>
      <property name="hibernate.format_sql" value="true"/>
      <property name="hibernate.use_sql_comments" value="true"/>
      <property name="hibernate.id.new_generator_mappings" value="true"/>
      
    </properties>
  </persistence-unit>
</persistence>
```

* dialect는 데이터베이스 방언이라는 뜻인데,

## 데이터베이스 방언

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

![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/04_jpa_dialect.PNG?raw=true)

## 애플리케이션 개발

* 엔티티 매니저 팩토리 설정
  * Persistence를 통해서 xml설정을 읽고, EntityManagerFactory를 만들어야 한다.
  * 트랜잭션 단위로 기능을 만들때 항상 엔티티 매니저를 새로 만들어야 한다. EntityManagerFactory를 통해서 만든다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/05_jpa_entity_manager.PNG?raw=true)

* 엔티티 매니저 설정
* 트랜잭션
* 비즈니스 로직(CRUD)
* 주의할 점
  * 엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에서 공유
  * 엔티티 매니저는 쓰레드간에 공유하면 안된다(사용하고 버려야 한다.)
  * JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

```java
package basic;

import basic.entity.Member;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class Main {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory =
            Persistence.createEntityManagerFactory("hello");
        EntityManager em = entityManagerFactory.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            Member member = new Member();
            member.setId(100L);
            member.setName("name");

            em.persist(member);

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

## Reference

- 자바 ORM 표준 JPA 프로그래밍
- 저자 직강 - <https://www.youtube.com/watch?v=WfrSN9Z7MiA&list=PL9mhQYIlKEhfpMVndI23RwWTL9-VL-B7U>