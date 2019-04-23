# JPA 내부구조

JPA를 공부할 때 가장 중요한게 **객체와 관계형 데이터베이스를 매핑하는 것(Object Relational Mapping)**과 **영속성 컨텍스트**를 이해하는 것이다. 두가지 개념은 꼭 알고 JPA를 활용하자.

> 영속성 컨텍스트
>
> 프록시와 즉시로딩, 지연로딩

## 엔티티 매니저 팩토리와 엔티티 매니저

* JPA는 스레드가 하나 생성될 때 마다(매 요청마다) EntityManagerFactory에서 EntityManager를 생성한다.
* EntityManager는 내부적으로 DB 커넥션 풀을 사용해서 DB에 붙는다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/10_jpa_em.PNG?raw=true)

## 영속성 컨텍스트

* 일단 JPA를 이해하는데 가장 중요한 용어이다.

* "엔티티를 영구 저장하는 환경"이라는 뜻

* `EntityManager.persist(entity);`

* 엔티티 매니저? 영속성 컨텍스트?

  * **영속성 컨텍스트는 논리적인 개념**이다.
  * 눈에 보이지 않는다.
  * 엔티티 매니저를 통해서 영속성 컨텍스트에 접근한다.

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/11_jpa_em.PNG?raw=true)

  * 스프링에서 EntityManager를 주입 받아서 쓰면, 같은 트랜잭션의 범위에 있는 EntityManager는 동일 영속성 컨텍스트에 접근한다.

## 엔티티의 생명주기

![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/12_entity_lifecycle.PNG?raw=true)

* 비영속(new/transient)

  * 영속성 컨텍스트와 전혀 관계가 없는 상태

    ```java
    // 객체를 생성한 상태(비영속)
    Member member = new Member();
    member.setId("member1");
    member.setUsername("회원1");
    ```

* 영속(managed)

  * 영속성 컨텍스트에 저장된 상태

  * 엔티티가 영속성 컨텍스트에 의해 관리된다.

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

* 준영속(detached)

  * 영속성 컨텍스트에 저장되었다가 분리된 상태

    ```java
    // 회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
    em.detach(member);
    ```

* 삭제(removed)

  * 삭제된 상태. DB에서도 날린다.

    ```java
    // 객체를 삭제한 상태
    em.remove(member);
    ```

## 영속성 컨텍스트의 이점

* 애플리케이션과 DB사이에 왜 중간에 영속성 컨텍스트가 있냐. 왜 필요하냐. 아래와 같은 개념들이 가능하려면, 영속성 컨텍스트가 존재해야 한다.

* **1차 캐시**

  * 영속성 컨텍스트(엔티티 매니저)에는 내부에 1차 캐시가 존재한다.

  * 엔티티를 영속성 컨텍스트에 저장하는 순간. 1차 캐시에

  * key : @Id로 선언한 필드 값, value : 해당 엔티티 자체로 캐시에 저장된다.

  * 따라서, 1차 캐시에서 조회하면 어떻게 되냐.

  * find()가 일어나는 순간, 엔티티 매니저 내부의 1차 캐시를 먼저 찾는다.

  * 1차 캐시에 엔티티가 존재하면 바로 반환한다. DB 들리지 않는다.

  * **주의! 1차 캐시는 글로벌하지 않다. 해당 스레드 하나가 시작할때 부터 끝날때 까지 잠깐 쓰는거다. 공유하지 않는 캐시다**

    * 100명 한테 요청 100개 오면, 엔티티 매니저 100개 생기고 1차캐시도 100개 생긴다. 스레드 종료되면, 그때 다 사라진다.
    * 트랜잭션의 범위 안에서만 사용하는 굉장히 짧은 캐시 레이어이다.

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

  * 데이터베이스에서 조회

    * member2를 조회하는데 1차 캐시에 해당 엔티티가 없다.
    * 그러면 DB에서 꺼내온다. 그리고 **1차 캐시에 저장**한다.
    * 그 후에 member2를 반환한다.

