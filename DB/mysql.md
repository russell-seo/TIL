# Mysql DB에서 트랜잭션 시 작동원리 이해하기


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
