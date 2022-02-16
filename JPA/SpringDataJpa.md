
# Spring Data Jpa


- [Entity를 DTO로 변환](#entity를-dto로-변환)
- [Entity에서 DTO 페치조인](#entity를-dto로-변환---페치-조인-최적화)
- [JPA에서 DTO 바로 조회](#jpa에서-dto로-바로-조회)
- [Auditing](#auditing)
- [Pageable](#pageable)

## API 개발 고급 정리

 - 권장순서
   1. Entity 조회 방식으로 우선 접근
      - 페치조인으로 쿼리수를 최적화
      - 컬렉션 최적화
         - 페이징 필요 `hibernate.default_batch_fetch_size`, `@Batchsize`로 최적화
         - 페이징 필요 X -> 페치 조인 사용
   2. Entity 조회 방식으로 해결이 안되면 DTO로 조회 방식 사용
  

 ## Entity를 DTO로 변환
 ~~~java
 @GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2() {
 List<Order> orders = orderRepository.findAll();
 List<SimpleOrderDto> result = orders.stream()
               .map(o -> new SimpleOrderDto(o))
               .collect(toList());
 return result;
}

@Data
static class SimpleOrderDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 
 public SimpleOrderDto(Order order) {
 orderId = order.getId();
 name = order.getMember().getName();
 orderDate = order.getOrderDate();
 orderStatus = order.getStatus();
 address = order.getDelivery().getAddress();
 }
}
 ~~~

- Entity를 DTO로 변환하는 일반적인 방법
- 쿼리가 총 1 + N + N 번 실행된다
  - `order`조회 1번(order 조회 결과 수가 N이 된다)
  - `order -> member` 지연 로딩 조회 N 번
  - `order -> delivery` 지연 로딩 조회 N 번
  - order 결과가 2개면 최악의 경우 1 + 2 + 2 번 실행된다
    - 지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 날리지 않는다.

## Entity를 DTO로 변환 - 페치 조인 최적화

Controller 코드는 위 와 동일

OrderRepository - 추가 코드
~~~java
public List<Order> findAllWithMemberDelivery() {
 return em.createQuery(
               "select o from Order o" +
               " join fetch o.member m" +
               " join fetch o.delivery d", Order.class)
 .getResultList();
}
~~~
- Entity 페치 조인 을 사용하여 쿼리 1번에 조회
- 페치 조인으로 `order -> member`, `order -> delivery` 는 이미 조회 된 상태이므로 지연로딩 X

## JPA에서 DTO로 바로 조회

OrderQueryRepository - DTO로 조회하는 전용 리포지토리를 생성하는걸 추천
~~~java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {
  private final EntityManager em;
 
 public List<OrderSimpleQueryDto> findOrderDtos() {
    return em.createQuery(
          "select new 
                  jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, 
                  o.orderDate, o.status, d.address)" +
                  " from Order o" +
                  " join o.member m" +
                  " join o.delivery d", OrderSimpleQueryDto.class)
                  .getResultList();
 }
}
~~~

- 일반적인 SQL을 사용할 때 처럼 원하는 값을 선택해서 조회
- `new` 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환
- SELECT 절에서 원하는 데이터를 직접 선택하므로 DB -> 애플리케이션 네트워크 용량 최적화(생각보다 미비)
- Repository 재사용성 떨어짐, API 스펙에 맞춘 코드가 Repository에 들어가는 단점

`정리`

`쿼리 방식 선택 권장 순서`

1. `우선 Entity를 DTO로 변환하는 방법을 선택한다`
2. `필요하면 Fetch Join으로 성능을 최적화 한다. -> 대부분의 성능 이슈가 해결`
3. `그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.`
4. `최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template를 사용해서 SQL을 직접 사용`




# 컬렉션(Collection) 조회 최적화

 ## Entity를 DTO로 변환
 
 Controller
 ~~~java
 @GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2() {
 List<Order> orders = orderRepository.findAll();
 List<OrderDto> result = orders.stream()
 .map(o -> new OrderDto(o))
 .collect(toList());
 return result;
}
 ~~~



 DTO
 
 - Order -> OrderDTO로 매핑해야하며, Order 에 있는 orderItems 도 OrderItemDTO로 받아야 한다.
 - 많은 주니어 개발자들이 조회하는 `Entity`는 DTO로 변환하며 Presentation 쪽으로 반환하지만, `Entity안에 있는 Entity는 DTO로 변환하지 않는데 이 또한 DTO로 변환하며 반환해 주어야 한다.`
~~~java
@Data
static class OrderDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 private List<OrderItemDto> orderItems;
 
         public OrderDto(Order order) {
             orderId = order.getId();
             name = order.getMember().getName();
             orderDate = order.getOrderDate();
             orderStatus = order.getStatus();
             address = order.getDelivery().getAddress();
             orderItems = order.getOrderItems().stream()
                                               .map(orderItem -> new OrderItemDto(orderItem))
                                               .collect(toList());
         }
}

@Data
static class OrderItemDto {
  private String itemName;//상품 명
  private int orderPrice; //주문 가격
  private int count; //주문 수량
  
  public OrderItemDto(OrderItem orderItem) {
      itemName = orderItem.getItem().getName();
      orderPrice = orderItem.getOrderPrice();
      count = orderItem.getCount();
  }
}
~~~

 위 코드의 문제점은 
 
  - 지연 로딩으로 너무 많은 SQL이 실행된다.
  - SQL 실행 수
      - `order` 1번
      - `member`,`address` N번(order 조회 수 만큼)
      - `orderItem` N번(order 조회 수 만큼)
      - `item` N번 (orderItem 조회 수 만큼)


## Entity -> DTO로 변환 (페치 조인)

Repository
~~~java
public List<Order> findAllWithItem() {
 return em.createQuery(
              "select distinct o from Order o" +
              " join fetch o.member m" +
              " join fetch o.delivery d" +
              " join fetch o.orderItems oi" +
              " join fetch oi.item i", Order.class)
       .getResultList();
}

~~~

 - 페치 조인으로 SQL이 1번만 실행됨
 - `1 : N 에서 fetch join 할 시 DB ROW 갯수가 N개 만큼 증가하게 됨` 
   
주의 :  예) Order : orderItems 는 1 : N 관계이다. 만약 Order 가 2건이며, Order 1건 당 2개의 orderItems 를 가진다면. 
   `즉 Order 기준으로 ordetItems 와 fetch join 할 시 2개의 row가 아닌 총 4개의 row가 발생한다.`
 
 - `distinct`를 사용한 이유는 1 : N join이 있으므로 DB ROW 수가 증가한다. 그 결과 Order Entity 수 도 증가한다.
    
    하지만 JPA의 `distinct`는 두 가지 기능이 있다.
    
    1. DB에 distinct를 추가해서 쿼리를 날려준다. 하지만 DB의 `distinct`는 모든 column의 값이 같은 것만 중복을 제거 해준다. 즉 같지 않다면 똑같이 N 개의 ROW가 발생
    2. DB에서 1번과 같은 경우가 발생하지만 JPA에서 `distinct`는 애플리케이션 내부적으로 id 값이 같은 column에 대해 제거를 하고 가져와 준다 


 - 하지만 1 : N 페치 조인의 단점은 `페이징이 불가능` 하다.

    참고 : 컬렉션 페치 조인을 사용하면 `페이징이 불가능` 하다. Hibernate는 경고 로그를 남기면서 모든 데이터를 DB에서 읽어오고, 메모리에
    페이징 해버린다(매우 위험하다. Out of Memory 가 발생할 수 있다)
    
    참고 : 컬렉션 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. 1개를 페치 조인 할 때도 데이터가 뻥튀기
    되기 때문에 2개 이상이면 데이터가 부정합하게 조회 될 수 있다.
    

## Entity를 DTO로 변환 - 페이징과 한계 돌파

- 컬렉션을 페치 조인하면 페이징이 불가능하다.
   - 컬렉션을 페치조인 하면 1:N 조인이 발생하므로 데이터가 예측할 수 없이 증가한다.
   - 1:N에서 일(1)을 기준으로 페이징하는 것이 목적, 그러나 데이터는 다(N)을 기준으로 row가 생성된다.
   - Order를 기준으로 페이징하고 싶은데, 다(N)인 OrderItem을 조인하면 OrderItem이 기준이 된다.
- 이 경우 하이버네이트는 경고 로그를 남기고  모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다. 최악의 경우 장애로 이어질 수 있다.

### 해결책

- 먼저 ToOne(OneToOne, ManyToOne)관계를 모두 페치조인 한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
- 컬렉션은 지연로딩으로 조회한다.
- 지연 로딩 성능 최적화를 위해 `hibernate.default_batch_fetch_size`,`@BatchSize`를 적용한다.
   - hibernate.default_batch_fetch_size : 글로벌 설정
   - @BatchSize : 개별 최적화
   - 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size만큼 IN 쿼리로 조회한다.

Repository
~~~java
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
   return em.createQuery(
              "select o from Order o" +
              " join fetch o.member m" +
              " join fetch o.delivery d", Order.class)
       .setFirstResult(offset)
       .setMaxResults(limit)
       .getResultList();
}
~~~

