# Spring Data JPA와 QueryDSL

> JPA 기반 프로젝트
>
> * Spring Data JPA
> * QueryDSL

## Spring Data JPA

* 지루하게 반복되는 CRUD 문제를 세련된 방법으로 해결
* 개발자는 인터페이스만 작성한다
* 스프링 데이터 JPA가 구현 객체를 동적으로 생성해서 주입

* 스프링 데이터 JPA 적용 전

    ```java
    public class MemberRepository {
        public void save(Member member) {...}
        public Member findOne(Long id) {...}
        public List<Member> findAll() {...}
        
        public Member findByUsername(String username) {...}
    }
    ```

    ```java
    public class ItemRepository {
        public void save(Item item) {...}
        public Item one(Long id) {...}
        public List<Item> findAll() {...}
    }
    ```

* 스프링 데이터 JPA 적용 후

    * 스프링 데이터 JPA에서는 JpaRepository 인터페이스를 제공한다.

    * 우리가 상상할 수 있는 모든 API를 모두 처리한다.

    * 인터페이스가 인터페이스를 상속받을 때는 extends를 사용한다.

        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> {
            Member findByUsername(String username);
        }
        ```

        ```java
        public interface ItemRepository extends JpaRepository<Item, Long> {
            // 비어있음
        }
        ```

### 스프링 데이터 JPA 적용 후 클래스 다이어그램

* JpaRepository에서 엄청 많은 기능을 제공한다. 인터페이스 상속 받아서 사용하면 된다.

* 기본적으로 CRUD를 구현하지 않아도 된다. 인터페이스 호출해서 쓸 수 있다.

* 스프링 로딩 시점에 MemberRepository와 ItemRepository의 구현체를 만든다.

    ![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/15_jpa_repo.PNG?raw=true)

### 공통 인터페이스 기능

* JpaRepository 인터페이스: 공통 CRUD 제공

* 제네릭은 <엔티티, 식별자>로 설정한다.

* 스프링에 스프링 데이터 프로젝트와 스프링 데이터 JPA프로젝트가 따로 존재한다.

* 스프링 데이터 프로젝트에 공통적인 기능을 가지고 있고, JPA 특화된 기능을 스프링 데이터 JPA 프로젝트에서 가지고 있다.

    ![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/16_jpa_common_interface.PNG?raw=true)

### 쿼리 메서드 기능

* 위에서 스프링 데이터 JPA 적용 전의 코드를 보면 findByUsername() 메서드가 있다.
* JpaRepository만으로 이걸 어떻게 처리하나? 
* 이를 위해 쿼리 메서드라는 엄청난 기능이 존재한다.
* 메서드 이름으로 쿼리를 생성한다. @Query 어노테이션으로 쿼리를 직접 정의할 수 있다.

### 메서드 이름으로 쿼리 생성

* **메서드 이름만으로 JPQL 쿼리를 생성**한다.

* 선언된 메서드에 대해서는 애플리케이션 로딩 시점에 쿼리를 다 만들어 버린다.

* 그래서 에러를 사전에 잡을 수 있다.

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        List<Member> findByName(String username);
    }
    ```

    ```java
    List<Member> member = memberRepository.findByName("hello")
    ```

    ```sql
    // 실행된 SQL
    SELECT * FROM MEMBER M WHERE M.NAME = "hello"
    ```

### 이름으로 검색 + 정렬

* 자주 쓰는 기능 정렬을 위해 Sort를 넣어줄 수 있다. 스프링 데이터에서 만들어 놨다.

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        List<Member> findByName(String username, Sort sort);
    }
    ```

    ```sql
    // 실행된 SQL
    SELECT * FROM MEMBER M WHERE M.NAME = "hello" ORDER BY AGE DESC
    ```

### 이름으로 검색 + 정렬 + 페이징

* 추가로 페이징을 위한 기능도 만들어 놨다.

* 실제로 사용할땐 꼭 DTO로 변환해서 뿌리자.

* Page객체의 content에 데이터가 나오고,

* Pageable, sort, first, empty등의 정보를 얻을 수 있다.

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        Page<Member> findByName(String username, Pageable pageable);
    }
    ```

    ```java
    // 예를 들어 이런식으로 사용할 수 있다.
    @GetMapping("/hello")
    public Page<Member> member() {
        PageRequest request = PageRequest.of(1, 10);
        return repository.findByName("hello1", request);
    }
    ```

