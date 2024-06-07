# Redis Cluster

- `Redis Cluster`
  - Hash 기반으로 Slot 16384 로 구분
  - Hash 알고리즘은 CRC16 을 사용
  - Slot = crc16(key) % 16384
  - Key 가 Key{hashkey} 패턴이면 실제 crc16에 hashkey가 사용된다.
  - 특정 Redis 서버는 이 slot range 를 가지고 있고, 데이터 migration은 이 slot 단위의 데이터를 다른 서버로 전달하게 된다.(migrateCommand 이용)
  - Redis Cluster 를 구성하는 각 Redis는 다른 모든 Redis 들과 직접 연결하여 gossip Protocol을 통해 통신한다.
    - gossip Protocol을 통해서 각 Redis는 redis 상태정보를 교환한다. Redis cluster node 들은 2개의 TCP connection port 가 필요하며 일반적으로 cluster bus port 는 기본 6379포트에 10000을 더한 16379를 사용한다.

## Cluster Bus
  - 자체적인 binary protocol 을 통해 node-to-node 통신을 한다.
  - failure detection, configuration update, failover authorization 등을 수행한다.
    
Redis Cluster 에서 Pub/Sub 에 대해서 기술해 볼려고 한다. 현재 필자가 운영하고있는 서버는 Redis Sentinel로 구성되어 있는데 유저가 많아져서 Cluster 환경으로 확장할 경우에 Pub/Sub에 문제가 없는지에 대해 질문이 들어왔기에 이에 대해 테스트해보고 기술할 예정이다.

## Redis Pub/Sub의 동작원리

기본적으로 Redis Cluster의 Pub/Sub은 모든 노드에 데이터를 뿌리게 된다. 그래서 한대의 서버에 Publish 를 하면 해당 노드는 모든 Primary + Replica 들에 Publish 를 BroadCast 하게 되고, 

어떤 노드에 Subscribe를 하든 해당 BroadCast되는 메시지를 받을 수 있다.

![image](https://github.com/russell-seo/TIL/assets/79154652/84de414f-3b66-42f7-8bb0-11183b0a068b)

`일단 Redis Cluster에서의 문제는 항상 Broadcast 하는 것이다`, 안그래도 Pub/Sub 자체가 Pattern 을 지원해서 모든 채널을 확인해야 하기 때문에 보통 메시지 전달은 채널수 + 채널에 붙은 클라이언트 수 만큼 루프를 돌아야 한다.

항상 모든 서버에서 일단 Broadcast 해야 한다는게 전체 성능에 영향을 미칠 수가 있게 되는 것이다. 

이러한 문제때문에 Redis 에 오히려 싱글노드로 구성했을 때와 비슷하게 모든 마스터 노드에 부하가 가게 되어있다. 이 문제를 해결하기 위해서 Redis 에서 `Shared Pub/Sub` 을 Redis 7.X 에서부터 도입하게 되었다.

## Shared Pub/Sub

이는 기존 Redis Cluster의 특성을 기존 그대로 사용한다. 일반적으로 Key를 crc16으로 Hash 해서 해당 키가 속한 Slot 을처리하는 서버로만 메시지를 전달하게 되는데 이 특성을 이용해서 Pub/Sub 도 하나의 Shard 군에서만 처리하게 된다.

![image](https://github.com/russell-seo/TIL/assets/79154652/defaa32f-c85d-423a-adf8-1e147b3a2cbd)



--- 참고
[https://charsyam.wordpress.com/2022/04/18/%EC%9E%85-%EA%B0%9C%EB%B0%9C-redis-7-x-%EC%97%90%EC%84%9C%EC%9D%98-shardedpubsub/](https://charsyam.wordpress.com/2022/04/18/%EC%9E%85-%EA%B0%9C%EB%B0%9C-redis-7-x-%EC%97%90%EC%84%9C%EC%9D%98-shardedpubsub/)