Controller
~~~java
/**
 * V3.1 엔티티를 조회해서 DTO로 변환 페이징 고려
 * - ToOne 관계만 우선 모두 페치 조인으로 최적화
 * - 컬렉션 관계는 hibernate.default_batch_fetch_size, @BatchSize로 최적화
 */
@GetMapping("/api/v3.1/orders")
public List<OrderDto> ordersV3_page(
 @RequestParam(value = "offset", defaultValue = "0") int offset,
 @RequestParam(value = "limit", defaultValue = "100") int limit){
 
   List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);
   
   List<OrderDto> result = orders.stream().map(o -> new OrderDto(o)).collect(toList());
   
   return result;
 }
~~~

application.yml(글로벌 설정)

![image](https://user-images.githubusercontent.com/79154652/147529436-94debe19-e4db-4cf7-a76b-56aaef7a3efd.png)

- 개별로 설정하려면 `@BatchSize`를 적용하면 된다.(컬렉션은 컬렉션 필드에, Entity는 Entity 클래스에 적용)


- __장점__
  - 쿼리 호출수가 `1 + N` -> `1 + 1`로 최적화 된다.
  - 조인보다 DB 데이터 전송량이 최적화 된다.(Order와 OrderItem을 조인하면 Order가 OrderItem 만큼 중복해서 조회된다. 이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다)
  - 페치 조인 방식과 비교해서 쿼리 호출수가 약간 증가하지만, DB 데이터 전송량이 감소한다.
  - 컬렉션 페치조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.
- 결론

  - ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서 ToOne 관계는 페치조인으로 쿼리수를 줄이고 해결하며, 나머지는 `hibernate.default_batch_fetch_size`로 최적화 하자.

>참고 : `default_batch_fetch_size`의 크기는 적당한 사이즈를 골라야 하는데, 100~1000 사이를 선택하는 것을 권장한다. 이 전략을 SQL IN 절을 사용하는데, DB에 따라 IN 절 파라미터를 1000으로
        제한하기도 한다. 
     
 ## JPA에서 DTO 직접 조회
     
  `Repository`
  
  ~~~java
@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {
   private final EntityManager em;
 /**
 * 컬렉션은 별도로 조회
 * Query: 루트 1번, 컬렉션 N 번
 * 단건 조회에서 많이 사용하는 방식
 */
  
 public List<OrderQueryDto> findOrderQueryDtos() {
  
  //루트 조회(toOne 코드를 모두 한번에 조회)
  List<OrderQueryDto> result = findOrders();
 
 //루프를 돌면서 컬렉션 추가(추가 쿼리 실행)
 result.forEach(o -> {
 List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
         o.setOrderItems(orderItems);
 });
       return result;
 }
 
 /**
 * 1:N 관계(컬렉션)를 제외한 나머지를 한번에 조회
 */
 
 private List<OrderQueryDto> findOrders() {
     return em.createQuery(
          "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
          " from Order o" +
          " join o.member m" +
          " join o.delivery d", OrderQueryDto.class)
            .getResultList();
 }
 
 /**
 * 1:N 관계인 orderItems 조회
 */
 
 private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery(
        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
        " from OrderItem oi" +
        " join oi.item i" +
        " where oi.order.id = : orderId", OrderItemQueryDto.class)
              .setParameter("orderId", orderId)
              .getResultList();
  }
}

  ~~~
  
  `OrderQueryDto`
  
  ~~~java
  @Data
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 private List<OrderItemQueryDto> orderItems;
 
 public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate,