* 사용 예시

    ```java
    Pageable page = new PageRequest(1, 20, new Sort...);
    Page<Member> result = memberRepository.findByName("hello", page);
    
    //전체 수
    int total = result.getTotalElements();
    
    //데이터
    List<Member> members = result.getContent();
    
    전체 페이지 수, 다음 페이지 및 페이징을 위한 API 다 구현되어 있음
    ```

* 실행된 SQL 2가지

    * 데이터 조회

        ```sql
        SELECT *
        FROM
            ( SELECT ROW_.*, ROWNUM ROWNUM_
              FROM
                  ( SELECT M.*
                    FROM MEMBER M WHERE M.NAME = 'hello'
                    OEDER BY M.NAME
                  ) ROW_
             WHERE ROWNUM <= ?
            )
        WHERE ROWNUM_>?
        ```

    * 전체 수 조회

        ```sql
        SELECT COUNT(1) FROM MEMBER M WEHRE M.NAME = 'hello'
        ```

### @Query, JPQL 정의

* @Query 어노테이션을 사용해서 직접 JPQL을 지정할 수 있다.

* 마찬가지로 Named 쿼리도 애플리케이션 로딩 시점에 파싱을 하므로, 이로 인해 런타임 에러를 내지 않을 수 있다.

    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        
        @Query("select m from Member m where m.username = ?1")
        Member findByUsername(String username, Pageable pageable);
    }
    ```

### Web 페이징과 정렬 기능

* 컨트롤러에서 페이징 처리 객체를 바로 인젝션 받을 수 있다

* page: 현재 페이지

* size: 한 페이지에 노출할 데이터 건수

* sort: 정렬 조건

* `/members?page=0&size=20&sort=name,desc`

    ```java
    @RequestMapping(value = "/members", method = RequestMethod.GET)
    String list(Pageable pageable, Model mobel) {}
    ```

## QueryDSL

* SQL, JPQL을 **코드로** 작성할 수 있도록 도와주는 빌더 API
* JPA 크리테이라에 비해서 편리하고 실용적이다
* 오픈소스

### SQL, JPQL의 문제점

* SQL, JPQL은 문자열이다. Type-check가 불가능하다.

* 잘 해봐야 애플리케이션 로딩 시점에 알 수 있다. 컴파일 시점에 알 수 있는 방법이 없다. 자바와 문자열의 한계이다.

* 해당 로직 실행 전까지 작동여부 확인을 할 수 없다.

* 해당 쿼리 실행 시점에 오류를 발견한다.

    ```sql
    SELECT * FROM MEMBERRR WHERE MEMBER_ID = '100'
    ```

### QueryDSL 장점

* 문자가 아닌 코드로 작성하자
* 컴파일 시점에 문법 오류를 발견하자.
* 코드 자동완성(IDE 도움)
* 단순하고 쉽다.  코드 모양이 JPQL과 거의 비슷하다
* 동적 쿼리

### QueryDSL - 동작원리 쿼리타입 생성

* 위의 장점을 실현하기 위해서 오픈 소스를 만들었다.

* Member.java @Entity를 가지고, QMember.java라는 QueryDSL 전용 객체를 만들어 버린다.

* 세팅이 좀 힘들지만, 세팅을 하고나면 아래와 같이 사용할 수 있다.(gradle, maven 도움을 받아서)

* 엔티티 매니저를 JPAQueryFactory에 넣고, 

* QMember 객체를 가지고 아래와 같은 방식으로 쿼리를 코드로 짤 수 있다.

* 가장 큰 장점은 IDE의 도움을 받을 수 있다는 것과, 컴파일 타임에 오류를 잡아서 뭐리 때문에 실수를 할 일이 없다는 것이다.

    ```java
    //JPQL
    select m from Member m where m.age > 18
        
        
    JPAFactoryQuery query = new JPAQueryFactory(em);
    QMember m = QMember.member;
    
    List<Member> list = 
        query.selectFrom(m)
             .where(m.age.gt(18))
             .orderBy(m.name.desc())
             .fetch();
    ```

### QueryDSL - 조인

```java
JPAQueryFactory query = new JPAQueryFactory(em);
QMember m = QMember.member;
QTeam t = QTeam.team;

