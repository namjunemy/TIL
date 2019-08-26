# 프록시와 연관관계 관리

> 목차
>
> * 프록시
> * 즉시 로딩과 지연 로딩
> * 지연 로딩 활용
> * 영속성 전이: CASCADE
> * 고아 객체
> * 영속성 전이 + 고아 객체, 생명주기

## 프록시

* 질문으로 부터 프록시에 대한 학습을 시작한다.
* Member 엔티티를 조회할 때 Team도 함께 조회해야 할까?
  * 실제로 필요한 비즈니스 로직에 따라 다르다.
  * 비즈니스 로직에서 필요하지 않을 때가 있는데, 항상 Team을 함께 가져와서 사용할 필요는 없다.
  * 낭비가 발생하게 된다.
  * JPA는 이 낭비를 하지 않기 위해, 지연로딩과 프록시라는 개념으로 해결한다.

### 프록시 기초

* 지연 로딩을 이해하려면, 프록시의 개념에 대해서 명확하게 이해해야 한다.

* JPA에서 em.find() 말고, em.getReference()라는 메서드도 제공 된다.

* **em.find()**는 DB를 통해서 **실제 엔티티 객체를 조회**하는 메서드이고

* **em.getReference()**는 DB의 조회를 미루는 **가짜(프록시) 엔티티 객체를 조회하는 메서드**이다.

* Member 엔티티

  ```java
  @Entity
  @Getter
  @Setter
  public class Member extends BaseEntity {
  
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
  
      @Column(name = "name")
      private String username;
  
      private Integer age;
  
      @Enumerated(EnumType.STRING)
      private RoleType roleType;
  
      @Lob
      private String description;
  
      @ManyToOne
      @JoinColumn(name = "team_id")
      private Team team;
  
      @OneToOne
      @JoinColumn(name = "locker_id")
      private Locker locker;
  
      @OneToMany(mappedBy = "member")
      private List<MemberProduct> memberProducts = new ArrayList<>();
  }
  ```

* em.find()로 멤버를 조회하면 아래와 같이 데이터베이스에 쿼리가 바로 나간다.

  ```java
  Member member = new Member();
  member.setCreatedBy("creator");
  
  em.persist(member);
  
  em.flush();
  em.clear();
  
  Member findMember = em.find(Member.class, member.getId());
  System.out.println("findMember.id = " + findMember.getId());
  System.out.println("findMember.username = " + findMember.getUsername());
  
  tx.commit();
  ```

  ```sql
  Hibernate: 
      /* insert hello.jpa.Member
          */ insert 
          into
              Member
              (id, createdBy, createdDate, lastModifiedBy, lastModifiedDate, age, description, locker_id, roleType, name) 
          values
              (null, ?, ?, ?, ?, ?, ?, ?, ?, ?)
  Hibernate: 
      select
          member0_.id as id1_4_0_,
          member0_.createdBy as createdB2_4_0_,
          member0_.createdDate as createdD3_4_0_,
          member0_.lastModifiedBy as lastModi4_4_0_,
          member0_.lastModifiedDate as lastModi5_4_0_,
          member0_.age as age6_4_0_,
          member0_.description as descript7_4_0_,
          member0_.locker_id as locker_10_4_0_,
          member0_.roleType as roleType8_4_0_,
          member0_.team_id as team_id11_4_0_,
          member0_.name as name9_4_0_,
          locker1_.id as id1_3_1_,
          locker1_.name as name2_3_1_,
          team2_.id as id1_8_2_,
          team2_.createdBy as createdB2_8_2_,
          team2_.createdDate as createdD3_8_2_,
          team2_.lastModifiedBy as lastModi4_8_2_,
          team2_.lastModifiedDate as lastModi5_8_2_,
          team2_.name as name6_8_2_ 
      from
          Member member0_ 
      left outer join
          Locker locker1_ 
              on member0_.locker_id=locker1_.id 
      left outer join
          Team team2_ 
              on member0_.team_id=team2_.id 
      where
          member0_.id=?
  findMember.id = 1
  findMember.username = creator
  ```

