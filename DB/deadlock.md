# Select for update, insert DeadLock

필자는 게임회사에 재직중인데 MMORPG의 채팅서버를 개발하고 있으며 곧 출시를 앞에두고 성능 테스트를 하던중 DeadLock 이 걸리는 기능이 발견되었고 이를 해결하다 보니 Mysql 에서 `select for update` 와 `insert`가 같은 트랜잭션으로 묶여서 실행되면 `데드락`이 종종 발생할 수 있다는 사실을 알고 이를 정리할려고 한다.



## SELECT FOR UPDATE

MYSQL 에서 SELECT FOR UPDATE 는 하나 또는 특정 범위의 ROW에 대해 여러 세션에서 접근하여 발생할 수 있는 동시성 문제를 해결하기 위해 이용할 수 있다.

기본적으로 `Intention Lock(table-level` IX락과, `Exclusive Lock(row-level)` X락이 걸리게 된다.

이렇게 해당 ROW에 X락이 걸리기 때문에 다른 트랜잭션에서 해당 ROW에 접근하지 못하고 대기하게 된다.










참고
---
[https://medium.com/daangn/mysql-gap-lock-%EB%91%90%EB%B2%88%EC%A7%B8-%EC%9D%B4%EC%95%BC%EA%B8%B0-49727c005084](https://medium.com/daangn/mysql-gap-lock-%EB%91%90%EB%B2%88%EC%A7%B8-%EC%9D%B4%EC%95%BC%EA%B8%B0-49727c005084)