List<Member> list = 
    query.selectFrom(m)
         .join(m.team, t)
         .where(t.name.eq("teamA"))
         .fetch();
```

### QueryDSL - 페이징 API

```java
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;

List<Member> list = 
    query.selectFrom(m)
         .where(m.age.gt(18))
         .orderBy(m.name.desc())
         .limit(10)                  // 페이징을 위해 limit, offset도 그냥 넣어줄 수 있다.
         .offset(10)
         .fetch();
```

### QueryDSL - 동적 쿼리

* 사실 QueryDSL을 쓰는 진짜 이유는 동적 쿼리 때문이다.

* JPQL은 정적 쿼리이다. 문자열을 더하기 해야하는데, 헬이다.

* QueryDSL은 코드를 더하는 것이기 때문에, 그것 보다 수월하게 할 수 있다.

* BooleanBuilder에 조건을 쭉쭉 넣고 쿼리를 실행시키면 된다. 더 자세히 공부해보자.

    * 조건이 있으면 넣고, 없으면 안넣고 불린빌더만 가지고 동적 쿼리가 생성이 된다!!!

* 원하는 필드만 뽑아서 DTO로 뽑아내는 기능도 QueryDSL이 다 지원한다.

    ```java
    String name = "member";
    int age = 9;
    
    QMember m = QMember.member;
    
    BooleanBuilder builder = new BooleanBuilder();
    if(name != null) {
        builder.and(m.name.contains(name));
    }
    if(age != 0) {
        builder.and(m.age.gt(age));
    }
    
    List<Member> list = 
        query.selectFrom(m)
             .where(builder)
             .fetch();
    ```

### QueryDSL - 이것은 자바다!

* 객체지향적인 관점에서 이 점이 가장 중요하다.

    * 예를 들어 쿠폰의 상태와 마케팅 뷰 카운트를 체크하는 공통으로 사용되는 서비스 필수 제약 조건이 있다고 가정하자.

        ```java
        return query.selectFrom(coupon)
                    .where(
                         coupon.type.eq(typeParam),
                         coupon.status.wq("LIVE"),
                         marketing.viewCount.lt(markting.maxCount)
               )
               .fetch();
        ```

* 자바다!!

* 그러므로 중복코드를 함수로 extract해서 재사용할 수 있다.

* 제약조건을 조립할 수가 있다.

    * extract method를 통해서 가독성과 재사용성을 확보하면서

    * 쿼리를 코드로 리팩토링 할 수 있다.

        ```java
        return query.selectFrom(coupon)
                    .where(
                         coupon.type.eq(typeParam),
                         isServiceable()
               )
               .fetch();
        
        private BooleanExpression isServiceable() {
            return coupon.status.wq("LIVE")
                .and(marketing.viewCount.lt(markting.maxCount));
        }
        ```

* 실무에서 JPA를 사용한다면, 스프링 데이터 JPA를 꼭 사용하자. QueryDSL도 꼭 사용하자.

## 실무 경험 공유

* 수 조 단위의 정산을 하는데, JPA로 다 처리한다.
* 크리티컬한 결제 같은 시스템도 JPA로 다 처리한다. SpringBoot + JPA + QueryDSL 기본으로 깔고 간다.
* JPA로 실무를 하다 보면, 테이블 중심에서 객체 중심으로 개발 패러다임이 변화된다.
* 유연한 데이터베이스 변경의 장점과 테스트
    * Junit 통합 테스트시에 H2 DB 메모리 모드로 돌려서 사용한다.
    * 로컬 PC에는 H2 DB 서버 모드로 실행한다.
    * 개발 운영은 MySQL, Oracle로 한다.
    * 데이터베이스 방언을 설정만 바꾸면 가능한 일이다.
* 데이터베이스 변경 경험(개발 도중 MySQL -> Oracle로 바뀐적도 있다.)
* 테스트, 통합 테스트시에 CRUD를 믿고 간다.(내가 짠 쿼리는 그것 마저 테스트를 거쳐 가야 한다.)
    * 이런거 테스트 할 시간에 CRUD 믿고, 핵심 비즈니스 테스트 코드를 열심히 짜자.
* 빠르게 에러를 발견할 수 있다.
    * 쿼리 때문에 문제가 발생한적이 한번도 없다.
    * 컴파일 시점에 대부분 오류를 발견할 수 있다.
    * 늦어도 애플리케이션 로딩 시점에 발견한다.
    * 최소한 뭐리 문법 실수나 오류는 거의 발생하지 않는다.
* 대부분이 비즈니스 로직의 오류이다.

### 성능

* JPA 자체로 인한 성능 저하 이슈는 거의 없다.
* 성능 이슈 대부분은 JPA를 잘 이해하지 못해서 발생한다.
    * 즉시로딩: 쿼리가 튄다. -> 지연 로딩을 변경한다.
        * 어노테이션 수정으로 바로 적용 된다.
    * N + 1 문제 -> 대부분 페치 조인으로 문제를 해결한다.
        * 이 부분, 이부분 페치 조인으로 바꾸자. SQL 베이스로 프로젝트를 짰다면 갈아 엎어야 한다.
    * 하나의 쿼리에 집중해서 성능을 고민하는게 아니라, JPA를 사용해서 빠르게 개발한 다음에
    * 실제 성능 테스트를 진행하고, 병목이 발생하는 구간들을 찾아서 설정을 빠르게 변경하고 다시 반복해서 성능 테스트 할 수 있다.

### 생산성

* 단순 코딩 시간이 줄어든다. -> 개발 생산성이 향상 된다. -> 잉여 시간이 발생한다.
* **비즈니스 로직 작성시 흐름이 끊기지 않는다.**
    * SQL 중심으로 개발하다 보면, 비즈니스 로직 자바로 짜다가 머리의 컨텍스트가 SQL로 스위칭 된다.
    * 다시 자바로 온다, 다시 SQL로 간다. 병목 발생하고 커피마시러 간다. 야근한다.
* 남는 시간에 더 많은 테스트를 작성하고
    * 순수 자바 코드가 많아져서 테스트 코드 만들기에 용이하다.
* 남는 시간에 기술 공부를 하고
* 남는 시간에 코드에 금칠을 하고
* 팀원 대부분은 다시는 과거로 돌아가고 싶어하지 않는다.
* 이 얘기는 JPA를 잘 공부하고, 잘 알고 프로젝트를 진행 했을 경우 이야기이다. 대부분 잘 모르고 진행하면 JPA를 원망한다.

### 많이 하는 질문

* ORM 프레임워크를 사용하면 SQL과 데이터베이스는 잘 몰라도 되나요?
    * 더 잘 알아야 한다. SQL과 DB를 모르고 ORM을 사용하는건 말이 안된다. 객체와 DB를 잘 알고 그것을 더 편하게 사용하려고 ORM을 사용하는 것이다. 결국 둘 다 잘해야 된다.
* 성능이 느리지 않나요?
    * 잘 쓰면 최적화 할 수 있는 포인트가 더 많다.
* 통계 쿼리처럼 매우 복잡한 SQL은 어떻게 하나요?
    * 거의 다 QueryDSL로 처리하고, DTO로 뽑아낸다.
    * 정말 안될 경우 네이티브 쿼리 사용한다.
* MyBatis와 어떤 차이가 있나요?
    * JPA를 사용하면서 같이 써도 된다. 다만, flush 같은 이슈 처리들을 잘 고려 해야 한다.
    * MyBatis는 쿼리를 직접 다 짜야 하지 않나.
* 하이버네이트 프레임워크를 신뢰할 수 있나요?
    * 쿠팡, 배민 거래량이 조단위로 들어가는 회사들에서 기본으로 JPA 깔고 간다.
* 제 주위에는 MyBatis(iBatis, myBatis)만 사용하는데요?
    * 주변에 JPA 할 줄 아는 사람이 있어야 같이 적용하지, 사실 쿼리중심으로 개발하는 지금의 SI에서는 힘들다.
* 학습 곡선이 높다고 하던데요?
    * 일주일 공부하고 평생 시간 아끼자.
    * 어려운 포인트는 찾아서 딥하게 공부하자.
    *  시간이 절약되는 만큼 비즈니스 로직을 고민할 수 있는 시간이 늘어난다.

### 팀 서버 기술 스택

* Java8
* Spring Frmework(SpringBoot default)
* JPA, Hibername
* Spring Data JPA
* QueryDSL
* Junit, Spock(Test)