* 그러나, em.getReference()로 멤버를 조회하면, 실제로 필요한 시점에 데이터베이스에 쿼리가 나간다.

  * 실행결과에서 보면 findMember.username 필드를 출력할 때, DB에서 조회가 필요하므로 그때 쿼리가 나간다.
  * 그리고 findMember.getClass()로 객체를 확인하면 Member객체가 아니라, 하이버네이트가 강제로 만든 가짜 클래스인 HibernateProxy 객체인 것을 볼 수 있다.

  ```java
  Member member = new Member();
  member.setUsername("creator");
  
  em.persist(member);
  
  em.flush();
  em.clear();
  
  Member findMember = em.getReference(Member.class, member.getId());
  System.out.println("findMember = " + findMember.getClass());
  System.out.println("findMember.id = " + findMember.getId());
  System.out.println("findMember.username = " + findMember.getUsername());
  
  tx.commit();
  ```

  ```sql
  Hibernate: 
      /* insert hello.jpa.Member
          */ insert 
          into
              Member
              (id, createdBy, createdDate, lastModifiedBy, lastModifiedDate, age, description, locker_id, roleType, name) 
          values
              (null, ?, ?, ?, ?, ?, ?, ?, ?, ?)
  findMember = class hello.jpa.Member$HibernateProxy$yJgMgbkR
  findMember.id = 1
  Hibernate: 
      select
          member0_.id as id1_4_0_,
          member0_.createdBy as createdB2_4_0_,
          member0_.createdDate as createdD3_4_0_,
          member0_.lastModifiedBy as lastModi4_4_0_,
          member0_.lastModifiedDate as lastModi5_4_0_,
          member0_.age as age6_4_0_,
          member0_.description as descript7_4_0_,
          member0_.locker_id as locker_10_4_0_,
          member0_.roleType as roleType8_4_0_,
          member0_.team_id as team_id11_4_0_,
          member0_.name as name9_4_0_,
          locker1_.id as id1_3_1_,
          locker1_.name as name2_3_1_,
          team2_.id as id1_8_2_,
          team2_.createdBy as createdB2_8_2_,
          team2_.createdDate as createdD3_8_2_,
          team2_.lastModifiedBy as lastModi4_8_2_,
          team2_.lastModifiedDate as lastModi5_8_2_,
          team2_.name as name6_8_2_ 
      from
          Member member0_ 
      left outer join
          Locker locker1_ 
              on member0_.locker_id=locker1_.id 
      left outer join
          Team team2_ 
              on member0_.team_id=team2_.id 
      where
          member0_.id=?
  findMember.username = creator
  ```

### 프록시 특징

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/29_proxy.PNG?raw=true)

* **실제 클래스를 상속** 받아서 만들어진다.
  * 하이버네이트가 내부적으로 상속받아서 만든다. 
* 실제 클래스와 겉 모양이 같다.
* 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 된다.



![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/30_proxy.PNG?raw=true)

* 프록시 객체는 실제 객체의 참조(target)을 보관한다.
* 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.

### 프록시 객체의 초기화

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/31_proxy.PNG?raw=true)

```java
Member member = em.getReference(Member.class, member.getId());
member.getName();
```

1. em.getReference()로 프록시 객체를 가져온 다음에, **getName() 메서드를 호출 ** 하면
2. MemberProxy 객체에 처음에 target 값이 존재하지 않는다. JPA가 영속성 컨텍스트에 초기화 요청을 한다.
3. 영속성 컨텍스트가 DB에서 조회해서
4. 실제 Entity를 생성해준다.
5. 그리고 프록시 객체가 가지고 있는 target(실제 Member)의 getName()을 호출해서 결국 member.getName()을 호출한 결과를 받을 수 있다.
6. 프록시 객체에 target이 할당 되고 나면, 더이상 프록시 객체의 초기화 동작은 없어도 된다.

실제로는 위와 같이 프록시 객체가 동작 한 것이다.

### 프록시 특징 정리

* 프록시 객체는 처음 사용할 때 한 번만 초기화 된다.

* 프록시 객체를 초기화 할 때, 프록시 객체가 실제로 엔티티로 바뀌는 것은 아니다.

  * 초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근 가능하다.
  * 정확히 말하면 target에 값이 채워지는 것 뿐이다.
  * em.getReference()로 조회한 클래스를 getClass()로 보면, HibernateProxy 클래스였던 것을 위에서 학습했다.

* (**심화**)프록시 객체는 원본 엔티티를 상속 받는다고 했다. 프록시 객체와 원본 객체가 타입이 다르다. 타입 체크시 주의해야 한다.

  * == 비교 실패한다. jpa에서 타입 비교는 웬만하면, instanceOf를 사용해야 한다.

    ```java
    Member find = em.find(Member.class, member1.getId());
    Member reference = em.getReference(Member.class, member2.getId());
    System.out.println("m1 == m2 : " + (m1 == m2));
    ```

    ```java
    find == reference : false
    ```

    ```java
    System.out.println("find : " + (find instanceof Member));
    System.out.println("reference : " + (reference instanceof Member));
    ```

    ```java
    find : true
    reference : true
    ```

