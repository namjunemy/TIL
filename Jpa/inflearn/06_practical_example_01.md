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