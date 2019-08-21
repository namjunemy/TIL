## 실전 예제 1 - 요구사항 분석과 기본 매핑

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

### 도메인 모델 분석

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/05_domain_model.png?raw=true)

* 회원과 주문의 관계
    * **회원**은 여러 번 **주문**할 수 있다.(일대다 관계)
* 주문과 상품의 관계
    * **주문**할 때 여러 **상품**을 선택할 수 있다.
    * 반대로 같은 **상품**도 여러 번 **주문**될 수 있다.
    * **주문상품 이라는 모델**을 만들어서 다대다 관계를 일대다, 다대일 관계로 풀어낼 수 있다.

### 테이블 설계

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/06_table.png?raw=true)

### 엔티티 설계와 매핑

* 테이블 설계에 맞춰서 아래와 같이 엔티티를 설계하고 매핑 한다.
* 이렇게 해보고, 실제로 문제점이 뭔지 파악하자

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/07_entity.PNG?raw=true)

* [엔티티 매핑 커밋 로그](https://github.com/namjunemy/orm-jpa-basic/commit/b538f7f12bf97112491816dfdf43c6a4b53e342e)

* 컬럼의 제약조건들을 엔티티 필드에 선언해 놓으면 굳이 DB 테이블 상세조회 하지 않아도 된다.

* 인덱스들도 엔티티에 @Table(indexex = ) 속성으로 추가해 놓으면 나중에 객체를 중심으로 JPQL 쿼리 작성할 때 도움이 된다.

  * 인덱스를 실제로 거는게 아니고, 쿼리짤 때 참조 가능.

* 위와 같이 엔티티를 만들고 주문 정보를 조회해서 주문한 회원을 참조하려고 할때, 아래와 같이 코드를 짤 수 밖에 없다.

  * `order`  객체에서 `getMember()` 를 통해서 멤버를 참조할 수 있어야 객체지향스럽게 코드를 작성할 수 있다.

  ```JAVA
  Order order = em.find(Order.class, 1L);
  Long memberId = order.getMemberId();
  
  Member member = em.find(Member.class, memberId);
  // order.getMember();
  ```

### 데이터 중심 설계의 문제점

* 위의 방식은 객체 설계를 테이블 설계에 맞춘 방식이다.
* 테이블의 외래키를 객체에 그대로 저장한다.
* 객체 그래프의 탐색이 불가능하고, 참조가 없으므로 UML도 사실 잘못 된것이다.
* 이 문제를 해결하고자 이제 연관관계 매핑을 배운다.

## 실전 예제 2 - 연관관계 매핑 시작

### 테이블 구조

* 테이블 구조는 이전과 동일하다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/06_table.png?raw=true)

### 객체 구조

* 참조를 사용하도록 변경한다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/10_ex2_object_model.png?raw=true)

### 매핑

* 테이블 구조에서 FK가 있는 일대다에서 다쪽에서 단방향 매핑을 우선적으로 한다. 그리고 나서 필요하면 양방향 매핑을 진행한다.

* MEMBER와 ORDERS 1:N 관계에서 단방향 매핑

    * 연관관계 매핑은 끝.

    ```java
    @Entity
    @Table(name = "orders")
    @Getter
    @Setter
    public class Order {
    
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        @Column(name = "order_id")
        private Long id;
        
        // 외래키 식별자 그대로 매핑한 것 제거
        //@Column(name = "member_id")
        //private Long memberId;
    
        @ManyToOne
        @JoinColumn(name = "member_id")
        private Member member;
    
        private LocalDateTime orderDateTime;
    
        @Enumerated(EnumType.STRING)
        private OrderStatus status;
    }
    ```

* OEDERS와 ORDER_ITEM, ITEM의 1:N 관계 단방향 매핑

    ```java
    @Entity
    @Getter
    @Setter
    public class OrderItem {
    
        @Id
        @GeneratedValue
        @Column(name = "order_item_id")
        private Long id;
    
        // 외래키 식별자 제거
        //@Column(name = "order_id")
        //private Long orderId;
        
        @ManyToOne
        @JoinColumn(name = "order_id")
        private Order order;
    
        // 외래키 식별자 제거
        //@Column(name = "item_id")
        //private Long itemId;
    
        @ManyToOne
        @JoinColumn(name = "item_id")
        private Item item;
        
        private int orderPrice;
    
        private int count;
    }
    ```

* ITEM 엔티티 입장에서는 역방향으로 참조를 가져갈 상황이 없을 것으로 판단, 보통은 주문 -> 주문_상품 테이블을 통해서 상품을 조회함. 따라서 양방향 매핑은 가져가지 않고, 단방향에서 매핑을 끝냄.

* 이제 단방향 매핑이 끝났으므로, 필요한 경우를 판단해서 양방향 매핑 진행

