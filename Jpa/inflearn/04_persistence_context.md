# 영속성 컨텍스트

JPA를 공부할 때 가장 중요한게 **객체와 관계형 데이터베이스를 매핑하는 것(Object Relational Mapping)**과 **영속성 컨텍스트**를 이해하는 것이다. 두가지 개념은 꼭 알고 JPA를 활용하자.

> 영속성 컨텍스트
>
> 프록시와 즉시로딩, 지연로딩

## 엔티티 매니저 팩토리와 엔티티 매니저

- JPA는 스레드가 하나 생성될 때 마다(매 요청마다) EntityManagerFactory에서 EntityManager를 생성한다.
- EntityManager는 내부적으로 DB 커넥션 풀을 사용해서 DB에 붙는다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/10_jpa_em.PNG?raw=true)

## 영속성 컨텍스트

- 영속성 컨텍스트는 **JPA를 이해하는데 가장 중요한 용어**이다.

- "**엔티티를 영구 저장하는 환경**"이라는 뜻

- `EntityManager.persist(entity);`

  - 앞의 예제에서 persist()로 db에 객체를 저장하는 것이라고 배웠지만,
  - 실제로는 DB에 저장하는 것이 아니라, 영속성 컨텍스트를 통해서 엔티티를 영속화 한다는 뜻이다.
  - 정확히 말하면 persist() 시점에는 영속성 컨텍스트에 저장한다. DB 저장은 이후이다.

- 엔티티 매니저? 영속성 컨텍스트?

  - **영속성 컨텍스트는 논리적인 개념**이다.
  - 눈에 보이지 않는다.
  - 엔티티 매니저를 통해서 영속성 컨텍스트에 접근한다.

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/11_jpa_em.PNG?raw=true)

  - 스프링에서 EntityManager를 주입 받아서 쓰면, 같은 트랜잭션의 범위에 있는 EntityManager는 동일 영속성 컨텍스트에 접근한다.

## 엔티티의 생명주기

![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/12_entity_lifecycle.PNG?raw=true)

- **비영속(new/transient)**

  - 영속성 컨텍스트와 전혀 관계가 없는 상태

    ```java
    // 객체를 생성만 한 상태(비영속)
    Member member = new Member();
    member.setId("member1");
    member.setUsername("회원1");
    ```

- **영속(managed)**

  - 영속성 컨텍스트에 저장된 상태

  - 엔티티가 영속성 컨텍스트에 의해 관리된다.

  - 이때 DB에 저장 되지 않는다. 영속 상태가 된다고 DB에 쿼리가 날라가지 않는다.

  - 트랜잭션의 커밋 시점에 영속성 컨텍스트에 있는 정보들이 DB에 쿼리로 날라간다.

    ```java
    // 객체를 생성한 상태(비영속)
    Member member = new Member();
    member.setId("member1");
    member.setUsername("회원1");
    
    EntityManager em = emf.createEntityManager();
    em.getTransaction().begin();
    
    // 객체를 저장한 상태(영속)
    em.persist(member);
    ```

- **준영속(detached)**

  - 영속성 컨텍스트에 저장되었다가 분리된 상태

    ```java
    // 회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
    em.detach(member);
    ```

- **삭제(removed)**

  - 삭제된 상태. DB에서도 날린다.

    ```java
    // 객체를 삭제한 상태
    em.remove(member);
    ```

## 영속성 컨텍스트의 이점

- 애플리케이션과 DB사이에 왜 중간에 영속성 컨텍스트가 있냐. 왜 필요하냐. 아래와 같은 개념들이 가능하려면, 영속성 컨텍스트가 존재해야 한다.

### 1차 캐시

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/03_persistence_context_cache.PNG?raw=true)

- 영속성 컨텍스트(엔티티 매니저)에는 내부에 1차 캐시가 존재한다.

- 엔티티를 영속성 컨텍스트에 저장하는 순간. 1차 캐시에

- key : @Id로 선언한 필드 값, value : 해당 엔티티 자체로 캐시에 저장된다.

- 1차 캐시가 있으면 어떤 이점이있을까?  조회할 때 이점이 생긴다.

- find()가 일어나는 순간, 엔티티 매니저 내부의 1차 캐시를 먼저 찾는다.

- 1차 캐시에 엔티티가 존재하면 바로 반환한다. DB 들리지 않는다.

