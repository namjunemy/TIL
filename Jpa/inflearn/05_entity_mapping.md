# **엔티티** 매핑

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

## 데이터베이스 스키마 자동 생성

* 사실 엔티티의 매핑 정보만 보면 어떤 테이블인지, 어떤 쿼리를 만들어야 되는지 알 수 있다. 그래서 JPA에서는 아예 어플리케이션 로딩시점에 DB 테이블을 자동으로 생성해주는 기능을 지원한다.

    * DDL을 애플리케이션 실행 시점에 자동 생성

* 운영에서는 사용하면 위험하고, 개발단이나 로컬에서 개발할 때만 사용해야 한다.

    * 사용하더라도 생성된 DDL을 적절히 다듬은 후에 사용할 것
    * 초기 개발 시점이나 로컬에서 빠르게 개발할 때, 필드 추가하고 -> DB CREATE문 추가하고 하는 작업 생략 가능

* DB 방언을 활용해서 데이터베이스에 맞는 적절한 DDL문을 생성한다.

* **hibernate.hbm2ddl.auto 옵션**

    | 옵션        | 설명                                                         |
    | ----------- | ------------------------------------------------------------ |
    | create      | 기존 테이블 삭제 후 다시 생성(DROP + CREATE)                 |
    | create-drop | create와 같으나 종료시점에 테이블 DROP(테스트에서 사용하면 도움됨) |
    | update      | 변경분만 반영(운영DB에는 사용하면 안됨). **추가하는 변경분만 반영, DROP X** |
    | validate    | 엔티티와 테이블이 정상 매핑되었는지만 확인                   |
    | none        | 사용하지 않음                                                |

### DDL 자동 생성 주의할 점

* **운영 장비에는 절대 create, create-drop, update 사용하면 안된다.**
* 개발 초기 단계는 create 또는 update
* 어느정도 진행 후 테스트 서버는 update 또는 validate
    * 여러 명의 개발자가 같이 쓰는 테스트 서버에서는 create 사용하지 말자, 테스트 했던 데이터 다 날아간다.
* 스테이징과 운영 서버는 validate 또는 none
    - DDL을 애플리케이션 로딩 시점에  update로 alter만 한다고 해도, 볼륨이 큰 운영 DB에 대해서 수행하면 DB 락 걸리면서 시스템 장애로 이어진다.
    - 웬만하면 자동으로 만들어진 DDL 보다는, 직접 작성해서 테스트 서버에서 검증한 스크립트를 쓰자.
    - 결론은 가급적이면 다수가 사용하는 스테이징, 운영(테스트 서버 포함)에서는 ddl-auto 사용하지 말자.
        - **근본적으로 웹 어플리케이션에서 사용하는 DB계정은 운영 장비에서 alter나, drop 못하도록 계정 자체를 분리하는 것이 좋다.** 

### DDL 생성 기능

* @Column을 사용해서 제약 조건을 추가 할 수 있다.
    * 회원 이름은 필수, 10자 초과 X
    * @Column(nullable = false, length = 10)
* 유니크 제약 조건 추가
    * @Column(unique = true)
* 위와 같이 DDL을 생성할 때 우리가 사용하는 어노테이션들(**DDL 생성 기능**)은 JPA의 실행 매커니즘에 영향을 주지 않는다. DB에만 영향을 준다.
    * 매핑테이블 지정 @Table(name = "another")은 런타임에 영향을 준다.

## 필드와 컬럼 매핑

* 이해를 돕기위한 요구사항
    * 회원은 일반 회원과 관리자로 구분해야 한다.
    * 회원 가입일과 수정일이 있어야 한다.
    * 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.

```java
@Entity
public class Member {

    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;
}
```

### 필드-컬럼 매핑 어노테이션 정리

#### @Column

- @Column(name = "USERNAME")
- name
    - 필드와 매핑할 테이블의 컬럼 이름
    - default = 객체의 필드이름
- insertable, updatable
    - TRUE/FALSE 설정. 읽기 전용
    - default = true