OrderStatus orderStatus, Address address) {
 this.orderId = orderId;
 this.name = name;
 this.orderDate = orderDate;
 this.orderStatus = orderStatus;
 this.address = address;
 }
}
  
  ~~~
  
  `OrderItemQueryDto`
  
  ~~~java
  
@Data
public class OrderItemQueryDto {
 
 @JsonIgnore
 private Long orderId; //주문번호
 private String itemName;//상품 명
 private int orderPrice; //주문 가격
 private int count; //주문 수량
 
 public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
    this.orderId = orderId;
    this.itemName = itemName;
    this.orderPrice = orderPrice;
    this.count = count;
 }
}
  
  ~~~
  
  - Query : 루트 1번, 컬렉션 N번
  - ToOne(N:1, 1:1)관계들을 먼저 조회하고, ToMany(1:N)관계는 각각 별도로 처리한다.
     - 이런 방식을 선택한 이유
        - ToOne 관계는 조인해도 데이터 row 수가 증가하지 않는다.
        - ToMany(1:N) 관계는 조인하면 row 수가 증가한다.
  - row 수가 증가하지 않는 ToOne 관계는 조인으로 최적화하기 쉬우므로 한번에 조회하고, ToMany 관계는 최적화 하기 어려우므로 `findOrderItems()`같은 별도의 메소드로 조회
  

