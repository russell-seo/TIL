# DB LOCK

 ## LOCK 이란?
   - 트랜잭션 처리의 순서를 보장하기 위한 방법
   - DB가 처리하는 가장 작은 단위
   - 트랜잭션이 완벽하게 처리될 때 까지 다른 트랜잭션 개입을 막아주는 방식


 ## LOCK 의 종류

   - `Shared Lock` : S-LOCK 이라고 보통 얘기한다.
     - Row-Level Lock 중 하나이다.
     - 데이터 Read 에 대한 Lock
     - S-LOCK을 사용하는 쿼리 끼리는 같은 row에 접근이 가능하다.
   - `Exclusive Lock` : X-LOCK 이라고 한다.
     - Row-Level Lock 중 하나
     - 데이터 Write 에 대한 Lock
     - 트랜잭션 완료 될때 까지 유지되며 Lock이 해제 될 때 까지 다른 트랜잭션은 해당 리소스에 접근 불가능
     - SELECT...FOR UPDATE, INSERT등 수정 쿼리를 날릴 때 ROW 에 걸리는 LOCK
  > S-LOCK, X-LOCK 경쟁 여부
  > 1. 여러 트랜잭션은 한 ROW에 S-LOCK을 걸 수 있다.
  > 2. S-LOCK이 걸려있는 ROW에 다른 트랜잭션의 X-LOCK은 걸 수 없다.
>   3. X-LOCK 이 걸려있는 ROW에 다른 트랜잭션의 X-LOCK, S-LOCK 걸수 없다, 다른 트랜잭션이 수정/삭제 하고 있는 ROW는 읽기, 수정, 삭제 모두 불가능  

  
   - `Intention Lock`
       - Table-Level Lock
       - Row에 대해서 나중에 어떤 Row-Level Lock을 걸지 미리 Table-Level 에 걸어두는 Lock
       - `SELECT....FOR IN SHARE MODE` 가 실행되면 Intention Shared Lock이 테이블에 걸린다. Row-Level 에 S-LOCK 걸림
       - `SELECT....FOR UPDATE, INSERT, DELETE` 가 실행되면 Intention Exclusive Lock 이 테이블에 걸린다, Row-Level 에 X-LOCK 걸림
       - Intention Lock 은 여러 트랜잭션에서 접근 가능
   - `Row-Level Lock` : InnoDB Storage engine 은 기본적으로 Row-Level Lock 사용
   - `Record Lock`
     - DB Index Record 에 걸리는 LOCK
     - 개별 Index Record 에 S-LOCK or X-LOCK 설정
     - PK, UK로 조회해서 하나의 레코드에 LOCK을 거는 방법

   ### Gap Lock

   Gap Lock 은 실제 레코드에 대한 잠금이 아닌 레코드와 인접한 사이의 간격을 잠그는 LOCK 이다.

   여기서 말하는 Gap 은 인덱스 중 실제 DB에 레코드가 없는 ROW를 말한다.

   DB에서 직접 알아보자. lock 이라는 테이블이 존재하고 이 테이블에는 id, name 컬럼이 존재한다.

   <img width="227" alt="스크린샷 2023-11-08 오후 7 40 20" src="https://github.com/russell-seo/TIL/assets/79154652/df52bec1-178d-4b1e-a5bd-b4e75f416a51">

   위와 같이 DB 에 데이터가 들어가 있다고 생각하자.

   존재하지 않는 id=3 인 row의 데이터를 업데이트 하는 쿼리를 날려보자.
   ~~~mysql
   mysql > begin; //트랜잭션 시작

   mysql > update side.`lock` set name = 'newbe' where id = 3;
   ~~~

   ~~~mysql
   mysql > select * from performance_schema.data_locks;
   ~~~

   <img width="1236" alt="스크린샷 2023-11-08 오후 7 44 27" src="https://github.com/russell-seo/TIL/assets/79154652/f8002ca7-ab56-450a-8805-6e33770fe866">

   위에서 UPDATE 쿼리에 대한 잠금을 확인 해보면

   1. TABLE 에 대해 IX LOCK 이 걸린 것을 확인할 수 있다.
   2. 해당 RECORD에 X-LOCK, GAP-LOCK이 걸린 것을 확인 할 수 있다. 즉 존재하지 않는 RECORD에 대해서는 GAP LOCK을 얻는다.

   - 또한 SELECT...FOR UPDATE, DELETE등 X 잠금을 얻는 쿼리는 모드 GAP LOCK을 획득한다.
   - 범위에 대한 LOCK 을 획득 시 해당 범위 안에 존재하지 않는 RECORD는 모두 GAP LOCK을 획득하게 된다.
   - 
   
   - `Next-key Lock`
   - `Insert intention Lock`
   - `Auto-Inc Lock`