* MEMBER와 ORDERS 1:N 관계의 경우 실제로는 MEMBER에서 OrderList를 가지고 있는 것은 좋은 설계가 아니다. 

    * 예를 들어, 특정 회원의 주문내역을 보고싶은 경우
    * MEMBER가 가지고 있는 orderList를 get해서 다시 ORDERS 테이블을 조회해서 주문내역을 찾는 것은 잘못된 설계이다. 아무리 객체지향 입장에서 본다고 하지만, 연관관계의 관심사를 잘 끊어 내는것도 중요하다. 복잡해진다. 회원은 거의 회원 정보만 가지고 있으면 된다.
    * 주문내역이 필요하다면, ORDERS 테이블에서 MEMBER_ID를 FK로 가지고 있기 때문에 조회 시작점을 ORDERS로 두면 된다.

* ORDERS와 ORDER_ITEM의 1:N 관계에서 양방향 매핑 설정

    * 회원의 주문서를 뽑았는데, 주문에 관련된 상품목록들은 자주 조회 될 가능성이 높으므로, 비즈니스 적으로 의미가 있다. 양방향 매핑 설정 해주자.

    ```java
    @Entity
    @Table(name = "orders")
    @Getter
    @Setter
    public class Order {
    
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        @Column(name = "order_id")
        private Long id;
    
        @ManyToOne
        @JoinColumn(name = "member_id")
        private Member member;
    
        // 양방향 매핑 설정
        @OneToMany(mappedBy = "order")
        private List<OrderItem> orderItems = new ArrayList<>();
    
        private LocalDateTime orderDateTime;
    
        @Enumerated(EnumType.STRING)
        private OrderStatus status;
    }
    ```

    * 양방향 연관관계의 편의 메소드 생성

        * Order와 OrderItem의 연관관계 편의 메소드 생성해서 양방향 매핑 관계를 원자적으로 관리.

        * 편의 메소드 생성은 고려를 통해서 어느쪽에 만들지 선택하기 나름이다.

            ```java
            @Entity
            @Table(name = "orders")
            @Getter
            @Setter
            public class Order {
            
                @Id
                @GeneratedValue(strategy = GenerationType.IDENTITY)
                @Column(name = "order_id")
                private Long id;
            
                @ManyToOne
                @JoinColumn(name = "member_id")
                private Member member;
            
                @OneToMany(mappedBy = "order")
                private List<OrderItem> orderItems = new ArrayList<>();
            
                // 연관관계 편의 메서드
                public void addOrderItem(OrderItem orderItem) {
                    this.orderItems.add(orderItem);
                    orderItem.setOrder(this);
                }
            
                private LocalDateTime orderDateTime;
            
                @Enumerated(EnumType.STRING)
                private OrderStatus status;
            }
            ```

            ```java
            // 주문 객체를 만들고
            Order order = new Order();
            
            // 주문 상품을 쭉쭉 넣어줄 수 있다.
            order.addOrderItem(new OrderItem());
            ```

* 단방향 매핑을 쭉 하고나서, 양방향 매핑을 필요할 때 해주면 된다. 하지만, 실전에서 경험을 해보면 조회시 JPQL 쿼리를 많이 짜게 되는데, 그래서 양방향 연관관계를 많이 걸게 된다.

## 실전 예제 3 - 다양한 연관관계 매핑

### 엔티티 - 배송, 카테고리 추가

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/19_add_entity.png?raw=true)

* 주문과 배송은 1:1(@OneToOne)
* 상품과 카테고리는 N:M(@ManyToMany)

### ERD - 배송, 카테고리 추가

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/20_add_erd.png?raw=true)

### 추가된 엔티티 상세

* 실습에서는 JPA가 지원하는 다대다 매핑을 경험해보자.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/20_add_erd.png?raw=true)

### 구현 코드

* Order와 Delivery 일대일 @OneToOne 매핑

    * Order - 연관관계의 주인

        ```java
        @Entity
        @Table(name = "orders")
        public class Order {
        
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            @Column(name = "order_id")
            private Long id;
        
            @ManyToOne
            @JoinColumn(name = "member_id")
            private Member member;
        
            // Delivery와 1:1 매핑. 연관관계의 주인은 Order.
            @OneToOne
            @JoinColumn(name = "delivery_id")
            private Delivery delivery;
        
            @OneToMany(mappedBy = "order")
            private List<OrderItem> orderItems = new ArrayList<>();
        
            public void addOrderItem(OrderItem orderItem) {
                this.orderItems.add(orderItem);
                orderItem.setOrder(this);
            }
        
            private LocalDateTime orderDateTime;
        
            @Enumerated(EnumType.STRING)
            private OrderStatus status;
        }
        ```

    * Delivery - mappedBy 설정

        ```java
        @Entity
        public class Delivery {
        
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;
        
            private String city;
        
            private String street;
        
            private String zipcode;
        
            private DeliveryStatus status;
        
            @OneToOne(mappedBy = "delivery")
            private Order order;
        }
        ```

