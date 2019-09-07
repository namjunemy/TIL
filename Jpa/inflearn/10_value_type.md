# 값 타입

> * 기본값 타입
> * **임베디드 타입(복합 값 타입)**
> * 값 타입과 불변 객체
> * 값 타입의 비교
> * **값 타입 컬렉션**

## JPA의 데이터 타입 분류

### 엔티티 타입

* @Entity로 정의하는 객체
* 데이터가 변해도 식별자로 지속해서 추적 가능
* ex) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능

### 값 타입

* int, Integer, String 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
* 식별자가 없고 값만 있으므로 변경시 추적 불가
* ex) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체함
* 분류
  * 기본값 타입
  * 임베디드 타입
  * 컬렉션 값 타입

## 기본값 타입

 * 종류
   * 자바 기본 타입(int, double)
   * 래퍼 클래스(Integer, Long)
   * String
* 생명 주기를 엔티티에 의존한다.
  * ex) 회원을 삭제하면 이름, 나이 필드도 함께 삭제한다.
* 값 타입은 공유하면 절대 안된다.
  * ex) 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안된다. Side Effect 발생 하면 안됨.
* 참고
  * 자바의 기본 타입은 절대 공유 되지 않는다.
  * int, double 같은 기본 타입은 절대 공유 되지 않는다.
  * 기본 타입은 공유되지 않고 항상 값을 복사한다.
  * **Integer 같은 래퍼 클래스**나 **String** 같은 특수한 클래스는 공유 가능한 객체이지만, 변경할 수 없는 **Immutable** class 이다.

## 임베디드 타입(emvedded type, 복합 값 타입)

* 새로운 값 타입을 직접 정의할 수 있다
* JPA는 임베디드 타입(embedded type)이라 함
* 주로 기본 값 타입을 모아서 만들기 때문에 복합 값 타입이라고도 함
* **주의할 점** : int, String과 같은 **값 타입** 이다. 엔티티 아니다. 변경하면 끝난다.

### 이해하기

* 회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편번호를 가진다.

  * id, name, startDate, endDate, city, street, zipcode
  * 요소들을 보면, 묶어서 공통으로 쓰일 수 있을 것 같은 필드군 들이 보인다.
  * 근무 시작일과 종료, 그리고 주소의 도시, 번지, 우편번호는 묶어서 시스템에서 공통으로 쓸 수 있다.

* 실제 생활에서 회원 정보를 얘기할 때 저 필드들을 다 나열하면서 얘기하나? 

* 실제로는 회원은 이름, 근무기간, 집주소 가지고 있어요 라고 추상화해서 쉽게 표현한다.

* 위의 표현을 아래와 같이 묶어서 임베디드 타입으로 구현 할 수 있다.

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/34_embedded.PNG?raw=true)

### 임베디드 타입 사용법

* @Embeddable
  * 값 타입을 정의 하는 곳에 표시
* @Embedded
  * 값 타입을 사용하는 곳에 표시
* 기본 생성자 필수.

### 임베디드 타입의 장점

* 재사용이 가능하다
* 클래스 내의 높은 응집도
* Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있다.
* 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존한다.

### 임베디드 타입과 테이블 매핑

* 실제로 임베디드 타입을 사용하면서 DB 테이블은 따로 바뀔것이 없다.

* DB는 데이터를 잘 관리하는 것이 목적이기 때문에, 한 통에 다 담기는게 맞다.

* 근데, 객체는 데이터 뿐만아니라 행위 까지 다 가지고 있기 때문에, 공통 요소를 묶어서 임베디드 타입으로 들고 있으면 가져갈 수 있는 이점이 많다.

  * Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드 등

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/35_embedded.PNG?raw=true)

* 임베디드 타입은 엔티티의 값일 뿐이다.

* 임베디드 타입을 사용하기 전과 후에 매핑 테이블은 같다.

* 객체와 테이블을 아주 **세밀하게(find-grained) 매핑하는 것이 가능** 하다.

* **잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다.**

* [임베디드 타입 연습 코드](https://github.com/namjunemy/orm-jpa-basic/commit/943af2fa8ce84bf029344cc130ec4f51e951224e)

### 임베디드 타입과 연관관계

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/36_embedded.PNG?raw=true)