* **동일성(identity) 보장**

  * 영속 엔티티의 동일성을 보장한다.

  * 1차 캐시 덕분에 member1을 두번 조회해도 다를 객체가 아니다. 같은 레퍼런스가 된다.

  * 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다.

    ```java
    Member a = em.find(Member.class, "Member1");
    Member b = em.find(Member.class, "Member2");
    
    Systen.out.println(a == b); // 동일성 비교 true 
    ```

* **트랜잭션을 지원하는 쓰기 지연(transactional write-behind)**

  * 트랜잭션 내부에서 `persist()`가 일어날 때, 

  * 엔티티들을 1차 캐시에 저장하고, 논리적으로 쓰기 지연 SQL 저장소 라는 곳에 INSERT 쿼리들을 생성해서 쌓아 놓는다.

  * DB에 바로 넣지 않고 기다린다.

  * 언제 넣냐. `commit()`하는 시점에 DB에 동시에 쿼리들을 쫙 보낸다.(쿼리를 보내는 방식은 동시 or 하나씩 옵션에 따라)

  * 이렇게 쌓여있는 쿼리들을 DB에 보내는 동작이 `flush()` 이다.

  * `flush()` 는 1차캐시를 지우지는 않는다. 쿼리들을 DB에 날려서 DB와 싱크를 맞추는 역할을 한다.

  * 실제로 쿼리를 보내고 나서, `commit()`한다.

  * 트랜잭션을 커밋하게 되면, `flush() 와 commit()` 두가지 일을 하게 되는 것이다.

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

* **변경 감지(Dirty Checking)**

  * 엔티티 수정이 일어나면 update()나 persist()로 영속성 컨텍스트에 알려줘야 하지 않을까?

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

  * 엔티티 데이터만 수정하면 끝이다. 데이터만 set하고 트랜잭션을 커밋하면 자동으로 업데이트 쿼리가 나간다.

  * 어떻게 이게 가능할까?

  * 변경 감지를 Dirty Checking이라고 한다.

  * 사실은 1차 캐시에 저장할 때 동시에 스냅샷 필드도 저장한다.

  * 그러고나서 commit()또는 flush()가 일어날 때 **엔티티와 스냅샷을 비교**해서, 변경사항이있으면 UPDATE SQL을 알아서 만들어서 DB에 저장한다.

    ![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/13_dirty_checking.PNG?raw=true)

  * update() 만들면 되지 왜 이렇게 복잡한 방법으로 처리하나...

    * 사상 때문이다. 우리는 자바 컬렉션에서, 리스트에서 값을 변경하고 리스트에 다시 그 값을 담지 않는다.
    * 값을 변경하면 변경된 리스트가 유지되는 것과 같은 컨셉이다.

  * 따라서, 영속상태의 엔티티를 가져와서 값만 바꾸면 수정은 끝이다.

  * **Flush 발생**

    * 변경 감지
    * 수정된 엔티티 쓰기지연 SQL 저장소에 등록
    * 쓰기지연 SQL 저장소의 쿼리를 데이터베이스에 전송(등록, 수정, 삭제 쿼리)

  * 영속성 컨택스트를 Flush하는 방법

    * em.flush()로 직접호출

    * 트랜잭션 커밋시 플러시 자동 호출

    * JPQL 쿼리 실행하면 플러시 자동 호출

      * JPQL 쿼리 실행시 플러시가 자동으로 호출되는 이유는

      * 아래와 같은 상황에서 JPA가 제공하는  JPQL로 쿼리 날리면, 조회되지 않는다. JPA에서 자동화 되어있다.

      * 자동으로 플러시 되지 않는 다른 라이브러리와 섞어 쓰는 경우에는 flush()를 꼭 해줘야 된다.

        ```java
        em.persist(memberA);
        em.persist(memberA);
        em.persist(memberA);
        
        // 중간에 JPQL 실행
        query = em.createQuery("select m from Member m", Member.class);
        List<Member> members = query.getResultList();
        ```

  * `flush()` 는 !!

    * 영속성 컨텍스트를 비우지 않는다.
    * 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화 한다.
    * 플러시라는 개념이 존재할 수 있는 이유는!
      * DB에 트랜잭션이라는 작업의 단위가 있기 때문이다.
      * 커밋 직전에만 동기화 하면 된다.

