# 엔티티 매핑

JPA를 공부할 때 가장 중요한게

**객체와 관계형 데이터베이스를 매핑하는 것(Object Relational Mapping)** 과

**영속성 컨텍스트와 JPA가 내부적으로 어떤 매커니즘으로 동작하는지 이해하는 것** 이다.

### 엔티티 매핑 소개

* 객체와 테이블 매핑
  * @Entity, @Table
* 필드와 컬럼 매핑
  * @Column
* 기본 키 매핑
  * @Id
* 연관관계 매핑
  * @ManyToOne, @JoinColumn

## 객체와 테이블 매핑

### @Entity

* @Entity가 붙은 클래스는 JPA가 관리, 엔티티라고 한다.
* JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수
* **주의할 점**
  * 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
  * final 클래스, enum, interface, inner 클래스 사용X
  * 저장할 필드에 final 사용 X
* @Entity 속성
  * @Entity(name = "Member")
    * JPA에서 사용할 엔티티 이름 지정
    * Default는 클래스 이름을 그대로 사용
    * 다른 패키지에 같은 클래스 이름이 없어서 겹칠 염려가 없다면, 가급적 기본값을 사용한다.

### @Table

* @Table은 엔티티와 매핑할 테이블을 지정한다
* ex)
  * `@Table(name = "MBR")`
  * `INSERT INTO ... MBR`

| 속성                   | 기능                               | 기본값      |
| ---------------------- | ---------------------------------- | ----------- |
| name                   | 매핑할 테이블 이름                 | 엔티티 이름 |
| catalog                | 데이터베이스 catalog 매핑          |             |
| schema                 | 데이터베이스 schema 매핑           |             |
| uniqueConstraints(DDL) | DDL 생성시에 유니크 제약 조건 생성 |             |

### Reference

- [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)