* (**심화**)영속성 컨텍스트에 찾는 엔티티가 이미 있으면, em.getReference()를 호출해도 실제 엔티티를 반환한다. 반대도 똑같다.

  * 생각을 해보면, 이미 영속성 컨텍스트에 올려논 객체를 굳이 다시 프록시로 감싸서 반환하는게 의미가 없다. 이점이 없다. JPA는 그렇게 하지 않는다.

  * **JPA는 하나의 영속성 컨텍스트에서 조회하는 같은 엔티티의 동일성을 보장한다.**

    * JPA가 기본적으로 제공하는 매커니즘 중 하나이다. 반복가능한 읽기(repeatable read) 제공

  * 따라서, 아래의 코드에서 두 객체는 같다. em.gerReference()로 프록시 객체를 굳이 가져오지 않는다.

    ```java
    Member find = em.find(Member.class, member.getId());
    Member reference = em.getReference(Member.class, member.getId());
    
    System.out.println("find == reference : " + (find == reference)); // true
    ```

  * (**더 심화**) 그렇다면 둘다 getReference() 로 가져오면?!

    * 둘 다, 프록시 객체이다. 근데, **같은 프록시 객체**다. JPA는 한 트랜잭션에서 조회하는 같은 엔티티의 동일성을 보장한다. 프록시 객체도.

      ```java
      Member reference1 = em.getReference(Member.class, member.getId());
      Member reference2 = em.getReference(Member.class, member.getId());
      
      System.out.println("reference1 == reference2 : " + (reference1 == reference2)); // true
      ```

    * 그러면, getReference()로 먼저 가져오고, find()로 실제 객체를 조회하면?

      * 하나는 프록시 객체, 하나는 당연히 find니까 실제 객체이지 않을까?

      * 결론 부터 말하면 **둘 다 같은 프록시 객체를 반환**한다.

      * 한 트랜잭션에서 조회하는 같은 엔티티의 동일성을 보장 하기 위해서.

      * 한 트랜잭션 내에서 reference == find를 true로 반환하기 위해서 이렇게 동작한다.

      * **여기서 가장 중요한 것은, 이렇게 내부적으로 JPA가 복잡하게 다 처리해주지만, 우리가 개발할때는 프록시던 진짜 객체던 중요하지 않다. 그냥 멤버 조회 하면서 개발 하면 된다.**

        ```java
        Member reference = em.getReference(Member.class, member.getId());
        Member find = em.find(Member.class, member.getId());
        
        System.out.println("reference == find : " + (reference == find)); // true
        ```

* 실무에서 많이 만나게 되는 문제

  * 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 초기화 문제가 발생한다.

  * 고로, **트랜잭션의 범위 밖에서 프록시 객체를 조회**하려고 할 때!

    * 하이버네이트는 org.hibernate.LazyInitializationException 예외를 터뜨린다.

    * em.detach(), em.close(), em.clear() 모두 똑같은 예외가 발생한다.

    * 프록시 객체를 초기화 할 수 없다. 더이상 영속성 컨텍스트의 도움을 받지 못한다.

      ```java
      Member member = new Member();
      member.setUsername("creator");
      
      em.persist(member);
      
      em.flush();
      em.clear();
      
      Member reference = em.getReference(Member.class, member.getId());
      
      em.detach(reference);
      //em.close도 동일
      
      System.out.println("findMember.username = " + reference.getUsername());
      
      tx.commit();
      ```

      ```sql
      org.hibernate.LazyInitializationException: could not initialize proxy [hello.jpa.Member#1] - no Session
      	at org.hibernate.proxy.AbstractLazyInitializer.initialize(AbstractLazyInitializer.java:169)
      	at org.hibernate.proxy.AbstractLazyInitializer.getImplementation(AbstractLazyInitializer.java:309)
      	at org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor.intercept(ByteBuddyInterceptor.java:45)
      	at org.hibernate.proxy.ProxyConfiguration$InterceptorDispatcher.intercept(ProxyConfiguration.java:95)
      	at hello.jpa.Member$HibernateProxy$bCLaGKb1.getUsername(Unknown Source)
      	at JpaMain.main(JpaMain.java:35)
      ```

### 프록시 관련 Utils

* 프록시 확인을 도와주는 Util성 메소드들이 있다.

* 프록시 인스턴스의 초기화 여부를 직접 확인

  * PersistenceUnitUtil.isLoaded(Object entity);

    ```java
    // 엔티티 매니저 팩토리로 부터 get
    boolean isLoaded = emf.getPersistenceUnitUtil().isLoaded(referenceMember);
    ```

* 프록시 클래스 확인 방법

  * entity.getClass().getName() 출력 (..javasist.. or HibernateProxy...)

* 프록시 강제 초기화

  * org.hibernate.Hibernate.initialize(entity);

    ```java
    Hibernate.initialize(referenceMamber);
    ```

* 참고로

  * JPA 표준은 강제 초기화 메서드(initialize)가 없다. Hibernate가 지원한다.

  * 그냥 프록시 객체에서 getXXX() 호출해서 강제로 초기화 한다.

    