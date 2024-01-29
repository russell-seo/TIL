# Select for update, insert DeadLock

필자는 게임회사에 재직중인데 MMORPG의 채팅서버를 개발하고 있으며 곧 출시를 앞에두고 성능 테스트를 하던중 DeadLock 이 걸리는 기능이 발견되었고 이를 해결하다 보니 Mysql 에서 `select for update` 와 `insert`가 같은 트랜잭션으로 묶여서 실행되면 `데드락`이 종종 발생할 수 있다는 사실을 알고 이를 정리할려고 한다.



## select for update
