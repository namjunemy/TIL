# 실전 예제 1 - 요구사항 분석과 기본 매핑

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

## 도메인 모델 분석

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/05_domain_model.png?raw=true)

* 회원과 주문의 관계
    * **회원**은 여러 번 **주문**할 수 있다.(일대다 관계)
* 주문과 상품의 관계
    * **주문**할 때 여러 **상품**을 선택할 수 있다.
    * 반대로 같은 **상품**도 여러 번 **주문**될 수 있다.
    * **주문상품 이라는 모델**을 만들어서 다대다 관계를 일대다, 다대일 관계로 풀어낼 수 있다.

## 테이블 설계

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/06_table.png?raw=true)

## 엔티티 설계와 매핑

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

