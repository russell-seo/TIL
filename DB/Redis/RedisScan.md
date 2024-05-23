# Redis Scan 으로 Keys 대체 및 테스트

- Redis 는 싱글 쓰레드 기반으로 동작하는 InMemory Key/Value 기반의 NoSQL DB이다. 즉 1개의 요청만 처리가 가능하다는 말이다.
- Redis 에서 `Keys`, `flushall`등의 시간복잡도가 O(N)이며 모든 데이터를 순회하는 명령은 가급적 프로덕션 레벨에서는 사용을 금지하고 있다.
  - `Keys`, `flushall`등의 오래 걸리는 요청들로 인해서 다른 요청들이 Blocking 되는 현상으로 성능 상의 문제가 생길 수 있기 때문이다.
  - 프로덕션 레벨에서 특히 Redis 의 Key 를 가져오는 경우가 많다. 여기서 Keys 대신 `Scan` 이라는 명령어를 사용하여 이러한 문제점을 해결 할 수 있다.
 


## Redis Scan 이란?
  - Redis Scan은 4가지의 명령이 존재합니다.
    - `SCAN`은 전체 key 목록에서, `SSCAN`은 set 안에서, `ZSCAN`은 sorted set 안에서, `HSCAN`은 hash 안에서 가져오는 명령입니다.
    ~~~redis
    SCAN cursor [MATCH PATTERN] [COUNT count]

    SSCAN key cursor [MATCH PATTERN] [COUNT count]

    ZSCAN key cursor [MATCH PATTERN] [COUNT count]

    HSCAN key cursor [MATCH PATTERN] [COUNT count]
    ~~~

cursor 값을 0 으로 지정한 SCAN/SSCAN/ZSCAN/HSCAN 명령으로 순회가 시작되고, 이어지는 순회에 사용할 cursor 값과, 지정한 패턴과 일치하는 키를 최대 지정한 `count`만큼 반환 합니다.

반환된 cursor 값이 0이면 순회가 종료됩니다. Redis 가 Scan 으로 데이터를 반환하는 것은 바로 `Bucket 한턴에 하나씩 순회`하는 것이다. 아래에 Redis 가 저장될때의 동작 과정을 기술해 놓았으니 참고해보면 좋을 것이다.

>[Redis 데이터 저장원리](https://github.com/russell-seo/TIL/blob/main/DB/RedisApply.md)