* **지연 로딩(Lazy Loading)**

  * 뒤에서 자세히. 실전에서 애먹는다.

## 준영속 상태

* 영속 -> 준영속
* 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태(detached)
* 영속성 컨텍스트가 제공하는 기능을 사용하지 못함. 쿼리 안나감.
* 준영속 상태로 만드는 방법
  * em.detach(entity) - 특정 엔티티만 준영속 상태로 전환
  * em.clear() - 영속성 컨텍스트를 완전히 초기화
  * em.close() - 영속성 컨텍스트를 종료

## 프록시와 즉시로딩, 지연로딩

스프링 프레임워크 쓰다보면, 트랜잭션의 범위에서만 영속성 컨텍스트가 유지 된다. 그렇다면 서비스 레이어에서 컨트롤러 레이어로 나가는 순간, 영속성 컨텍스트가 유지가 안된다. 그러면 지연 로딩을 사용할 수가 없다!!

* Member를 조회할 때 Team도 함께 조회해야 할까?

  * 단순히 member 정보만 사용하는 비즈니스 로직에서는 굳이 팀을 미리 조회할 필요가 없다.
    ```java
    @Entity
    public class Member {
      @Id
      @GeneratedValue
      private Long id;
    
      @Column(name = "USERNAME")
      private String name;
    
      @ManyToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "TEAM_ID")
      private Team team;
      ...
    }
    ```
  
  * 실제로는 어떤식으로 동작할까?
  
    * LAZY 로딩 설정을 하면, 애플리케이션 로딩 시점에 Member의 Team에는 NPE가 나면 안되므로 프록시 객체(가짜 객체)가 들어간다.
    * 그리고 나서 멤버가 팀을 가져와서, **실제 사용하는 순간**에 팀의 값을 채운다. 쿼리 날려서 가져온다.
    
  
  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/14_lazy_proxy.PNG?raw=true)
  
* 위에서 언급했지만 스프링을 사용하면서 트랜잭션의 범위 밖 준영속 상태에서 조회를 하려고하면 LazyInitializationException을 마주치게 된다. 주의하자.

  * 범위에서 벗어난 이야기 지만 SpringBoot에서 Default로 open session in view(영속성 컨텍스트를 뷰 렌더링이 끝나는 시점까지 개방한 상태로 유지하는 것) 옵션을 TRUE로 주고 있다.
  * 해당 옵션이 TRUE라는 것은 뷰 렌더링 종료시까지 커넥션을 물고 있다는 이야기이다. 이와 관련해서 퍼포먼스적인 이슈가 동반되는데 추가적으로 공부해보자.
    * <http://egloos.zum.com/aeternum/v/2798098>
    * <http://kwonnam.pe.kr/wiki/springframework/springboot/jpa>
  * 스프링부트 + JPA 사용시 해당 옵션은 FALSE로 끄고 개발하자.

* Member와 Team을 자주 함께 사용하면?

  * 즉시 로딩 EAGER를 사용해서 함께 조회하면 된다. 
  * 하지만, 현업에서는 전부 LAZY로 걸고 조회하는 시점에 fetchJoin을 통해서 동적으로 한번에 조회하는 방법을 사용한다.

## 프록시와 즉시로딩 주의

* 가급적 지연 로딩을 사용할 것
* 즉시 로딩을 적용하면 곳곳에서 예상하지 못한 SQL이 발생한다.
* 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
* @ManyToOne, @OneToOne은 기본이 **EAGER** 다.
  * **LAZY로 설정하자!**
* @OneToMany, @ManyToMany는 기본이 **LAZY** 이다.