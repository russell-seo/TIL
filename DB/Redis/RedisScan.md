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

cursor 값을 0 으로 지정한 SCAN/SSCAN/ZSCAN/HSCAN 명령으로 순회가 시작되고, 이어지는 순회에 사용할 cursor 값과, 지정한 패턴과 일치하는 키를 최대 지정한 `count`만큼 반환 합니다.(기본 count 는 10)

반환된 cursor 값이 0이면 순회가 종료됩니다. Redis 가 Scan 으로 데이터를 반환하는 것은 바로 `Bucket 한턴에 하나씩 순회`하는 것이다. 아래에 Redis 가 저장될때의 동작 과정을 기술해 놓았으니 참고해보면 좋을 것이다.

>[Redis 데이터 저장원리](https://github.com/russell-seo/TIL/blob/main/DB/RedisApply.md)



### Scan과 Cursor

---

Redis Scan 에서의 Cursor는 `bucket을 검색해야할 다음 index 값`이라고 볼 수 있다. 실제로 실행 시켜 보면 0, 1, 2 이렇게 증가하지 않는다.

그 이유중에는 하나의 실제 Cursor 값이 다음 index의 reverse 값을 취하기 때문이다.

왜 reverse 를 취하는 것일까? 이것은 실제 적으로 1 씩 증가하는 형태라면.. cursor 가 언제 끝나는지 알려주기가 애매해서 라고 한다. 즉 끝났다는 값을 다시 줘야하는데 그것보다는 0으로 시작해서 다시 0으로 끝날 수 있도록 reverse 형태를 취하는 것이다.

> Redis 의 Scan 명령은 싱글쓰레드 아키텍쳐에서 Keys와 SMEMBERS 명령이 가진 문제점을 해결한 유용한 명령이지만 여러 문제점이 있기는 하다.
>
> 1. 기본적으로 scan의 경우 table의 한 블럭을 가져오는 것 이라서 여기에 개수가 많으면 시간이 많이 걸릴 확률이 높아진다. 하지만 rehashing 테이블이 bitmasking 크기만큼 커지므로, 블럭이 극단적으로 커질 가능성은 높지 않다.
>
> 2. set/sorted set/hash의 내부 구조가 hash table이나 skiplist가 아닐 경우(ziplist로 구현되어 있을 경우), 한 컬렉션의 모든 데이터를 가져오므로 Keys명령와 비슷한 문제가 발생할 수 있다.
> (ziplist로 구현되는 경우는 hash 같은 경우에는 field 개수가 512개 이하이거나, value값이 64byte 이상이어야 ziplist 가 아닌 hashtable로 구성된다고 알고 있다.)[참고](http://redisgate.kr/redis/configuration/ds_ziplist_hashes.php)
>
> 3. 명령의 옵션으로 count값을 지정할 수 있지만, 정확히 그 개수를 보장하지 않는다.
>
> 4. 순회가 시작(cursor값을 0 으로 지정한 scan 명령) 된 이후에 추가된 항목은 전체 순회 가 끝날 떄 가지 반환되지 않는다.
>
> 5. hash table이 확장/축소/rehashing 될 때 다시 스캔하지 않기 때문 같은 항목이 여러번 반환 될 수 있다. 반환 된 키 값으로 다른 명령을 실행하려면 주의 해야 한다.



## Scan 의 Count 수로 알아보는 성능 비교

필자는 타사의 과제리뷰 테스트를 하면서 Scan 에 대해서 직접 구현해보고 Key 설계 및 Scan 의 Count 값으로 인해 성능을 비교해 볼 예정이다.

먼저 Redis 는 광고 sdk를 타사의 서비스에 붙여서 app과 campaign 기준으로 노출 및 클릭 count 를 업데이트 하는 용도로 사용되고 매일 count 를 aggreagte 하여 db에 업데이트 하는 용도로 사용된다고 가정을 해보자.

기존에 설계되어있는 Redis Key는 serv:app:{appId}:cid:{cid}:date:{date}:hours:{hours} 로 설계되어 있었고 key 설계를 변경해야 했다.

저는 일단 해당 key 의 문제라고 하면 시간별로 key가 시간순으로 계속 무분별하게 늘어나는 것 입니다. 그래서 hash 타입으로 field를 날짜순으로 저장하고 해당 날짜에 들어온 요청의 value 값을 1씩 올려주게 변경을 하였다.

`key = serv:app:{appId}:cid:{cid}`, `field = 2024-05-24`, `value = 1`  

![image](https://github.com/russell-seo/TIL/assets/79154652/8dcca9a0-fe05-49a6-b697-9b4758cdaccf)

약 120만개 정도의 데이터를 Redis 에 미리 위와 같은 Key Field value 값으로 저장해 놓았다.

---

Redis 와 통신하기 위한 RedisTemplate 를 정의해 주고 redisTemplate의 execute 메소드를 통해서 Redis 명령어를 사용할 수 있다.

~~~ java

public Set<String> scan(String key, int count){
        return redisTemplate.execute((RedisCallback<Set<String>>) connection -> {
            Cursor<byte[]> scan = connection.scan(ScanOptions.scanOptions().match(key.getBytes()).count(count).build());
            Set<String> keys = new HashSet<>();
            while(scan.hasNext()){
                String rkey = new String(scan.next());
                keys.add(rkey);
            }
            scan.close();
            return keys;
        });
    }

~~~


### Count = 1000

~~~ java

@Test
void getAllAppByCid(){
int cid = new Random().nextInt(2000);
var key = "stats:app:"+"*"+":cid:"+cid;
//시작시간
long start = System.currentTimeMillis();
//

Set<String> scans = redisService.scans(key, 1000);

//
long end = System.currentTimeMillis();
long time = end - start;
System.out.println("GET KEY" +" ==== 걸리는 시간 ==== " + time + "ms");
System.out.println("key 갯수 = " + scans.size());
//

}
~~~

![image](https://github.com/russell-seo/TIL/assets/79154652/87c852a5-ce10-41c3-853f-dfe2b841ad5c)

Count 가 1000 일때 2000 중에서 랜덤으로 1개의 숫자를 가져와서 scan 의 성능을 처리했을때 key 를 가져오는데 1163ms 가 걸리는걸 확인 할 수 있다.
