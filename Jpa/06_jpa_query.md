# JPA 객체지향 쿼리

앞의 내용들로 기본기를 다졌고, 지금부터는 활용 단계이다.

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
* 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
* 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
* 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.