
# Querydsl 의 기본문법

  이번 글은 Querydsl의 기본 문법에 대해 글을 적어 볼려고 한다.
  
  먼저 Querydsl과 JPQL을 비교해보겠다.
  
  ## Querydsl vs JPQL
  
  ~~~java
  
  JPAQueryFactory qf;
  
  @Test
  public void startJPQL(){
  
  // JPQL을 사용한 member1 찾기
  
    String qlString = "select m from Member m " +
                      "where m.username = :username";
                      
        Member findMember = em.createQuery(qlString, Member.class)
         .setParameter("username", "member1")
         .getSingleResult();
         
         assertThat(findMember.getUsername()).isEqualTo("member1");
  
  }
  
  @Test
  public void startQuerydsl(){
  
  //Querydsl 사용한 member1 찾기
    JPAQueryFactory qf = new JPAQueryFactory(em);
    
    Memeber findMember =qf
                          .select(Qmember.member)
                          .from(Qmember.member)
                          .where(Qmember.member.name.eq("member1") //파라미터 바인딩 처리
                          .fetchOne();
    
    assertThat(findMember.getUsername()).isEqualTo("member"1);
  
  }
  
  ~~~
  
  - `EntityManager` 로 `JPAQueryFactory` 생성
  - Querydsl 은 JPQL 빌더
  - JPQL: 문자(실행 시점 오류), Querydsl: 코드(컴파일 시점 오류)
  - JPQL: 파라미터 바인딩 직접, Querydsl: 파라미터 바인딩 자동 처리
  
  
  - JPAQueryFactory 를 필드로 설정 할 수도 있다.
  
  > JPQQueryFactory를 필드로 제공하면 동시성 문제는 어떻게 될까? 동시성 문제는 JPAQueryFactory를 생성할 때 제공하는 EntiryManager(em)에 달려있다. 스프링 프레임워크는 여러 쓰레드에서 동시에 같은 EntityManager에 접근해도 
    트랙재션 마다 별도의 영속성 컨텍스트를 제공하기 때문에, 동시성 문제는 걱정하지 않아도 된다.
  
  
  ## 기본  Q-Type 활용
      
    Q클래스 인스턴스를 사용하는 2가지 방법
    
    ~~~java
    
    QMember qMember = new QMember("m"); //별칭 직접 설정
    QMember qMember = QMember.member; //기본 인스턴스 사용
    
    ~~~
    
    > 참고 : 같은 테이블을 조인해야하는 경우가 아니면 기본 인스턴스를 사용하자.
 
 ## 검색 조건 쿼리
 
 ~~~java
 
 @Test
 public void search(){
    
  Member findMember =  qf
    .selectFrom(member)
    .where(member.username.eq("member1")
    .and(member.age.eq(10))
    .fetchOne();
 
 }
 
 ~~~
  
  - 검색 조건은 `.and()`, `.or()`를 메서드 체인으로 연결할 수 있다.

### JPQL이 제공하는 모든 검색 조건 
~~~java

member.username.eq("member1") // eq -> username = "member1" > equal
member.username.ne("member1") // ne -> username != "member1"  > not equal
member.username.eq("member1").not() // 

member.username.isNotNull(); // 이름이 is not null

member.age.in(10,20) // age in (10,20)
member.age.notIn(10,20) // age not in(10, 20)
member.age.between(10,30) // between 10, 30


member.age.goe(30) // age >= 30 크거나 같다.
member.age.gt(30) // age > 30 크다.
member.age.loe(30) // age <= 30 작거나 같다.
member.age.lt(30) // age < 30 작다.

member.username.like("member%") //like 검색
member.username.contains("member") // like '%member%' 검색
member.username.startsWith("member") // like 'member%' 검색
~~~


### And 조건을 파라미터로 처리
~~~java
@Test
public void searchAndParam() {
 List<Member> result1 = queryFactory
                             .selectFrom(member)
                             .where(member.username.eq("member1"),
                                    member.age.eq(10))
                             .fetch();
 assertThat(result1.size()).isEqualTo(1);
}


~~~

- `where()`에 파라미터로 검색조건을 추가하면 `AND` 조건이 추가됨
- 이 경우 `null` 값은 무시 -> 메서드 추출을 활용해서 동적 쿼리를 깔끔하게 만들 수 있음.

### 결과조회

- `fetch()` : 리스트 조회, 데이터 없으면 빈 리스트 반환
- `fetchOne()` : 단 건 조회
      
    - 결과가 없으면 : `null`
    
    - 결과가 둘 이상이면:`com.querydsl.core.NonUniqueResultException`

- `fetchFirst()` : limit(1).fetchOne()
- `fetchResults()` : 페이징 정보 포함, total count 쿼리 추가 실행
- `fetchCount()` : count 쿼리로 변경해서 count 수 


~~~java

//페이징에서 샤용

QueryResults<Member> results = 
                                qf
                                .selectFrom(member)
                                .fetchResults();
                                
//Count 쿼리로 변경
long count = qf
              .selectFrom(member)
              .fetchCount();

~~~

### 정렬

~~~java

@Test
public void sort(){
    
    JPAQueryFactory qf = new JPAQueryFactory(em);
    
    List<Member> result = qf
                          .selectFrom(member)
                          .where(member.age.eq(10))
                          .orderBy(member.age.desc(), member.username.asc().nullsLast())
                          .fetch();
}

~~~

- `desc()`, `asc()` : 일반 정렬
- `nullsLast()`, `nullFirst()` : null 데이터 순서 부여


### 페이징

~~~java
@Test
public void paging(){

  List<Member> result =  qf
                          .selectFrom(member)
                          .orderBy(member.username.desc())
                          .offset(1) // 0부터 시작된다 원래
                          .limit(2) // 최대 2개 조회
                          .fetch();
   
}

//전체 조회수가 필요하면 fetchResults

@Test
public void paging2() {
 QueryResults<Member> queryResults = queryFactory
                                       .selectFrom(member)
                                       .orderBy(member.username.desc())
                                       .offset(1)
                                       .limit(2)
                                       .fetchResults();


~~~

> 참고 : 실무에서 페이징 쿼리를 작성할 때, 데이터를 조회하는 쿼리는 여러 테이블을 조인해야 하지만, count 쿼리는 조인이 필요 없는 경우도 있다.
        그런데 이렇게 자동화된 count 쿼리는 원본 쿼리와 같이 모두 조인을 해버리기 때문에 성능이 안나올 수 있다. count 쿼리에 조인이 필요없는 성능 최적화가 필요하다면,
        count 전용 쿼리를 별도로 작성해야 한다.


### 기본집합 및 GroupBy

~~~java
@Test
public void aggregation(){
  List<Tuple> result = queryFactory
                           .select(member.count(),
                           member.age.sum(),
                           member.age.avg(),
                           member.age.max(),
                           member.age.min())
                           .from(member)
                           .fetch();

}
    Tuple tuple = result.get(0);
    
    assertThat(tuple.get(member.count())).isEqualTo(4);
   assertThat(tuple.get(member.age.sum())).isEqualTo(100);
   assertThat(tuple.get(member.age.avg())).isEqualTo(25);
   assertThat(tuple.get(member.age.max())).isEqualTo(40);
   assertThat(tuple.get(member.age.min())).isEqualTo(10);
   
   
@Test
public void group(){
      List<Tuple> result = qf
                          .select(team.name, member.age.avg())
                          .from(member)
                          .join(member.team, team)
                          .groupBy(team.name)
                          .fetch();
      
      Tuple teamA = result.get(0);
      Tuple teamB = result.get(1);

// 그룹화된 결과를 제한하려면 having

.groupBy(item.price)
.having(item.price.gt(1000))

}
~~~

- JPQL이 제공하는 모든 집합 함수를 제공한다.
- tuple 은 프로젝션과 결과반환에서 설명한다.
