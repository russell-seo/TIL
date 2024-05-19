# Redis Cluster

- Redis Cluster
  - Hash 기반으로 Slot 16384 로 구분
  - Hash 알고리즘은 CRC16 을 사용
  - Slot = crc16(key) % 16384
  - Key 가 Key{hashkey} 패턴이면 실제 crc16에 hashkey가 사용된다.
  - 특정 Redis 서버는 이 slot range 를 가지고 있고, 데이터 migration은 이 slot 단위의 데이터를 다른 서버로 전달하게 된다.(migrateCommand 이용)

Redis Cluster 에서 Pub/Sub 에 대해서 기술해 볼려고 한다. 현재 필자가 운영하고있는 서버는 Redis Sentinel로 구성되어 있는데 유저가 많아져서 Cluster 환경으로 확장할 경우에 Pub/Sub에 문제가 없는지에 대해 질문이 들어왔기에 이에 대해 테스트해보고 기술할 예정이다.

## Redis Pub/Sub의 동작원리

기본적으로 Redis Cluster의 Pub/Sub은 모든 노드에 데이터를 뿌리게 된다. 그래서 한대의 서버에 Publish 를 하면 해당 노드는 모든 Primary + Replica 들에 Publish 를 BroadCast 하게 되고, 

어떤 노드에 Subscribe를 하든 해당 BroadCast되는 메시지를 받을 수 있다.