- nullable(DDL)
    - null 허용여부 결정, DDL 생성시 사용(not null 제약 조건이 추가됨)
- unique(DDL)
    - 유니크 제약 조건, DDL 생성시 사용
    - 실제로는 필드에 조건을 주면 alter문으로 constraint 추가할 때 컬럼명이 랜덤으로 생성된다.
    - **@Table의 uniqueConstrains 옵션에 이름까지 줄 수 있고, 복수로 설정 가능하다.**
    - https://stackoverflow.com/questions/3126769/uniqueconstraint-annotation-in-java
- columnDefinition(DDL)
    - DB 컬럼 정의를 직접 할 수 있다.
    - 정의한 문구가 DDL에 그대로 들어간다.
    - default는 자바 필드 타입과 DB 방언 정보를 사용해서 만듦
    - ex)
        - varchar(100) default 'EMPTY'
- length(DDL)
    - 문의 길이 제약조건, String 타입에만 사용
    - default = 255
- precision, scale(DDL)
    - 큰 숫자를 표현하는 자료형 BigDecimal 타입(BigInteger도)에서 사용한다.
    - precision은 소수점을 포함한 전체 자릿수를 지정하는 옵션이고,
    - scale은 소수점 자리수를 지정하는 옵션이다.
    - Double, float 타입에는 적용되지 않는다. 

#### @Temporal

- @Temporal(TemporalType.TIMESTAMP)
- 날짜 타입. 시간과 관련된 매핑
- Date 뿐만아니라 자바8에서 지원하는 LocalDatetime도 지원한다.
- **LocalDate, LocalDateTime을 사용할 때는 어노테이션 생략 가능**하다.(최신 하이버네이트 지원)
    - LocalDate -> date 타입(ex : 2018-01-01)
    - LocalDateTime -> timestamp 타입 (ex : 2018-01-01 11:11:11)

#### @Enumerates

- @Enumerated(EnumType.STRING)
- 자바의 Enum 타입 매핑을 지원한다.
- 현**업에서는 EnumType을 무조건 STRING으로 지정** 해야한다.
- 기본 값인 ORDINAL로 설정하면 **Enum 순서로 숫자가 매핑되는데, Enum 중간에 필드가 하나 추가 되면 다 꼬이게 된다.**

#### @Lob

- 컨텐츠의 길이가 너무 길 경우(varchar를 넘는) 바이너리 파일로 DB에 바로 밀어 넣어야 하는데, 보통 이런 경우에 사용한다.
- 공통적으로 @Lob으로 사용하면 된다.
- CLOB, BLOB 매핑은 지정할 수 있는 속성이 없다. 아래의 기준으로 자동 매핑 된다.
    - CLOB : String, char[], java.sql.CLOB
    - BLOB : byte[], java.sql.BLOB

#### @Transient

- 이 필드는 매핑하지 않는다.
- 애플리케이션에서 DB에 저장하지 않는 필드
- 주로 메모리상에서만 임시로 관리하고 싶을 경우 사용

## 기본 키 매핑

### 기본 키 매핑 어노테이션

- @Id

    * @Id만 사용하면 직접 매핑, 직접 Id를 세팅함

