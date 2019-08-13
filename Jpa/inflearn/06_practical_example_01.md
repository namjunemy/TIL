# 실전 예제 1 - 요구사항 분석과 기본 매핑

### 요구사항 분석

- 회원은 상품을 주문할 수 있다.

- 주문 시 여러 종류의 상품을 선택할 수 있다.

### 기능

- 회원
    - 등록, 조회
- 상품
    - 등록, 수정, 조회
- 주문
    - 상품주문, 주문내역조회, 주문취소

## 도메인 모델 분석

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/05_domain_model.png?raw=true)

* 회원과 주문의 관계
    * **회원**은 여러 번 **주문**할 수 있다.(일대다 관계)
* 주문과 상품의 관계
    * **주문**할 때 여러 **상품**을 선택할 수 있다.
    * 반대로 같은 **상품**도 여러 번 **주문**될 수 있다.
    * **주문상품 이라는 모델**을 만들어서 다대다 관계를 일대다, 다대일 관계로 풀어낼 수 있다.

## 테이블 설계

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/06_table.png?raw=true)

## 엔티티 설계와 매핑

* 테이블 설계에 맞춰서 아래와 같이 엔티티를 설계하고 매핑 한다.
* 이렇게 해보고, 실제로 문제점이 뭔지 파악하자

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/07_entity.PNG?raw=true)

* [엔티티 매핑 커밋 로그](https://github.com/namjunemy/orm-jpa-basic/commit/b538f7f12bf97112491816dfdf43c6a4b53e342e)

* 컬럼의 제약조건들을 엔티티 필드에 선언해 놓으면 굳이 DB 테이블 상세조회 하지 않아도 된다.

* 인덱스들도 엔티티에 @Table(indexex = ) 속성으로 추가해 놓으면 나중에 객체를 중심으로 JPQL 쿼리 작성할 때 도움이 된다.

  * 인덱스를 실제로 거는게 아니고, 쿼리짤 때 참조 가능.

* 위와 같이 엔티티를 만들고 주문 정보를 조회해서 주문한 회원을 참조하려고 할때, 아래와 같이 코드를 짤 수 밖에 없다.

  * `order`  객체에서 `getMember()` 를 통해서 멤버를 참조할 수 있어야 객체지향스럽게 코드를 작성할 수 있다.

  ```JAVA
  Order order = em.find(Order.class, 1L);
  Long memberId = order.getMemberId();
  
  Member member = em.find(Member.class, memberId);
  // order.getMember();
  ```

### 데이터 중심 설계의 문제점

* 위의 방식은 객체 설계를 테이블 설계에 맞춘 방식이다.
* 테이블의 외래키를 객체에 그대로 저장한다.
* 객체 그래프의 탐색이 불가능하고, 참조가 없으므로 UML도 사실 잘못 된것이다.
* 이 문제를 해결하고자 이제 연관관계 매핑을 배운다.