* Category와 Item 다대다 @ManyToMany 매핑

    * Category

        * 상위, 하위 카테고리 구현시 엔티티 셀프 참조로 연관관계 매핑이 가능하다.
        * joinColumns과 inverseJoinColumns 명시

        ```java
        @Entity
        public class Category {
        
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;
        
            private String name;
        
            // 상위 카테고리를 엔티티 셀프로 매핑할 수 있다.
            @ManyToOne
            @JoinColumn(name = "parent_id")
            private Category parent;
        
            // 하위 카테고리 매핑
            @OneToMany(mappedBy = "parent")
            private List<Category> child = new ArrayList<>();
        
            // 다대다 관계 매핑, joinColumns과 inverseJoinColumns 명시
            @ManyToMany
            @JoinTable(
                name = "category_item",
                joinColumns = @JoinColumn(name = "category_id"),
                inverseJoinColumns = @JoinColumn(name = "item_id")
            )
            private List<Item> items = new ArrayList<>();
        }
        ```

    * Item - mappedBy 설정

        ```java
        @Entity
        public class Item {
        
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            @Column(name = "item_id")
            private Long id;
        
            private String name;
        
            private int price;
        
            private int stockQuantity;
        
            @ManyToMany(mappedBy = "items")
            private List<Category> categories = new ArrayList<>();
        }
        ```

    * 실제 발생한 조인테이블 생성 SQL

        ```sql
        ...
        
        Hibernate: 
            
            create table category_item (
               category_id bigint not null,
                item_id bigint not null
            )
            
        ...
        ```

### 다대다 관계는 1:N, N:1로

* 실습에서는 ManyToMany를 경험했지만, 실전에서는 조인 테이블을 이용해서 1:N, N:1로 풀어내자.
* 실전에서는 중간 테이블이 단순하지 않다.
* @ManyToMany의 제약은
    * 중간 테이블에 필드를 추가하지 못한다는 것
    * 엔티티와 테이블이 불일치하는 문제가 생긴다는 것
* 실전에서는 @ManyToMany는 사용하지 말자

### @JoinColumn

* 외래 키를 매핑할 때 사용한다.

| 속성                                                         | 설명                                                         | 기본값                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| name                                                         | 매핑할 외래 키 이름                                          | 필드명 + _ + 참조하는 테이블의 기본 키 컬럼명 |
| referencedColumnName                                         | 외래 키가 참조하는 대상 테이블의 컬럼명                      | 참조하는 테이블의 기본키 컬럼명               |
| foreignKey(DDL)                                              | 외래 키 제약조건을 직접 지정할 수 있다. 이 속성은 테이블을 생성할 때만 사용한다. |                                               |
| unuque<br />nullable<br />insertable<br />updatable<br />columnDefinition<br />table | @Column의 속성과 같다.                                       |                                               |

### @ManyToOne의 주요 속성

* 다대일 관계 매핑

| 속성         | 설명                                                         | 기본값                                                    |
| ------------ | ------------------------------------------------------------ | --------------------------------------------------------- |
| optional     | false로 설정하면 연관된 엔티티가 항상 있어야 한다.           | TRUE                                                      |
| fetch        | 글로벌 페치 전략을 설정한다.                                 | @ManyToOne=FetchType.EAGER<br />@OneToMany=FetchType.LAZY |
| cascade      | 영속성 전이 기능을 사용한다.                                 |                                                           |
| targetEntity | 연관된 엔티티의 타입 정보를 설정한다. 이 기능은 거의 사용하지 않는다. 컬렉션을 사용해도 제네릭으로 타입 정보를 알 수 있다. |                                                           |

### @OneToMany의 주요 속성

* 일대다 관계 매핑
* **일대다는 mappedBy가 존재하는데, 다대일관계에서는 없다. 그 말은 다대일을 사용하면 연관관계의 주인이 되어야 한다는 것이다.**

| 속성         | 설명                                                         | 기본값                                                    |
| ------------ | ------------------------------------------------------------ | --------------------------------------------------------- |
| mappedBy     | 연관관계의 주인 필드를 선택한다.                             |                                                           |
| fetch        | 글로벌 페치 전략을 설정한다.                                 | @ManyToOne=FetchType.EAGER<br />@OneToMany=FetchType.LAZY |
| cascade      | 영속성 전이 기능을 사용한다.                                 |                                                           |
| targetEntity | 연관된 엔티티의 타입 정보를 설정한다. 이 기능은 거의 사용하지 않는다. 컬렉션을 사용해도 제네릭으로 타입 정보를 알 수 있다. |                                                           |

