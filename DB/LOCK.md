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

   `GAP LOCK 이 필요한 이유`
   - REPEATABLE READ 격리 수준 보장
    - GAP LOCK 이 없다면 PHANTOM READ 가 발생할 수 있다.
   ![image](https://github.com/russell-seo/TIL/assets/79154652/71d6700d-316c-4406-b7bf-cd19477dcea1)

   - 실제로 위와 같이 실행 시 3,6번의 데이터가 다르게 조회된다. 그 이유는 READ-COMMITED의 경우 3번 실행할 때 배터적 잠금을 걸지 않는다. 즉 GAP LOCK을 걸지 않는다.
   - 하지만 REPEATABLE READ는 같은 트랜잭션 범위 내 에서는 조회한 데이터가 동일 해야 한다. 그러므로 3번 쿼리를 실행할 때 id 인덱스에 GAP LOCK을 건다. 즉 id=2 에 GAP LOCK을 건다
   - 그러므로 4번 쿼리의 INSERT는 Session-1 트랜잭션이 끝날 때 까지 대기하고 끝나면 실행한다.
   - 이렇게 GAP LOCK을 사용하면서 Phantom Read 문제를 해결한다.

   - REPLICATION 일관성 보장(Binary Log Format = statement or Mixed 일때)
     - Mysql 서버의 Replication은 데이터의 복제를 위하여 바이너리 로그 파일을 사용하는데, 바이너리 로그 포맷은 `Statement, Row, Mixed`중 하나를 선택할 수 있다.
     - Statement와 Row는 일반적인 경우 실행된 SQL 문장을 바이너리 로그로 저장하기 때문에 거의 유사한 포맷

     ![image](https://github.com/russell-seo/TIL/assets/79154652/e74cce83-f6cf-49ad-9422-cb962bde6238)

     이 예제에서는 4번 INSERT 가 즉시 완료되지 않고 3번 UPDATE 세션의 트랜잭션이 COMMIT OR ROLLBACK 되어야 4번의 쿼리가 실행된다.

     이는 3번 쿼리가 id = 1, 3 뿐 만 아니라 id = 2인 GAP LOCK을 가지고 있기 때문이다.

     만약 GAP LOCK 없이 세션 1번보다 2번이 먼저 실행되고 COMMIT 되어 버리면 SOURCE DB와 REPLICATION DB의 트랜잭션 실행 순서가 거꾸로 되기 때문에 데이터가 서로 달라지게 된다.

     이렇게 바이너리 로그 포맷 = Statement 일 때 GAP LOCK 이 필요한 이유는 이런 복제 Source 와 Replica DB 데이터 정합성 때문이다.


   `GAP LOCK의 특징과 주의사항`

   Shared Gap Lock = Exclusive Gap Lock

   Next Key Lock = Record Lock + Gap Lock


   Gap Lock은 Mysql 내부적으로 Shared Lock 만 존재한다. 순수하게 다른 트랜잭션이 대상 간격에 대해서 INSERT 되는 것을 막아주는 것만이 목적이다.

   그래서 UPDATE, DELETE, SELECT...FOR UPDATE 문장에 의한 GAP LOCK 이더라도(설령 performance_schema.data_locks에 보여지는 정보가 “LOCK_MODE: X,GAP”로 Exclusive Gap Lock이라 하더라도)

   여러 트랜잭션 의 GAP LOCK은 호환된다. 즉 동시에 서로 다른 트랜잭션에서 UPDATE table set .. where id = 2 를 실행시켜도 잠금 경합이 발생하지 않는다. 


   `GAP LOCK 은 테이블의 데이터 건수가 적을수록 GAP LOCK의 범위가 넓어지는 역효과를 준다.`


   ### Next-key Lock

   
   - `Insert intention Lock`
   - `Auto-Inc Lock`
