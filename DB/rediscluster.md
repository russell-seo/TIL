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
[https://charsyam.wordpress.com/2022/04/18/%EC%9E%85-%EA%B0%9C%EB%B0%9C-redis-7-x-%EC%97%90%EC%84%9C%EC%9D%98-shardedpubsub/](https://charsyam.wordpress.com/2022/04/18/%EC%9E%85-%EA%B0%9C%EB%B0%9C-redis-7-x-%EC%97%90%EC%84%9C%EC%9D%98-shardedpubsub/)