## Auditing

- Entity를 생성 변경할 때 변경한 사람과 시간을 추적하고 싶으면?
  - 등록일
  - 수정일
  - 등록자
  - 수정자
 
 스프링 데이터 JPA 사용
 
 __설정__
 
 `@EnableJpaAuditing` -> 스프링 부트 설정 클래스에 적용해야함
 
 `@EntityListeners(AuditingEntityListener.class)` -> Entity에 적용
 
 __사용 어노테이션__
 
 - `@CreatedDate`
 - `@LastModifiedDate`
 - `@CreateBy`
 - `@LastModifiedBy`

~~~java

@EntityListeners(AuditingEntityListener.class)
@MappedSuperClass
@Getter
public class BaseEntity{

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;


}

~~~

__등록자, 수정자를 처리해주는 `AuditorAware` 스프링 빈 등록__

~~~java

@Bean
public AuditorAware<String> auditorProvider(){
  return () -> Optional.of(UUID.randomUUID().toString());
}
~~~
실무에서는 세션 정보나 스프링 시큐리티 로그인 정보에서 ID를 받음

>참고 : 실무에서 대부분의 Entity는 등록시간, 수정시간이 필요하지만, 등록자, 수정자는 없을 수도 있다. 그래서 다음과 같이 Base 타입을 분리하고 원하는 타입을 선택해서 상속한다.

~~~java
public class BaseTimeEntity{
 
  @CreatedDate
  @Column(updatable = false)
  private LocalDateTime createdDate;
  @LastModifiedDate
  private LocalDateTime lastModifiedDate;
 

}

public class BaseEntity extends BaseTimeEntity{
  @CreatedBy
  @Column(updatable = false)
  private String createdBy;
  @LastModifiedBy
  private String lastModifiedBy;
}

~~~

> 참고 : 저장시점에 등록일, 등록자는 물론이고 수정일, 수정자도 같은 데이터가 저장된다. 데이터가 중복 저장되는 것 같지만, 이렇게 해두면 변경 컬럼만 확인해도 마지막에 업데이트한 유저를 확인 할 수 있으므로 유지보수 관점에서 편리하다. 이렇게 하지 않으면 변경 컬럼이 `null` 일때 등록 컬럼을 또 찾아야 한다. 참고로 저장시점에 저장데이터만 입력하고 싶으면 `@EnableJpaAuditing(modifyOnCreate = false)` 옵션을 사용하면 된다.


