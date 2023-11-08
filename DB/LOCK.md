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
       - `SELECT....FOR UPDATE, INSERT, DELETE 가 실행되면 Intention Exclusive Lock 이 테이블에 걸린다, Row-Level 에 X-LOCK 걸림
   - `Row-Level Lock` : InnoDB Storage engine 은 기본적으로 Row-Level Lock 사용
   - `Record Lock`
   - `Gap Lock`
   - `Next-key Lock`
   - `Insert intention Lock`
   - `Auto-Inc Lock`