* 위의 그림에서 멤버는 임베디드 값 타입으로 Address와 PhoneNumber를 가진다.
* Address는 Zipcode라는 임베디드 타입을 가지고 있다.(임베디드 타입이 임베디드 타입을 가질 수 있다.)
* PhoneNumber라는 임베디드 타입이. PhoneEntity라는 Entity를 가질 수 있다. id만 가지고 있으면 된다.

### @AttributeOverride: 속성 재정의

* Member에서 Address를 중복으로 가지고 있으면 어떻게 될까?

  * 컬럼 명이 중복되서 MappingException: Repeated column ~~~ 에러 난다.

  ```java
  @Entity
  public class Member {
    ...
    @Embedded
    private Address homeAddress;
    
    @Embedded
    private Address workAddress;
    ...
  }
  ```

* 이 경우에 @AttributeOverride를 사용하면 된다.

  * 아래와 같이 새로운 컬럼에 저장하도록 컬럼 명 속성을 재정의 하면 된다.

  ```java
  @Entity
  public class Member {
    ...
    @Embedded
    private Address homeAddress;
    
    @Embedded
    @AttributeOverrides({
      @AttributeOverride(name="city", column=@Column(name = "WORK_CITY"),
      @AttributeOverride(name="street", column=@Column(name = "WORK_street"),
      @AttributeOverride(name="zipcode", column=@Column(name = "WORK_ZIPCODE")            
    })
    private Address workAddress;
    ...
  }
  ```

### 임베디드 타입과 null

* 임베디드 타입의 값이 null이라면, 매핑한 컬럼 값은 모두 null로 저장 된다.

## 값 타입과 불변 객체

* 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

### 값 타입 공유 참조

* 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 굉장히 위험하다.

* 사이드 이펙트가 발생한다.

