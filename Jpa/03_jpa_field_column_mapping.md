# JPA 필드와 컬럼 매핑

## 데이터베이스 스키마 자동 생성하기

* DDL을 애플리케이션 실행 시점에 자동 생성
* 테이블 중심 -> 객체 중심으로 이동한 것이다.
* 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성

* **이렇게 생성된 DDL은 개발 장비에서만 사용**
* 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용
* hibernate.hbm2ddl.auto
  * create
    * 기존 테이블 삭제 후 다시 생성(DROP + CREATE)
  * create-drop
    * create와 같으나 종료시점에 테이블 DROP(테스트에서 사용하면 도움됨)
  * update
    * 변경분만 반영(운영DB에는 사용하면 안됨)
  * validate
    * 엔티티와 테이블이 정상 매핑되었는지만 확인
  * none
    * 사용하지 않음

* **데이터베이스 스키마 자동 생성하기 주의**
  * 운영 장비에는 절대 create, create-drop, update 사용하면 안된다.
  * 개발 초기 단계는 create 또는 update
  * 테스트 서버는 update 또는 validate
  * 스테이징과 운영 서버는 validate 또는 none
    * 웬만하면 쓰면 안된다.

## 매핑 어노테이션

데이터베이스에 어떤식으로 매핑할지를 결정하는 철저히 매핑정보에 관련된 어노테이션들이다.

```java
@Entity
public class Member {
  @Id
  private Lond id;
  
  @Column(name = "USERNAME")
  private String name;
  
  private int age;
  
  @Temporal(TemporalType.TIMESTAMP)
  private Date regDate;
  
  @Enumerated(EnumType.STRING)
  private MemberType memberType;
  
  ...
}
```

* @Column

  * @Column(name = "USERNAME")

  * name
    * 필드와 매핑할 테이블의 컬럼 이름
  * insertable, updatable
    * TRUE/FALSE 설정. 읽기 전용
  * nullable
    * null 허용여부 결정, DDL 생성시 사용(not null 추가)
  * unique
    * 유니크 제약 조건, DDL 생성시 사용
  * columnDefinition, length, precision, scale(DDL)

* @Temporal

  * @Temporal(TemporalType.TIMESTAMP)
  * 시간과 관련된 매핑
  * Date 뿐만아니라 자바8에서 지원하는 LocalDatetime도 지원한다.

* @Enumerates

  * @Enumerated(EnumType.STRING)
  * 자바의 Enum 타입 매핑을 지원한다.
  * 현업에서는 EnumType을 무조건 STRING으로 지정 해야한다.
  * 기본 값인 ORDINAL로 설정하면 Enum 순서로 숫자가 매핑되는데, Enum 중간에 필드가 하나 추가 되면 다 꼬이게 된다.

* @Lob

  * 컨텐츠의 길이가 너무 길 경우 바이너리 파일로 DB에 바로 밀어 넣어야 하는데, 보통 이런 경우에 사용한다.
  * 공통적으로 @Lob으로 사용하면 된다.
  * CLOB, BLOB 매핑
  * CLOB : String, char[], java.sql.CLOB
  * BLOB : byte[], java.sql.BLOB

* @Transient

  * 이 필드는 매핑하지 않는다.
  * 애플리케이션에서 DB에 저장하지 않는 필드
  * 웬만하면 쓰지 않는 것이...

## 식별자 매핑 어노테이션

* 식별자 매핑 방법
  * @Id(직접 매핑)
  * @GeneratedValue(strategy = GenerationType.[**타입**])
    * 타입 설정
    * IDENTITY
      * 데이터베이스에 위임, MYSQL
    * SEQUENCE
      * 데이터베이스 시퀀스 오브젝트 사용, ORACLE
      * @SequenceGenerator 필요
    * TABLE
      * 키 생성용 테이블 사용, 모든 DB에서 사용
      * @TableGenerator 필요
    * AUTO
      * 방언에 따라 자동 지정, 기본값

## 권장하는 식별자 전략

* 기본 키 제약 조건
  * null 아님, 유일, 변하면 안된다.
* 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용하자.
* 예를 들어 주민등록번호도 기본 키로 적절하지 않다.
  * 주민번호를 PK로 설정하고 다른 테이블과 FK 참조시 그대로 주민번호가 넘어간다. 갑자기 나라에서 개인정보 목적으로 "DB에 주민번호 저장하지 마라"라고 한다. 헬게이트 오픈
* **권장**
  * **Long + 대체키 + 키 생성전략 사용**
  * 대체키는 전혀 비즈니스랑 관계없는 키.
  * AUTO_INCREMENT로 숫자로 PK를 사용하면, int쓰면 안된다. 생각보다 금방 끝난다..
  * Long 타입으로.

## Reference

- 자바 ORM 표준 JPA 프로그래밍
- 저자 직강 - <https://www.youtube.com/watch?v=WfrSN9Z7MiA&list=PL9mhQYIlKEhfpMVndI23RwWTL9-VL-B7U>