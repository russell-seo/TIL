
# Redis

  인메모리 데이터 저장소라고 부르는 Redis 에 대해 간략하게 공부한 내용을 기록해볼려고 한다.
  
  ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbrCdyD%2FbtqWBAZA5rE%2FPgs4thI67EvRWDRis29Jgk%2Fimg.png)
  
  Redis는 외부에 key-value 를 저장하는 서버를 말한다. `인메모리` 데이터 구조 저장소로 데이터베이스, `캐시`, 메시지 브로커로 사용한다고 말한다.


 여기서 인메모리에 대해 잠시 공부해보고 가자.
 
 ## 인메모리(In-memory)?
 
 ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FBcwuU%2FbtqWKYSRi5Q%2FjRblmXQlGtIdD4nURFkHWk%2Fimg.png)
 
 인메모리란 컴퓨터의 메인 메모리 `RAM`에 데이터를 올려서 사용하는 방법을 말한다. 왜 메모리에 데이터를 올릴까? 이유는 명확하게도 속도 때문이다.
 
 SSD, HDD 같은 저장공간에서 데이터를 가져오는 것보다 `RAM`에 올려진 데이터를 가져오는데 걸리는 속도가 수백배(HDD 기준) 이상 빠르다. 때문에 Redis 는 `빠른 속도`가 큰 장점이다.
 
 ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcJNBig%2FbtqWzsOg5he%2FX3tHPGyI7obqDSlc9LDokk%2Fimg.png)
 
 Redis는 빠른 속도를 자랑하는 대신 치명적인 단점이 있다. 바로 `용량` 이다. 내 노트북만 봐도 RAM 용량은 8GB 이다. 그래서 메인 데이터베이스로 사용하기에는 무리가 있다.
 
 그렇다고 RAM을 10TB씩 구매하자니 비용이라는 걸림돌이 있다.
 
 또 다른 특징으로는 Key-Value 형티의 NoSql 이라는 점이다. Redis가 다양한 형태의 데이터 구조를 지원하기는 하지만 복잡한 데이터를 저장하는 DB로 사용하기에는 어려움이 있다.
 
 즉, Redis는 보조적인 수단으로 사용되는 경우가 많고 가장 적합한 역할은 `캐시 데이터베이스 서버`이다.
 
 ex)게임의 랭킹 상위 100위를 보여주는 기능이 있다고 해보자. 랭킹 정보를 사용자에게 제공하기 위해 오라클같은 관계형 데이터베이스에 랭킹 정보를 저장을 하고 order by 로 불러올 수 있다. 
 그런데 사용자 수가 폭발적으로 증가해 수백만명으로 늘어나게 되면 어떻게 될까? 보여주는건 100명의 데이터뿐이지만 시간이 점점 오래 걸리게 될 것이다.
 이러한 문제를 해결하기 위해 캐시에 상위 100명의 랭킹정보를 담고 있다면 좋은 해결책이 될 것이다
 
 
 
 ## Redis 특징
 
  - 영속성을 지원하는 인메모리 데이터 저장소
  - 읽기 성능 증대를 위한 서버 측 복제를 지원합니다.
    - 전체 데이터베이스의 초기 복사본을 받는 Master/Slave 복제를 지원합니다.
    - Master에서 쓰기가 수행되면 Slave 데이터 세트를 실시간으로 업데이터 하기 위해 연결된 모든 Slave로 전송됩니다.
  - 쓰기 성능 증대를 위한 클라이언트 측 샤딩(Sharding)을 지원합니다.
  - `String, Set, Sorted Set, Hash, List` 과 같은 다양한 데이터형을 지원합니다.
  
    > 샤딩(Sharding)
      파티셔닝과 동일하며 같은 테이블 스키마를 가진 데이터를 다수의 데이터베이스에 분산하여 저장하는 방법을 의미합니다.
      
    Key-Value Store
    
    ![image](https://user-images.githubusercontent.com/42582516/133774329-00ddf3c0-a24e-40b0-9dd8-460616ea5400.png)
        
       - Redis는 거대한 Map 데이터 저장소 입니다.
       - Redis는 익히기 쉬우며 직관적입니다. 그러나 데이터를 Redis 자체 내에서는 처리하기 어렵습니다.
   
    __다양한 데이터 타입__
    
    - `String, Set, Sorted Set, Hash, List` 등의 타입을 지원합니다.


    __Persistence__
    
    ![image](https://user-images.githubusercontent.com/42582516/133775761-c7644499-ae6f-4aa8-bd25-8208780c41e0.png)
    
    
    - Redis는 영속성을 가집니다.
    - Redis는 데이터를 disk에 저장할 수 있습니다. 따라서 Redis는 서버가 강제 종료되고 재시작 하더라도 disk에 저장해놓은 데이터를 다시 읽어서 데이터가 유실되지 않습니다.
    
    
    ## Redis의 데이터를 disk에 저장하는 방식은 snapshot, AOF 방식이 있습니다.
    
    - `Snapshot` : RDB와 비슷하게 어떤 특정 시점의 데이터를 Dist에 담는 방식을 뜻합니다. Blocking 방식의 SAVE와 Non-blocking 방식의 BGSAVE 방식이 있습니다.
    - `AOF` : Redis의 모든 write/update를 순차적으로 재실행하고 데이터를 복구합니다.
    
    - 가장 좋은 방법은 두 방법을 혼용해서 사용하는 방법으로 주기적으로 snapshot으로 백업을 하고 다음 snapshot까지의 저장을 AOF 방식으로 수행하는 방식


# Redis 관리하기

Redis의 대표적인 특징은 Single Threaded(싱글쓰레드) 라는 점이다. 다시 말해 Redis는 한 번에 딱 하나의 명령어만 실행할 수 있다는 뜻 입니다.

만약 명령어를 포함한 Packet이 MTU(Maximum Trasmission Unit)보다 크면 Packet이 쪼개져서 올 수 있는데, Redis는 쪼개진 명령어를 합쳐서 하나

의 명령어가 되는 순간 그 명령어를 실행합니다. Single Thread라서 느리다고 생각 할 수 있지만, 평균적으로 __Get/Set 명령어 같은 경우 초당
10만개 정도 까지도 처리할 수 있다고 한다.

다만 `조심할 점은 처리시간이 긴 명령어를 중간에 넣으면 그 뒤에 있는 명령어들은 전부 기다려야 한다는 것` 이다. 대표적으로 전체 키를 불러오는

Keys 명령어가 상당히 오래걸리는데, 만약 중간에 Keys명령어를 실행하면 그 뒤에 오는 Get/Set 명령어들은 타임아웃이 나서 요청에 실패할 수도 있다.

> Redis를 사용하여 서비스를 운영하다보면 Redis 서버의 메모리가 한계에 도달할 수 있다. 메모리의 한계는 MaxMemory 값으로 설정할 수 있다.
  MaxMemory 수치까지 메모리가 다 차는 경우 Redis는 Max Memory Policy에 따라서 추가 메모리를 확보합니다.


- Maxmemory-policy  설정값
  
  1. noeviction : 기존 데이터를 삭제하지 않습니다. 메모리가 꽉 찬 경우에는 OOM(Out of Memory)오류 반환하고 새로운 데이터는 버린다.
  2. allkeys-lru :LRU(Least Recently Used)라는 페이지 교체 알고리즘을 통해 데이터를 삭제하여 공간을 확보한다.
  3. volatile-lru : expire set을 가진 것 중 LRU로 삭제하여 메모리 공간을 확보한다.
  4. allkys-random : 랜덤으로 데이터를 삭제하여 공간을 확보한다.
  5. volatile-random : expire-set을 가진 것 중 TTL(Time to live) 값이 짧은 것 부터 삭제한다.
  6. volatile-ttl : expire set을 가진 것 중 TTL이 짧은 것 부터 삭제
  7. allkeys-lfu: 가장 적게 엑세스한 키를 제거하여 공간을 확보
  8. volatile-lfu : expire set을 가진 것 중 가장 적게 엑세스한 키부터 제거하고 공간을 확보


  
  > Maxmemory 초과로 인해서 데이터가 지워지게 되는 것을 eviction 이라고 합니다. Redis에 들어가서 INFO 명령어를 친 후 evicted_keys 수치로
  보면 eviction이 발생했는지 알 수 있습니다. Amazon Elasticache를 사용하는 경우 모니터링 tab에 들어가면 eviction에 대한 그래프가 있는데
  이를 토해 Eviction 여부에 대한 알림을 받을 수 있습니다.
  
  
  - evicted_keys 값은 evicted된 키들의 count 이므로 크면 클수록 많은 데이터가 메모리에서 삭제되었다는 뜻 입니다.
    evicted_keys : 0
    
    그런데, Maxmemory가 설정된 대로 작동하면 좋겠지만, 그렇지 않은 경우가 있습니다. Redis는 쓰기 요청이 발생하면 __COW(Copy on Write)방식__ 을 통해 작동합니다. 쓰기 요청이 오면 OS는 fork()를 통해서 자식 프로세스를 생성합니다. `fork() 시에는 다른 가상메모리 주소를 할당받지만
    물리 메모리 블록을 공유합니다. 쓰기 작업을 시작하는 순간에는 수정할 메모리 페이지를 복사한 후에 쓰기 작업을 진행` 합니다. 즉, 기존에 쓰던
    메모리보다 추가적인 메모리가 필요합니다. 다만 전체 페이지 중에서 얼마나 작업이 진행될지를 모르기 때문에 fork시에는 기본적으로 복사할 사이즈 만큼의 free memory가 필요합니다.

  - Redis를 직접 설치할때 "/proc/sys/vm/overcommit_memory" 값을 1로 설정하지 않아 장애가 날 때가 있다.
  
    ![image](https://user-images.githubusercontent.com/79154652/152163664-5b1e6184-d61b-4dd4-bf26-cdbff136a716.png)
    
    overcommit_memory=0 이면 OS 는 주어진 메모리량 보다 크게 할당할 수가 없다. 즉 fork()시에 OS가 충분한 메모리가 없다고 판단하기 때문에
    에러를 발생시킨다. `overcommit_memomry=1`로 설정해서  OS한테 일단 over해서 메모리를 할당할 수 있도록 한 후에 max memomry에 도달한 경우 policy에 따라 처리되도록 설정하는 것이 좋다.
    
    > 또한 Redis에서는 memory 수치 중에서 used_memory_rss 값을 잘 살펴볼 필요가 있다. RSS 값은 데이터를 포함해서 실제로 사용하고 있는 used_memory 값 보다 클 수 있다. 이러한 현상이 발생하는 이유는 OS가 메모리를 할당할 때 page 사이즈의 배수만큼 할당하기 때문이다. 예를 들어
    page size = 4096인데, 요청 메모리 사이즈가 10 이라고 하면 OS는 4096만큼 할당한다. 이를 Fragmentation(파편화) 현상 이라고 하는데, 이것이 실제 사용한 메모리랑 할당된 메모리가 다른 원인이 된다.
