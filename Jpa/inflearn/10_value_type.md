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

## 컬렉션 값 타입(collection value type)