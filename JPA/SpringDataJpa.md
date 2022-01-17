
# Spring Data Jpa


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