## Pageable

  - 설명하기에 앞서 실무에서 가장 중요한 내용 먼저 언급하고 써내려 갈려고 한다.

    `실무에서 가장 중요한 것은 페이지를 유지하면서 Entity를 DTO로 변환하여 Controller에 뿌리는 것이다` -> 이는 Entity가 절대로 노출되서는 안되기 때문이다.
    
    ~~~java
    Page<Member> page = memberRepository.findByAge(10, pageRequest);
    Page<MemberDto> dtoPage = page.map(m -> new MemberDto());
    ~~~
    
     `카운트 쿼리를 분리해야 한다(실무에서는 left join, right join 등을 통해서 데이터를 가져오는데 총페이지 쿼리는 join을 할 필요가없다. 어차피 데이터 개수는 동일하다)`
      
      - 카운트 쿼리에서 성능 저하가 발생하기 때문이다.

   ~~~java
   @Query(value = "select m from Member m", countQuery = "select count(m) from Member m")
   Page<Member> findByMember(Pageable pageble);
   ~~~
  


  - 스프링 데이터 JPA를 사용하면 페이징과 정렬을 말도안되게 간단하게 구현이 가능하다.
  
    - org.springframework.data.domain.Sort : 정렬기능
    - org.springframework.data.domain.Pageable : 페이징 기능(내부에 `sort`포함)

    반환 타입
    
    - org.springframework.data.domain.Page : 추가 count 쿼리 결과를 포함하는 `페이징`
    - org.springframework.data.domain.Slice : 추가 count 쿼리 없이 다음 페이지만 확인 가능 (내부적으로 limit + 1 조회)
    - List : 추가 count 쿼리 없이 결과만 반환

 예제
 
 두번째 파라미터로 받은 `Pageable`은 인터페이스 이다. 즉 실제로 사용할 때는 해당 인터페이스를 구현한 `org.springframework.data.domain.PageRequest` 객체를 사용한다.
 
 `PageRequest` 생성자의 첫번째 파라미터에는 `현재페이지`를, 두번째 파라미터에는 `조회할 데이터 수`를 입력한다. 추가로 정렬 정보도 파라미터로 사용할 수 있다.
 
 참고로 페이지는 `0` 부터 시작한다.
 
 ~~~java 
 @Test
public void page() throws Exception {
       //given
       memberRepository.save(new Member("member1", 10));
       memberRepository.save(new Member("member2", 10));
       memberRepository.save(new Member("member3", 10));
       memberRepository.save(new Member("member4", 10));
       memberRepository.save(new Member("member5", 10));
       
       //when
       PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC,"username"));
       Page<Member> page = memberRepository.findByAge(10, pageRequest);
       
       //then
       List<Member> content = page.getContent(); //조회된 데이터
       assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
       assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수
       assertThat(page.getNumber()).isEqualTo(0); //페이지 번호
       assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호
       assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
       assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
}
 ~~~
 
 
 - 추가로 모바일 처럼 스크롤을 아래로 내리거나 `더보기` 같은 버튼을 클릭해서 보여주는 페이징은 `Slice`를 사용한다.
   별도의 개발필요없이 반환타입만 `Slice`로 변경해주면 된다.
   
   DB에 쿼리를 날릴때 limit + 1 숫자만큼 쿼리를 날린다.
   
   ~~~java
   Slice<Member> findByAge(int age, Pageable pageable);
   
   
   //Slice 로 반환된 객체로 알 수 있는 메소드
    int getNumber(); //현재 페이지
    int getSize(); //페이지 크기
    int getNumberOfElements(); //현재 페이지에 나올 데이터 수
    List<T> getContent(); //조회된 데이터
    boolean hasContent(); //조회된 데이터 존재 여부
    Sort getSort(); //정렬 정보
    boolean isFirst(); //현재 페이지가 첫 페이지 인지 여부
    boolean isLast(); //현재 페이지가 마지막 페이지 인지 여부
    boolean hasNext(); //다음 페이지 여부
    boolean hasPrevious(); //이전 페이지 여부
    Pageable getPageable(); //페이지 요청 정보
    Pageable nextPageable(); //다음 페이지 객체
    Pageable previousPageable();//이전 페이지 객체
   ~~~