- **주의! 1차 캐시는 글로벌하지 않다. 해당 스레드 하나가 시작할때 부터 끝날때 까지 잠깐 쓰는거다. 공유하지 않는 캐시다**

  - 100명 한테 요청 100개 오면, 엔티티 매니저 100개 생기고 1차캐시도 100개 생긴다. 스레드 종료되면, 그때 다 사라진다.
  - 트랜잭션의 범위 안에서만 사용하는 굉장히 짧은 캐시 레이어이다.
  - 전체에서 쓰는 글로벌 캐시는 2차 캐시라고 한다.

  ```java
  Member member = new Member();
  member.setId("member1");
  member.setUsername("회원1");
  
  ...
  
  // 1차 캐시에 저장됨
  em.persist(member);
  
  // 1차 캐시에서 조회
  Member findMember = em.find(Member.class, "Member1");
  ```

- **1차 캐시에 데이터가 없다면? 데이터베이스에서 조회 한다.**

  - member2를 조회하는데 1차 캐시에 해당 엔티티가 없다.
  - 그러면 DB에서 꺼내온다.
  - 그리고 **1차 캐시에 저장**한다.
  - 그 후에 member2를 반환한다.
  - 다시 member2를 조회하면, 1차 캐시에 있는 member2가 반환된다.
    * SELECT 쿼리가 안나간다. 한 트랜잭션 내에서.

### 동일성(identity) 보장

- 영속 엔티티의 동일성을 보장한다.

- 1차 캐시 덕분에 member1을 두번 조회해도 다를 객체가 아니다. 같은 레퍼런스가 된다.

- 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 **데이터베이스가 아닌 애플리케이션 차원에서 제공**한다.

  ```java
  Member a = em.find(Member.class, "Member1");
  Member b = em.find(Member.class, "Member2");
  
  Systen.out.println(a == b); // 동일성 비교 true 
  ```

### 트랜잭션을 지원하는 쓰기 지연(transactional write-behind) - 엔티티 등록

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/04_transactional_write_behind.PNG?raw=true)

- 트랜잭션 내부에서 `persist()`가 일어날 때, 

- 엔티티들을 1차 캐시에 저장하고, 논리적으로 **쓰기 지연 SQL 저장소** 라는 곳에 **INSERT 쿼리들을 생성해서 쌓아 놓는다**.

- DB에 바로 넣지 않고 기다린다.

- 언제 넣냐. `commit()`하는 시점에 DB에 동시에 쿼리들을 쫙 보낸다.(쿼리를 보내는 방식은 동시 or 하나씩 옵션에 따라)

- 이렇게 쌓여있는 쿼리들을 DB에 보내는 동작이 `flush()` 이다.

- `flush()` 는 1차캐시를 지우지는 않는다. 쿼리들을 DB에 날려서 DB와 싱크를 맞추는 역할을 한다.

- 실제로 쿼리를 보내고 나서, `commit()`한다.

- 트랜잭션을 커밋하게 되면, `flush() 와 commit()` 두가지 일을 하게 되는 것이다.

  ```java
  EntityManager em = emf.createEntityManager();
  EntityTransaction transaction = em.getTransaction();
  // 엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
  transaction.begin(); // 트랜잭션 시작
  
  em.persist(memberA);
  em.persist(memberB);
  // 이때까지 INSERT SQL을 데이터베이스에 보내지 않는다.
  
  // 커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
  transaction.commit(); // 트랜잭션 커밋
  ```

* persistence.xml에 아래와 같은 옵션을 줄 수 있다.

  * JDBC 일괄 처리 옵션으로 커밋 직전까지 insert 쿼리를 해당 사이즈 만큼 모아서 한번에 처리한다. 

  ```xml
  <property name="hibernate.jdbc.batch_size" value=10/>
  ```

### 변경 감지(Dirty Checking) - 엔티티 수정

- 엔티티 수정이 일어나면 update()나 persist()로 영속성 컨텍스트에 알려줘야 하지 않을까?

  ```java
  EntityManager em = emf.createEntityManager();
  EntityTransaction transaction = em.getTransaction();
  transaction.begin(); // 트랜잭션 시작
  
  // 영속 엔티티 조회
  Member memberA = em.find(Member.class, "memberA");
  
  // 영속 엔티티 수정
  memberA.setUsername("nj");
  memberA.setAge(27);
  
  //em.update(member) 또는 em.persist(member)로 다시 저장해야 하지 않을까?
  
  transaction.commit(); // 트랜잭션 커밋
  ```

- 엔티티 데이터만 수정하면 끝이다. 데이터만 set하고 트랜잭션을 커밋하면 자동으로 업데이트 쿼리가 나간다.

- 어떻게 이게 가능할까?