- @GeneratedValue(strategy = GenerationType.[**타입**])

    - @Id + @GeneratedValue(strategy = GenerationType.[**타입**]) -> ID 자동 할당

    - **IDENTITY**

        - 기본 키 생성을 데이터베이스에 위임한다.
        - DDL 조건에 설정되어서 들어간다.
        - MYSQL, PostgreSQL, SQL Server, DB2에서 사용
        - Ex)
            - MYSQL의 AUTO_INCREMENT 
        - **IDENTITY 전략의 특징**
            - JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL을 실행한다.
            - 하지만 AUTO_INCREMENT는 DB INSERT SQL 실행 후에 ID값을 알 수 있다.
            - 그러면 영속성 컨텍스트 1차 캐시에 @Id값이 저장 못하나?
            - 이 경우를 위해서 IDENTITY 전략은 예외적으로 em.persist() 시점에 즉시  INSERT SQL을 실행한다.
            - 그리고 JDBC가 INSERT문을 실행하고 리턴하는 ID값을 리턴해서 사용한다.(ID값을 SELECT문으로 찾진 않음)
            - 결과적으로 쓰기지연(버퍼링)을 사용할 수 없다. 하지만, 1차 캐시는 한 트랜잭션의 범위 안에서만 사용하는 굉장히 짧은 캐시 레이어이기 때문에 성능의 차이가 극단적으로 나진 않는다. 

    - **SEQUENCE**

        - 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 DB 오브젝트(ORACLE 시퀀스)

        - ORACLE, PostgreSQL, DB2, H2 DB에서 사용

        - 기본적으로 하이버네이트가 만드는 기본 시퀀스 오브젝트를 쓰는데, 테이블마다 시퀀스를 따로 관리하고 싶을때는 @SequenceGenerator를 사용하면 된다. Auto DDL 기능으로 시퀀스가 생성된다.

        - **SEQUENCE 전략의 특징**

            - SEQUENCE 전략도 **DB에서 만드는 시퀀스 오브젝트를 참조해야 한다.**
            - em.persist()시에(영속성 컨텍스트에 저장할 때) DB 시퀀스 오브젝트에서 next value를 호출해서 가져온 다음에
            - 트랜잭션 커밋시에 INSERT문의 Id 값에 세팅해서 사용. 쓰기지연 사용 가능
            - allocationSize는 시퀀스 한 번 호출에 증가하는 수를 설정하는 옵션이다.
                - **기본값이 50으로 설정 되어 있는데**, 성능 최적화에 사용된다.
                    - INSERT문 수행시에 매번 시퀀스 테이블을 참조해서 next value를 가져오지 않고, allocationSize인 50만큼을 미리 어플리케이션 메모리에 올려놓고 사용한다.
                    - 쭉 사용하다가 시퀀스 값이 51일 때, 다시 한번 호출해서 메모리에 올린다.
                    - 여러대 서버의 요청에 대해서 동시성을 보장한다.
                - **데이터베이스에서 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값을 반드시 1로 설정해야 된다.**

            ```java
            @Entity
            @SequenceGenerator(
                name = "MEMBER_SEQ_GENERATOR",
                sequesceName = "MEMBER_SEQ",  // 매핑할 데이터베이스 시퀀스 이름
                initialValue = 1, allocationSize = 1)
            public class Member {
                @Id
                @GenerateValue(strategy = GenerationType.SEQUESCE, 
                               generator = "MEMBER_SEQ_GENERATOR")
                private Long id;
            }
            ```

    - **TABLE**

        - 키 생성용 테이블을 생성해서, 데이터베이스 시퀀스를 흉내낸다.
        - 모든 DB에서 사용 가능하지만, 최적화가 되어있지 않은 테이블을 직접 사용하니까 성능상의 이슈가 있다.
        - @SequenceGenerator와 유사하게 사용. 테이블 마다 @TableGenerator 필요
        - **TABLE 전략의 특징**
            - **SEQUENCE 전략의 특징** 참조

    - **AUTO**

        - 방언에 따라 자동 지정, 기본값

### 권장하는 식별자 전략

* 기본 키 제약 조건
    * null이 아니고
    * 유일해야 하며
    * **변하면 안된다.**

* 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용하자.
    * 자연키 - 비즈니스 적으로 의미가 있는 키(전화번호, 주민등록번호 등)

* 예를 들어, 주민등록번호도 기본 키로 적절하지 않다.
    * 주민번호를 PK로 설정하고 다른 테이블과 FK 참조시 그대로 주민번호가 넘어간다. 갑자기 나라에서 개인정보 목적으로 "DB에 주민번호 저장하지 마라"라고 한다. 마이그레이션 헬..오픈

* 권장
    * **Long + 대체키 + 키 생성전략 사용**
    * 대체키는 전혀 비즈니스랑 관계없는 키.
    * AUTO_INCREMENT로 숫자로 PK를 사용하면, int쓰면 안된다. 생각보다 금방 끝난다..
    * Long 타입으로.

### Reference

- [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)

