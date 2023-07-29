# Mysql DB에서 트랜잭션 시 작동원리 이해하기(Redo VS Undo)

  - Redo Log : 변경 후의 값을 기록
  - Undo Log : 변경 전의 값을 기록


  ## Redo Log

  - Redo Log란 DB장애 발생시 복구에 사용되는 Log 이다.
  - Mysql 장애시 Buffer Pool에 저장되어 있던 데이터의 유실을 방지 하기 위해 사용된다.
  - Redo Log를 이해할려면 우선 InnoDB Buffer Pool 에 대해 이해해야 한다.
    - Buffer Pool은 InnoDB 엔진이 Table Caching 및 Index Data Caching을 위해 이용하는 메모리 공간
    - Buffer Pool이 클수록 상대적으로 캐싱되는 데이터의 양이 늘어나기 때문에 Disk 접근 횟수가 줄어들고 DB 성능 향상으로 이어진다.
    - But, Buffer Pool은 메모리 공간이기 때문에 Mysql 장애 발생 시 Buffer Pool에 있는 내용은 사라지게 된다.
    - 이것은 ACID를 보장할 수 없게 되고 데이터 복구가 될 수 없다는 것을 의미한다.
   
  ![image](https://github.com/russell-seo/TIL/assets/79154652/f4555f52-9261-4523-bba0-b9da0ce964cf)


  - 위 그림은 Mysql 의 Commit 실행 과정이다.

> 실제 DB에서 Commit 이 발생하면 바로 `디스크 영역`으로 들어가는 것이 아닌 메모리 영역(Buffer Pool & Log Buffer)에 들어가는 것을 확인 할 수 있다.

  ## 항상 Redo Log에 기록되나?

  - 항상 Redo Log에 기록되는 것은 아니다 `데이터 변경`이 있을 시에 기록되고 조회 쿼리는 기록되지 않는다.

  - Redo Log는 Redo Log Buffer에 저장된다. 즉 얘도 결국 메모리 영역이니 장애가 나면 사라지게 된다.
  - 이 Redo Log 를 복구하는 것이 Redo Log File 이다.
  - Redo Log File은 두 개의 파일로 구성되는데, 하나의 파일이 가득차면 log switch가 발생하며 다른 파일에 쓰게 된다. log switch가 발생할 때 마다 CheckPoint 이벤트도 발생하는데, 이때 InnoDB Buffer Pool Cache에 있던 데이터들이 백그라운드 스레드에 의해 디스크에 기록된다.




## Undo Log

- 실행 취소 로그 레코드의 집합으로 트랙잭션 실행 후 Rollback 시 Undo Log를 참조해서 이전 데이터로 복구할 수 있도록 로깅 해놓은 영역이다.
- 예를 들면 기존에 [서상원, 1000] 이라는 Row가 있으면 이를 1000->900으로 update 하는 동작을 수행한다고 하자, 그러면 DB에 해당 Column을 update 할 때 900이라는 데이터가 먼저 메모리 영역에 저장되고 기존의 1000이라는 데이터는 Undo Log에 기록된다. 이 때 만약 Commit 되지 않고 에러가 발생해서 Rollback이 된다면 이 Undo Log의 기존 데이터로 다시 원상복귀 시킨다.


~~~
--데이터 Insert
INSERT INTO member(m_id, m_name, m_area) VALUES(12, '홍길동', '서울');

-- 변경
UPDATE member SET m_area='경기' WHERE m_id = 12;
~~~

![image](https://github.com/russell-seo/TIL/assets/79154652/5a237338-9b91-4699-85d1-8bfbd0776d27)

- Undo Log도 Redo Log와 마찬가지로 Log Buffer에 기록된다. Undo Recoders 영역에 기록된다. 저장되는 데이터는 PK값과 변경되기 전의 데이터 이다.

![image](https://github.com/russell-seo/TIL/assets/79154652/4f1bb2c7-1bc0-4f66-9c49-ee8479362458)

`Redo Log`가 트랜잭션 커밋과 Checkpoint시 디스크에 기록, `Undo Log` 는 Checkpoint시 디스크에 기록

위 그림을 보면 Update 쿼리가 실행되면 Commit, Rollback 전 InnoDB buffer pool에 캐싱된 데이터는 update 한 정보로 수정됩니다. 데이터를 수정함과 동시에 Rollback을 대비하기 위해, 업데이트 전의 데이터를 Undo Records로 기록하는 것이다.
