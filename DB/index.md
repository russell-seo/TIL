

# Index 란?

  내가 찾고 싶은 데이터를 찾을 때, 모든 레코드에서 찾는 것 보다 특정한 범위 내 한정해서 데이터를 찾는게 빠르다.
  
  즉, 우리가 책에다 `포스트잇`을 붙여 넣거나 목차를 달아두어서 찾고자 하는 내용이 있으면 해당 내용이 속한 `포스트잇`이나 목차를 바로 찾아서
  둘러 보는 것과 같다. DB에서 `레인지 스캔`을 하려면 책처럼 색인(목차)이 필요하다. 이러한 색인, 포스트잇을 `DB에서 인덱스(Index)`라고 부른다.
  
  예를 들어 사용자가 100만명이 있는 테이블에서 userid 가 akdu39 라는 값을 찾고자 한다면, 다음 이 쿼리를 사용해야 한다.
  
  ~~~SQL
  select userid from user_table where userid = "akdu39"
  ~~~
  
  만약 인덱스가 없다면 100만개의 데이터를 모두 찾아보고 akdu39 값과 매칭되는 결과를 찾아야 합니다. 10개, 100개 정도되는 데이터의 양이라면
  상관 없을 수 있지만 100만개 라는 방대한 데이터에서 모든 값들을 하나하나 찾는 건 속도에 큰 영향을 준다.
  
  이때, userid 값 기준으로 index를 만들어주면 userid가 a,b,c순으로 정렬되어 있기 때문에 aku~ 부분에서 문자열을 바로 찾을 수 있다.
  `만약 해당하는 문자열이 없다면 b로 넘어가기 전에 탐색이 종료되기 때문에 전체탐색보다 빠르다.`
  
  > `한 테이블에서 여러개 index 설정이 가능하다. 다만 인덱스가 너무 많으면 새로운 Row를 추가할 때 마다 인덱스 값 또한 추가해주어야 되서
   한 테이블에 index는 최대 3~4개 정도가 적당하다.`
   
   <p align = "center">
   <img src = "https://t1.daumcdn.net/cfile/tistory/99240D3359FEEDA92A" width = "700" height = "400" />
   </p>
   
 - 디스크에서 읽는 것은 메모리에서 읽는 것 보다 성능이 훨씬 떨어진다.
    - 결국 인덱스 성능을 향상시킨다는 것은 `디스크 저장소에 얼마나 덜 접근하게 만드느냐`, `인덱스 Root 에서 Leaf 까지 오가는 횟수를 얼마나 줄이느냐` 에 달려있다. 
  
  
  
  ## 1. Index를 만들 때 어떤 기준으로 만드는게 좋은가?
  
  1. 크기가 큰 테이블만 만든다.
      - 크기가 작은 테이블에는 인덱스나 풀 스캔이나 큰 차이가 없다.
  2. 기본키 제약이나 유일성 제약이 부여된 열에는 필요가 없다.
      - PK가 부여된 열에는 자동으로 Index가 작성되어 있고, 유일성 제약이 붙어 있는 컬럼 또한 같다.
      - 이 2가지 제약이 붙은 열에 암묵적으로 Index가 작성된 이유는 값의 중복 체크를 하려면 데이터를 정렬해야 하는데 Index를 사용하면 편리하다
  3. Cardinality(카디널리티)가 높은 열에 만든다
       
       - `값의 분산도를 뜻한다`
       - 카디널리티란 전체 행에 대한 특정 컬럼의 중복 수치를 나타내는 지표
       
         (예를 들어 주민등록번호 같은 경우 중복 값이 없으므로 카디널리티가 높다, 단 이름은 주민등록번호에 비해 중복된 이름이 많다, 즉 카디널리티가 낮다고 말할 수 있다.)
         
  ## 2. 인덱스 컬럼 기준 (개인적으로 DB 성능에 가장 도움이 많이 되었다)
  > 먼저 1개의 컬러만 인덱스를 걸어야 한다면, 해당 컬럼은 `카디널리티`가 가장 높은 것을 잡아야 한다.
    
    카디널리티란 해당 컬럼의 값의 분산도 라고 말한다.
    
    즉 성별, 학년등은 카디널리티가 낮다고 말한다(값의 분산도가 낮다 -> 성별(남,여) 학년(1~6학년))
    
    반대로 주민등록번호, 계좌번호등은 카디널리티가 높다고 말한다(값의 분산도가 높다 -> 중복되는 주민등록번호는 존재하지 않음)
    
   - 인덱스로 최대한 효율을 뽑아내려면, `해당 인덱스로 많은 부분을 걸러내야 하기 때문`이다. 만약 성별을 인덱스로 잡으면 남/여중 하나를 선택하기 때문에 인덱스를 통해 50% 밖에 걸러내지 못한다.
     
     하지만 주민등록번호나 계좌번호 같은 경우엔 `인덱스를 통해 데이터의 대부분을 걸러내기 때문에` 빠르게 검색이 가능하다.
     
  ### 2.1 여러 컬럼으로 인덱스 구성시 기준 (개인적으로 DB 성능에 가장 도움이 많이 되었다)
   
   ![image](https://user-images.githubusercontent.com/79154652/148349112-cb2e448b-3d9c-4572-a2c9-e998000c5db2.png)

   - 인덱스를 2가지 형태로 생성하겠습니다.
   ~~~java
   CREATE INDEX IDX_SALARIES_INCREASE ON salaries (is_bonus, from_date, is_bonus);
    
    
   CREATE INDEX IDX_SALARIES_DECREASE ON salaries (group_no, from_date, is_bonus); 
   ~~~
   
   첫번째 인덱스는 `카디널리티가 낮은순에서 높은순`
   
   두번째 인덱스는 `카디널리티가 높은순에서 낮은순`
    
   월등한 차이는 나지 않지만 `카디널리티가 높은순에서 낮은순`으로 구성하는데 성능이 더 뛰어나다.
   
   ### 2.2 여러 컬럼으로 인덱스시 조건 누락
   
   꼭 인덱스의 컬럼을 모두 사용해야만 하는 것은 아니다.
   
   예를 들어 `group_no, from_date, is_bonus` 구성의 인덱스가 있다.
   
   여기서 만약 `group_no` 를 제외한 쿼리를 실행한다면 __전혀 인덱스를 타지 못한다.__
   
   하지만 인덱스 구성의 첫번째 조건인 `group_no`를 `조회조건에 포함되어야만 인덱스를 태울 수 있다.`
   
   > 결론
      - 첫번째 인덱스 컬럼이 조회 쿼리에 없으면 인덱스를 타지 않는다.
    
         
  ## 3. 인덱스 조회시 주의 사항
  
   - `between`, `like`, `<` , `>` 등 범위 조건은 해당 컬럼은 Index를 타지만, 그 뒤 Index 컬럼들은 Index가 사용되지 않습니다.
      - 즉, 범위조건으로 사용하면 안된다고 기억하면 좀 더 쉽다.
   
   - 반대로 `=`, `in` 은 다음 컬럼도 인덱스를 사용합니다.
        - `in`은 결국 `=`를 여러번 실행 시킨 것 이기 때문이다.
        - 단, `in`은 인자값으로 상수가 포함되면 문제없지만, `서브쿼리를 넣게되면 성능상 이슈가 발생한다.`
        - `in`의 인자로 `서브쿼리가 들어가면 서브쿼리의 외부가 먼저 실행`되고 `in`은 체크조건으로 실행되기 때문이다.
   
   - `AND` 연산자는 각 조건들이 읽어와야할 ROW수를 줄이는 역할을 하지만, `or`연산자는 비교해야할 ROW가 더 늘어나기 때문에 Full Scan이 발생할 확률이 높다.
        - `where`에서 `or`을 사용할 때는 주의가 필요하다.
   
   - 인덱스로 사용된 `컬럼값 그대로 사용해야만 인덱스가 사용` 된다.
      
     - 인덱스는 가공된 데이터를 저장하고 있지 않는다.
     - `where salary * 10 > 150000;`는 인덱스를 못타지만, `where salary > 150000 / 10;` 는 인덱스를 사용한다.
     - 컬럼이 문자열인데 숫자로 조회하면 `타입이 달라 인덱스가 사용되지 않습니다` 정확한 타입을 사용해야만 한다.

   - `null`값의 경우 is null 조건으로 인덱스 레인지 스캔 가능

      
  ## 4. B-Tree
  
  B-Tree의 핵심은 데이터가 정렬된 상태로 유지되어 있다는 것
  
  ![이미지](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqycZ2%2FbtqBQnr4QYG%2F7J8KpnmNaJiTjgS0K9TEIK%2Fimg.png)
  
  그림에 표시된 사각형으로 표시된 한개 한개의 데이터를 `노드(Node)` 라고 한다.
  
  가장 상단의 노드를 `루트 노드(Root Node)`, 중간 노드들을 `브랜치 노드(Branch Node)`, 가장 아래 노드들을 `리프 노드(Leaf Node)`라고 한다.
  
  ### 4.1 B+Tree
  
  인덱스 구조는 실제로 B-Tree 구조로 구성되어있다.
  
  B+Tree는 B-Tree의 확장개념으로, B-Tree의 경우, internal 또는 Branch 노드에 key,data를 담을수 있다. 하지만 `B+Tree`같은 경우 Branch 노드에 key만 담아두고 `data`는 담지 않는다. 
  `오직 leaf노드에만 key와 data를 저장하고 leaf노드 끼리 Linkedlist로 연결되어 있다.`
  
  
  ![이미지](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbRiL19%2FbtqBTMSBCWF%2FJ3nKw2qympUVxGThnVdLK0%2Fimg.png)
 
 
  - B+Tree의 장점
    
    - leaf 노드를 제외하고 데이터를 담아두지 않기 때문에 `메모리를 더 확보` 함으로써 더많은 key들을 수용할 수 있다.
    - 하나의 노드에 더 많은 key들을 담을 수 있기에 트리의 높이는 더 낮아진다(cache hit을 높일 수 있음)
    - 풀 스캔 시, B+Tree는 leaf 노드에 데이터가 모두 있기 때문에 한번의 선형탐색만 하면 되기에 B-Tree에 비해 빠르다, B-Tree는 모든 노드를 확인 해야 한다.
  
  
  ## 참조
  
 [https://jojoldu.tistory.com/243](https://jojoldu.tistory.com/243)