- 변경 감지를 Dirty Checking이라고 한다.

- 사실은 1차 캐시에 저장할 때 동시에 스냅샷 필드도 저장한다.

- 그러고나서 commit()또는 flush()가 일어날 때 **엔티티와 스냅샷을 비교**해서, 변경사항이있으면 UPDATE SQL을 알아서 만들어서 DB에 저장한다.

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/13_dirty_checking.PNG?raw=true)

- update() 만들면 되지 왜 이렇게 복잡한 방법으로 처리하나...

  - 사상 때문이다. 우리는 자바 컬렉션에서, 리스트에서 값을 변경하고 리스트에 다시 그 값을 담지 않는다.
  - 값을 변경하면 변경된 리스트가 유지되는 것과 같은 컨셉이다.

- 따라서, 영속상태의 엔티티를 가져와서 값만 바꾸면 수정은 끝이다.

### 엔티티 삭제

```java
Member memberA = em.find(Member.class, "memberA");

em.remove(memberA); // 엔티티 삭제
```

* 삭제는 위의 매커니즘이랑 같고, 트랜잭션의 commit 시점에 DELETE 쿼리가 나간다.

## 플러시

* 플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.
* 트랜잭션 커밋이 일어날 때 플러시가 동작하는데, 쓰기 지연 저장소에 쌓아 놨던 INSERT, UPDATE, DELETE SQL들이 데이터베이스에 날라간다.
*  쉽게 얘기해서 영속성 컨텍스트의 변경 사항들과 데이터베이스를 싱크하는 작업이다.

### 플러시 발생

* 플러시가 발생하면 어떤 일이 생기나
  * 변경을 감지한다. Dirty Checking.
  * 수정된 엔티티를 쓰기 지연 SQL 저장소에 등록한다.
  * 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.(등록, 수정, 삭제 SQL)
  * 플러시가 발생한 다고 커밋이 이루어지는게 아니고, 플러시 다음에 커밋이 일어난다.

### 영속성 컨텍스트를 Flush 하는 방법

* **em.flush()** 로 직접호출

  ```java
  // 영속
  Member member = new Member(200L, "A");
  em.persist(member);
  
  em.flush();
  
  System.out.println("플러시 직접 호출하면 쿼리가 커밋 전 플러시 호출 시점에 나감");
  
  em.commit();
  ```

* **트랜잭션 커밋**시 플러시 자동 호출

* **JPQL 쿼리 실행**하면 플러시 자동 호출

  * JPQL 쿼리 실행시 플러시가 자동으로 호출되는 이유는

  * 아래와 같이 member1,2,3을 영속화한 상태에서. 쿼리는 안날라간 상태

  * JPQL로 SELECT 쿼리를 날리려고 하면 저장되어 있는 값이 없어서 문제가 생길 수 있다.

  * JPA는 이런 상황을 방지하고자 JPQL 실행 전에 무조건 flush()로 DB와의 싱크를 맞춘 다음에 JPQL 쿼리를 날리도록 설정 되어 있다.

  * 그래서 아래의 상황에서는 JPQL로 멤버들을 조회할 수 있다.

    ```java
    em.persist(memberA);
    em.persist(memberA);
    em.persist(memberA);
    
    // 중간에 JPQL 실행
    query = em.createQuery("select m from Member m", Member.class);
    List<Member> members = query.getResultList();
    ```

* **플러시가 일어나면 1차 캐시가 삭제될까?**
  * 삭제 되지 않는다. 쓰기 지연 SQL 저장소에 있는 쿼리들만 DB에 전송되고 1차 캐시는 남아있다.

### 플러시 모드 옵션

* `em.setFlushMode(FlushModeType.COMMIT);`
  * **FlushModeType.AUTO**
    * 커밋이나 쿼리를 실행할 때 플러시(**기본값**)
  * FlushModeType.COMMIT
    * 커밋 할때만 플러시

### 플러시 정리

* 플러시는 영속성 컨텍스트를 비우지 않는다. 오해하면 안된다.
* 플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화 한다.
* 플러시가 동작할 수 있는 이유는 데이터베이스 트랜잭션이라는 작업 단위(개념)가 있기 때문이다.
  * 어쨋든 트랜잭션이 시작되고 커밋되는 시점에만 동기화 해주면 되기 때문에, 그 사이에서 플러시 매커니즘의 동작이 가능한 것이다.
* JPA는 기본적으로 데이터를 맞추거나 동시성에 관련된 것들은 데이터베이스 트랜잭션에 위임한다. 참고로 알아두자.