* 코드로 이해하기

    * 멤버 클래스

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
        
            @ManyToOne(fetch = FetchType.EAGER)
            @JoinColumn(name = "team_id", insertable = false, updatable = false)
            private Team team;
        
            @OneToOne
            @JoinColumn(name = "locker_id")
            private Locker locker;
        
            @OneToMany(mappedBy = "member")
            private List<MemberProduct> memberProducts = new ArrayList<>();
        
            @Embedded
            private Period workPeriod;
        
            @Embedded
            private Address homeAddress;
        
            public void changeTeam(Team team) {
                this.team = team;
                this.team.getMembers().add(this);
            }
        }
        ```

    * 멤버가 같은 주소 객체를 가지고 있다고 해보자.

    * 중간에 1번 멤버의 주소 객체를 가져와서 city 속성을 바꿔보자.

        ```java
        Address address = new Address("city", "street", "10000");
        
        Member member = new Member();
        member.setUsername("nj");
        member.setHomeAddress(address);
        em.persist(member);
        
        Member member2 = new Member();
        member2.setUsername("kim");
        member2.setHomeAddress(address);
        em.persist(member2);
        
        member.getHomeAddress().setCity("new city");
        
        tx.commit();
        ```

    * 실행 결과를 보면 UPDATE 쿼리가 두번 날라간다. DB를 확인해봐도 member 1과 2의 주소 값이 다 바뀌었다.

    * 이런 사이드 이펙트는 진짜 잡기가 어렵다.

    * 엔티티 간에 공유하고 싶은 값 타입은 엔티티로 만들어서 공유해야 한다.

### 값 타입 복사

* 위에서 확인한 것 처럼 값 타입의 실제인스턴스인 값을 공유하는 것은 매우  위험한 행위이다. 사이드 이펙트 추적도 어렵다.

* 대신 값(인스턴스)를 복사해서 사용한다.

    ```java
    Address address = new Address("city", "street", "10000");
    
    Member member = new Member();
    member.setUsername("nj");
    member.setHomeAddress(address);
    em.persist(member);
    
    //주소 객체 복사
    Address copyAddress = new Address(address.getCity(), address.getStreet(), address.getZipcode());
    
    Member member2 = new Member();
    member2.setUsername("kim");
    member2.setHomeAddress(copyAddress);
    em.persist(member2);
    
    member.getHomeAddress().setCity("new city");
    ```

### 객체 타입의 한계

* 위의 예처럼 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.

* 그런데 문제는, 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본타입이 아니라 객체 타입이다.

* 자바의 기본타입에 값을 대입하면 자바는 기본적으로 값을 복사한다.

* 근데 객체 타입은 참조 값을 직접 대입하는 것을 박을 방법이 없다. 왜냐면 객체의 타입이 같으면 그냥 다 대입해서 넣을 수 있다. 위의 예처럼 Address 타입에 copyAdress를 넣어야 하는데 개발자가 실수로 address를 넣었다. 피할 방법이 없다. 컴파일에서 잡지 못한다.

* 따라서, 객체의 공유 참조는 피할 수 없다.

* 기본 타입의 값 복사

    ```java
    int a = 10;
    int b = a; // 기본 타입은 값을 복사
    b = 4; // b는 4
    ```

* 객체 타입

    ```java
    Address a = new Address("old");
    Address b = a; // 객체 타입은 참조를 전달한다.
    b.setCity("new"); // a 주소 객체의 필드 city 가 변경 된다.
    ```

### 불변 객체(Immutable Object)

* 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단할 수 있다.
* 값 타입은 불변 객체(immutable object)로 설계해야 한다.
* 불변 객체 : 생성 시점 이후 절대 값을 변경할 수 없는 객체
* 생성자로만 값을 설정하고 Setter를 만들지 않으면 된다.
* 참 : Integer, String은 자바가 제공하는 대표적인 불변 객체이다.

* 위의 실습에서도 Address 임베디드 타입 객체에 setter를 제거하자.

* 불변이라는 작은 제약으로 사이드 이펙트라는 큰 재앙을 막을 수 있다.

* 그러면 원래 값 객체의 필드를 실제로 바꾸고 싶은 경우는?

    * 아래와 같이 값 객체를 통으로 교체 한다.

    ```java
    Address address = new Address("city", "street", "10000");
    
    Member member = new Member();
    member.setUsername("nj");
    member.setHomeAddress(address);
    em.persist(member);
    
    Address newAddress = new Address("new City", address.getStreet(), address.getZipcode());
    member.setHomeAddress(newAddress);
    
    tx.commit();
    ```

* 결론 : 값 타입은 불변으로 만들자.

## 값 타입의 비교

* 깂 타입은 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야 한다.

    ```java
    int a = 10;
    int b = 10;
    ```

* 그런데, 객체 타입은 값이 같아도 주소가 달라서 == 비교하면 false로 나온다.

    ```java
    Address a = new Address("서울");
    Address b = new Address("서울");
    ```

* 동일성(identity) 비교

    * 인스턴스의 참조 값을 비교한다. == 사용

* 동등성(equivalence) 비교

    * 인스턴스의 값을 비교, equals() 사용

* 값 타입은 a.equals(b)를 사용해서 동등성 비교를 해야 한다.

* 값 타입의 equals() 메소드를 적절하게 재정의해야 한다.(주로 모든 필드를 다 재정의 해야 함) + 해시코드 재정의도.

## 값 타입 컬렉션(collection value type)

* 값 타입을 컬렉션에 담아서 쓰는 것을 말한다.

* 연관관계 매핑에서 엔티티를 컬렉션으로 가지고 있는 것이 아니라

* 아래와 같이 좋아하는 음식이나, 주소변경 히스토리를 저장하는 값 타입을 컬렉션에 저장하는 경우를 말한다.

    ```java
    public class Member {
        @Id
        private Long id;
        
        private Set<String> favoriteFoods;
        private List<Address> addressHistory;
    }
    ```

* 근데, 이 관계를 DB테이블에 저장하려고 할때 문제가 생긴다. DB는 컬렉션을 저장하는 개념이 없다.

* 값 타입 컬렉션은 값 타입을 하나 이상 저장할 때 사용한다.

* @ElementCollection, @CollectionTable 어노테이션을 사용한다.

* 따라서, 컬렉션을 저장하기 위한 별도의 테이블이 필요하다.

* 일대다 관계로 풀어서 별도의 테이블로 만들어 내야 한다. 조인이 가능하도록.

    ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/37_collection_value_type.png?raw=true)

* 코드 실습

    * Member 클래스에 값 타입 컬렉션 추가

    * addressHistory는 임베디드 타입이므로 컬럼명들을 그대로 쓰면 되는데,

    * Set\<String\> favoriteFoods는 예외적으로 컬럼명을 지정해준다. 

        ```java
        @Entity
        @Getter
        @Setter
        public class Member extends BaseEntity {
        
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;
        
            ...
        
            @ElementCollection
            @CollectionTable(
                name = "favorite_food",
                joinColumns = @JoinColumn(name = "member_id"))
            @Column(name = "food_name")
            private Set<String> favoriteFoods = new HashSet<>();
        
            @ElementCollection
            @CollectionTable(
                name = "address",
                joinColumns = @JoinColumn(name = "member_id"))
            private List<Address> addressHistory = new ArrayList<>();
        
            ...
        }
        ```

        ```sql
        == favorite 테이블 생성 로그, FK 설정(member id로)
        
        Hibernate: 
            
            create table favorite_food (
               member_id bigint not null,
                food_name varchar(255)
            )
                
        Hibernate: 
            
            alter table favorite_food 
               add constraint FKetdr9m6ysgxkvs2f7vf15sac4 
               foreign key (member_id) 
               references Member
        ```

        ```sql
        == address 테이블 생성 로그, FK 설정
        
        Hibernate: 
            
            create table address (
               member_id bigint not null,
                city varchar(255),
                street varchar(255),
                zipcode varchar(255)
            )
        
        Hibernate: 
            
            alter table address 
               add constraint FK6ncq4527mw0seu2b7wr19mqpg 
               foreign key (member_id) 
               references Member
        ```

### 컬렉션 값 타입 저장

* 실제로 컬렉션 값 타입이 어떻게 사용 되는지 보자.

    * 컬렉션 값 타입에 값들을 넣고, persist하면

    ```java
    Member member = new Member();
    member.setUsername("nj");
    member.setHomeAddress(new Address("city", "street", "10000"));
    
    member.getFavoriteFoods().add("치킨");
    member.getFavoriteFoods().add("족발");
    member.getFavoriteFoods().add("피자");
    
    member.getAddressHistory().add(new Address("city1", "street1", "10001"));
    member.getAddressHistory().add(new Address("city2", "street2", "10002"));
    member.getAddressHistory().add(new Address("city3", "street3", "10003"));
    
    em.persist(member);
    
    tx.commit();
    ```

    * 아래의 내용으로 쿼리가 나간다.

        * 먼저 Member가 저장되고,
        * 값 타입 컬렉션을 저장하는 테이블에 Insert 쿼리가 6개 나간다.
        * 어? 연관관계 설정이 되어있지 않은 다른 테이블에 insert 쿼리가 나갔네?
        * Member의 라이프사이클에 같이 적용이 됐다.
        * 값 타입이기 때문에 그렇다. Member에 소속된 값 타입들은 생명주기가 Member를 따라간다.
        * 테이블만 따로 만들어서 관리하는 것 뿐이지, 별도로 persist하거나 update하거나 할 필요 없이 값을 바꾸고 Member를 저장하면 된다.
            * 일대다 연관관계에서 Cascade ALL로 설정하고, orphanRemoval true 설정한 것과 유사하다.
        * 값 타입 컬렉션은 영속성 전이(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.

        ```sql
        ibernate: 
            /* insert hello.jpa.Member
                */ insert 
                into
                    Member
                    (id, createdBy, createdDate, lastModifiedBy, lastModifiedDate, age, description, city, street, zipcode, locker_id, roleType, name, endDate, startDate) 
                values
                    (null, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.addressHistory */ 
                insert 
                into
                    address
                    (member_id, city, street, zipcode) 
                values
                    (?, ?, ?, ?)
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.addressHistory */ 
                insert 
                into
                    address
                    (member_id, city, street, zipcode) 
                values
                    (?, ?, ?, ?)
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.addressHistory */ 
                insert 
                into
                    address
                    (member_id, city, street, zipcode) 
                values
                    (?, ?, ?, ?)
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.favoriteFoods */ 
                insert 
                into
                    favorite_food
                    (member_id, food_name) 
                values
                    (?, ?)
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.favoriteFoods */ 
                insert 
                into
                    favorite_food
                    (member_id, food_name) 
                values
                    (?, ?)
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.favoriteFoods */ 
                insert 
                into
                    favorite_food
                    (member_id, food_name) 
                values
                    (?, ?)
        ```


### 컬렉션 값 타입 조회

* em.flush(), clear()로 영속성 컨텍스트를 비워 놓고. Member를 조회해보자.
* Member에 소속된 Embedded 타입의 city, street, zipcode는 같이 조회 되지만,
* 컬렉션들은 조회 되지 않는다. ! **컬렉션 값 타입들은 지연로딩 전략을 취한다.**

```java
Member member = new Member();
member.setUsername("nj");
member.setHomeAddress(new Address("city", "street", "10000"));

member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("족발");
member.getFavoriteFoods().add("피자");

member.getAddressHistory().add(new Address("city1", "street1", "10001"));
member.getAddressHistory().add(new Address("city2", "street2", "10002"));
member.getAddressHistory().add(new Address("city3", "street3", "10003"));

em.persist(member);

em.flush();
em.clear();

System.out.println("================== START");
em.find(Member.class, member.getId());
```

```sql
================== START
Hibernate: 
    select
        member0_.id as id1_7_0_,
        member0_.createdBy as createdB2_7_0_,
        member0_.createdDate as createdD3_7_0_,
        member0_.lastModifiedBy as lastModi4_7_0_,
        member0_.lastModifiedDate as lastModi5_7_0_,
        member0_.age as age6_7_0_,
        member0_.description as descript7_7_0_,
        member0_.city as city8_7_0_,
        member0_.street as street9_7_0_,
        member0_.zipcode as zipcode10_7_0_,
        member0_.locker_id as locker_15_7_0_,
        member0_.roleType as roleTyp11_7_0_,
        member0_.team_id as team_id16_7_0_,
        member0_.name as name12_7_0_,
        member0_.endDate as endDate13_7_0_,
        member0_.startDate as startDa14_7_0_,
        locker1_.id as id1_6_1_,
        locker1_.name as name2_6_1_ 
    from
        Member member0_ 
    left outer join
        Locker locker1_ 
            on member0_.locker_id=locker1_.id 
    where
        member0_.id=?
```

* 직접 컬렉션 값 타입을 터치 해보자.

    * Lazy 로딩으로 동작한다.
    * @ElementCollection() 어노테이션의 fetch 기본값이 LAZY로 잡혀있다.

    ```java
    System.out.println("================== START");
    em.find(Member.class, member.getId());
    Member findMember = em.find(Member.class, member.getId());
    
    List<Address> addressHistory = findMember.getAddressHistory();
    for (Address address : addressHistory) {
        System.out.println("address = " + address.getCity());
    }
    
    Set<String> favoriteFoods = findMember.getFavoriteFoods();
    for (String favoriteFood : favoriteFoods) {
        System.out.println("favoriteFood = " + favoriteFood);
    }
    
    tx.commit();
    ```

    ```sql
    ================== START
    Hibernate: 
        select
            member0_.id as id1_7_0_,
            member0_.createdBy as createdB2_7_0_,
            member0_.createdDate as createdD3_7_0_,
            member0_.lastModifiedBy as lastModi4_7_0_,
            member0_.lastModifiedDate as lastModi5_7_0_,
            member0_.age as age6_7_0_,
            member0_.description as descript7_7_0_,
            member0_.city as city8_7_0_,
            member0_.street as street9_7_0_,
            member0_.zipcode as zipcode10_7_0_,
            member0_.locker_id as locker_15_7_0_,
            member0_.roleType as roleTyp11_7_0_,
            member0_.team_id as team_id16_7_0_,
            member0_.name as name12_7_0_,
            member0_.endDate as endDate13_7_0_,
            member0_.startDate as startDa14_7_0_,
            locker1_.id as id1_6_1_,
            locker1_.name as name2_6_1_ 
        from
            Member member0_ 
        left outer join
            Locker locker1_ 
                on member0_.locker_id=locker1_.id 
        where
            member0_.id=?
    Hibernate: 
        select
            addresshis0_.member_id as member_i1_0_0_,
            addresshis0_.city as city2_0_0_,
            addresshis0_.street as street3_0_0_,
            addresshis0_.zipcode as zipcode4_0_0_ 
        from
            address addresshis0_ 
        where
            addresshis0_.member_id=?
    address = city1
    address = city2
    address = city3
    Hibernate: 
        select
            favoritefo0_.member_id as member_i1_4_0_,
            favoritefo0_.food_name as food_nam2_4_0_ 
        from
            favorite_food favoritefo0_ 
        where
            favoritefo0_.member_id=?
    favoriteFood = 족발
    favoriteFood = 치킨
    favoriteFood = 피자
    ```

### 컬렉션 값 타입 수정

* 이전에 학습 했지만, 값 타입은 Immutable 해야 한다. 절대 불변해야 한다.

* 따라서, 수정하고 싶다면 embedded 클래스인 Address 인스턴스 자체를 new 로 갈아 끼워야 한다.

    ```java
    System.out.println("================== START");
    Member findMember = em.find(Member.class, member.getId());
    
    findMember.setHomeAddress(new Address("newCity", "newStreet", "20001"));
    
    tx.commit();
    ```

* 컬렉션 값 타입 수정1 - String

    *  Set\<String\> 타입인 좋아하는 음식을 수정하려면, String 자체가 불변 객체 이므로,

    * 삭제하고, 다시 리스트에 넣어준다.

        ```java
        System.out.println("================== START");
        Member findMember = em.find(Member.class, member.getId());
        
        findMember.getFavoriteFoods().remove("치킨");
        findMember.getFavoriteFoods().add("한식");
        
        tx.commit();
        ```

    * 발생한 쿼리를 확인해보자.

        * 멤버를 SELECT 하고,
        * 지연로딩으로 favoriteFood를 SELECT 하고,
        * favoriteFood에서 치킨을 DELETE하고
        * favoriteFood에 한식을 INSERT한다.
        * 값 타입의 변경을 캐치해서 JPA에서 알아서 삭제, 추가 쿼리를 실행한다.
        * 영속성 전이가 되는 것 처럼 동작한다. 값 타입 컬렉션들은 소속 엔티티에 의존 관계를 다 맡긴다.

        ```sql
        ================== START
        Hibernate: 
            select
                member0_.id as id1_7_0_,
                member0_.createdBy as createdB2_7_0_,
                member0_.createdDate as createdD3_7_0_,
                member0_.lastModifiedBy as lastModi4_7_0_,
                member0_.lastModifiedDate as lastModi5_7_0_,
                member0_.age as age6_7_0_,
                member0_.description as descript7_7_0_,
                member0_.city as city8_7_0_,
                member0_.street as street9_7_0_,
                member0_.zipcode as zipcode10_7_0_,
                member0_.locker_id as locker_15_7_0_,
                member0_.roleType as roleTyp11_7_0_,
                member0_.team_id as team_id16_7_0_,
                member0_.name as name12_7_0_,
                member0_.endDate as endDate13_7_0_,
                member0_.startDate as startDa14_7_0_,
                locker1_.id as id1_6_1_,
                locker1_.name as name2_6_1_ 
            from
                Member member0_ 
            left outer join
                Locker locker1_ 
                    on member0_.locker_id=locker1_.id 
            where
                member0_.id=?
                
        Hibernate: 
            select
                favoritefo0_.member_id as member_i1_4_0_,
                favoritefo0_.food_name as food_nam2_4_0_ 
            from
                favorite_food favoritefo0_ 
            where
                favoritefo0_.member_id=?
        
        Hibernate: 
            /* delete collection row hello.jpa.Member.favoriteFoods */ delete 
                from
                    favorite_food 
                where
                    member_id=? 
                    and food_name=?
                    
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.favoriteFoods */ 
                insert 
                into
                    favorite_food
                    (member_id, food_name) 
                values
                    (?, ?)
        ```

* 컬렉션 값 타입 수정2 - Address 객체

    * List\<Address\> 타입의 AddressHistory의 값을 바꿔보자.

        * 이때, equals, hashCode가 잘 재정의 되어있어야 한다.

        ```java
        System.out.println("================== START");
        Member findMember = em.find(Member.class, member.getId());
        
        findMember.getAddressHistory().remove(new Address("city1", "street1", "10001"));
        findMember.getAddressHistory().add(new Address("new city1", "street1", "10001"));
        
        
        tx.commit();
        ```

    * 실행된 쿼리를 보면,

        * Member SELECT,
        * 지연 로딩으로 AddressHistory SELECT
        * AddressHistory 테이블인 Address 테이블중 Member 소속인 row들 전체 DELETE. 
            * ???? 테이블을 통으로 날린다?
        * 그리고, Address 3개를 새로 INSERT 한다.
        * 이 부분에 대해서 아래 제약사항에서 자세히 알아보자.

        ```sql
        ================== START
        Hibernate: 
            select
                member0_.id as id1_7_0_,
                member0_.createdBy as createdB2_7_0_,
                member0_.createdDate as createdD3_7_0_,
                member0_.lastModifiedBy as lastModi4_7_0_,
                member0_.lastModifiedDate as lastModi5_7_0_,
                member0_.age as age6_7_0_,
                member0_.description as descript7_7_0_,
                member0_.city as city8_7_0_,
                member0_.street as street9_7_0_,
                member0_.zipcode as zipcode10_7_0_,
                member0_.locker_id as locker_15_7_0_,
                member0_.roleType as roleTyp11_7_0_,
                member0_.team_id as team_id16_7_0_,
                member0_.name as name12_7_0_,
                member0_.endDate as endDate13_7_0_,
                member0_.startDate as startDa14_7_0_,
                locker1_.id as id1_6_1_,
                locker1_.name as name2_6_1_ 
            from
                Member member0_ 
            left outer join
                Locker locker1_ 
                    on member0_.locker_id=locker1_.id 
            where
                member0_.id=?
        Hibernate: 
            select
                addresshis0_.member_id as member_i1_0_0_,
                addresshis0_.city as city2_0_0_,
                addresshis0_.street as street3_0_0_,
                addresshis0_.zipcode as zipcode4_0_0_ 
            from
                address addresshis0_ 
            where
                addresshis0_.member_id=?
                
        Hibernate: 
            /* delete collection hello.jpa.Member.addressHistory */ 
            delete 
                from
                    address 
                where
                    member_id=?
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.addressHistory */ insert 
                into
                    address
                    (member_id, city, street, zipcode) 
                values
                    (?, ?, ?, ?)
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.addressHistory */ insert 
                into
                    address
                    (member_id, city, street, zipcode) 
                values
                    (?, ?, ?, ?)
        Hibernate: 
            /* insert collection
                row hello.jpa.Member.addressHistory */ insert 
                into
                    address
                    (member_id, city, street, zipcode) 
                values
                    (?, ?, ?, ?)
        ```

### 값 타입 컬렉션의 제약사항

* 값 타입 컬렉션은 값 타입은 엔티티와 다르게 식별자 개념이 없다.

* 값은 변경하면 추척이 어렵다.

    * 위에서 컬렉션 값 타입에 대한 테이블 - ADDRESS 테이블을 생성하는 쿼리를 보면

    * ID가 따로 존재하지 않는다. 만약 이 row에서 city 값이 변경 됐을 경우

    * 어떤 row의 값을 변경해야 하는지 추적할 수 있는 방법이 없다.

        ```sql
        Hibernate: 
            
            create table address (
               member_id bigint not null,
                city varchar(255),
                street varchar(255),
                zipcode varchar(255)
            )
        ```

* **그래서 값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.**

    * 다른 어노테이션 설정을 추가해서, 풀어낼 수도 있지만,
    * 이렇게 복잡하게 사용해야하면 쓰지 않는 것이 낫다. 다른 방법으로 풀어내는 것이 맞다.

* 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다.

    * NULL 입력 X, 중복 저장X 조건을 만족 시키도록.

### 값 타입 컬렉션 대안

* **실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려하는 것이 낫다.**

* **일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용하자.**

* 영속성 전이 + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용하자

* 코드로 이해하기

    * Member 클래스에서 AddressHistory 엔티티로 대체

        * @OneToMany와 @JoinColumn 으로 일대다 단방향 매핑을 한다. 
        * AddressEntity에서 내부적으 Address 값 타입을 포함한다.

        ```java
        @Entity
        @Getter
        @Setter
        public class Member extends BaseEntity {
        
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;
        
            ...
        
            @ElementCollection
            @CollectionTable(
                name = "favorite_food",
                joinColumns = @JoinColumn(name = "member_id"))
            @Column(name = "food_name")
            private Set<String> favoriteFoods = new HashSet<>();
        
            @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
            @JoinColumn(name = "member_id")
            private List<AddressEntity> addressHistory = new ArrayList<>();
        }
        ```

        ```java
        @Entity
        @Getter
        @Setter
        @Table(name = "ADDRESS")
        public class AddressEntity {
        
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;
        
            private Address address;
        
            public AddressEntity() {
            }
        
            public AddressEntity(String city, String street, String zipcode) {
                this.address = new Address(city, street, zipcode);
            }
        }
        
        ```

    * 사용 예시

        * AddressEntity를 넣는다.
        * @OneToMany와 @JoinColumn 으로 일대다 단방향 매핑을 했으니까,
        * 키를 관리하는 Member 테이블의 반대인  AddressEntity(Address 테이블)에 Update 쿼리가 나가는 것은 이미 학습했다. [참조](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/07_various_relational_mapping.md#%EC%9D%BC%EB%8C%80%EB%8B%A4-1n)
        * 이제는 ADDRESS 테이블에 ID라는 개념이 생겼다. 그리고 Member ID를 FK로 가지고 있다.
        * 자체적인 아이디가 있다는 것은 엔티티라는 얘기고, 이제는 특정 row를 찾아서 내부의 값을 수정할 수 있다.
        * 이런식으로 값 타입 컬렉션은 엔티티로 승격 시켜서 많이 사용한다.

        ```java
        Member member = new Member();
        member.setUsername("nj");
        member.setHomeAddress(new Address("city", "street", "10000"));
        
        member.getAddressHistory().add(new AddressEntity("city1", "street1", "10001"));
        member.getAddressHistory().add(new AddressEntity("city2", "street2", "10002"));
        member.getAddressHistory().add(new AddressEntity("city3", "street3", "10003"));
        
        em.persist(member);
        
        em.flush();
        em.clear();
        ```

        ```sql
        Hibernate: 
            /* insert hello.jpa.Member
                */ insert 
                into
                    Member
                    (id, createdBy, createdDate, lastModifiedBy, lastModifiedDate, age, description, city, street, zipcode, locker_id, roleType, name, endDate, startDate) 
                values
                    (null, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        Hibernate: 
            /* insert hello.jpa.AddressEntity
                */ insert 
                into
                    ADDRESS
                    (id, city, street, zipcode) 
                values
                    (null, ?, ?, ?)
        Hibernate: 
            /* insert hello.jpa.AddressEntity
                */ insert 
                into
                    ADDRESS
                    (id, city, street, zipcode) 
                values
                    (null, ?, ?, ?)
        Hibernate: 
            /* insert hello.jpa.AddressEntity
                */ insert 
                into
                    ADDRESS
                    (id, city, street, zipcode) 
                values
                    (null, ?, ?, ?)
        Hibernate: 
            /* create one-to-many row hello.jpa.Member.addressHistory */ update
                ADDRESS 
            set
                member_id=? 
            where
                id=?
        Hibernate: 
            /* create one-to-many row hello.jpa.Member.addressHistory */ update
                ADDRESS 
            set
                member_id=? 
            where
                id=?
        Hibernate: 
            /* create one-to-many row hello.jpa.Member.addressHistory */ update
                ADDRESS 
            set
                member_id=? 
            where
                id=?
        ```

### 정리

* 엔티티 타입의 특징
    * 식별자(ID)가 있다.
    * 생명 주기를 관리 한다.
    * 공유 가능하다
* 값 타입의 특징
    * 식별자가 없다.
    * 생명 주기를 엔티티에 의존한다
    * 공유하지 않는 것이 안전하다.(복사해서 사용)
    * 불변 객체로 만드는 것이 안전하다.
* 값 타입은 정말 값 타입이라 판단될 때만 사용하자.
* 엔티티와 값 타입을 혼동해서 엔티티를 값 타입으로 만들면 안된다.
* 식별자가 필요하고, 지속해서 값을 추적, 변경해야 한다면 그것은 값 타입이 아닌 엔티티 이다.

### Reference

- [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)