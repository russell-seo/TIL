
# Redis Replication

  - Redis를 구성하는 방법 중에서 Read 분산과 데이터 이중화를 위한 Master/Slave 구조가 있습니다. Master 노드는 쓰기/읽기를 전부 수행하고,
    Slave는 읽기만 가능합니다. 이렇게 하려면 Slave는 Master 의 데이터를 전부 가지고 있어야 한다. 이럴 때 발생하는 것이 Replication 입니다.
    
    `Replication은 마스터에 있는 데이터를 복제해서 Slave로 옮기는 작업입니다. Slave가 싱크를 받는 작업은 다음과 같다.`
    
  ## Master-Slave간 Replication 작업 순서
   1. Slave Configuration 쪽에 "replicaof<master IP><master PORT>" 설정을 하거나 REPLICAOF 명령어를 통해 마스터에 데이터를 Sync 요청한다.
    
   2. Master는 백그라운드에서 RDB파일(현재 메모리 상태를 담은 파일) 생성을 위한 프로세스를 진행합니다. 이 때 Master는 fork를 통해 메모리를 복사한다.
      이후에 fork한 프로세스에서 현재 메모리 정보를 디스크에 덤프 뜨는 작업을 진행한다.
    
   3. 2번 작업과 동시에 Master는 이후부터 들어오는 쓰기 명령들을 Buffer에 저장해 놓는다.
   4. 덤프 작업이 완료되면 Master는 Slave에 해당 RDB파일을 전달해주고, Slave는 디스크에 저장한 후에 메모리로 로드한다.
   5. 3번에서 모아두었던 쓰기 명령들을 전부 Slave로 보내준다.
  
   > Master가 fork하는 부분에서 자신이 쓰고 있는 메모리만큼 추가로 필요해진다. 따라서 Replication 을 할 때 OOM(Out of Memory)이 발생하지
      않도록 주의하는 것이 중요하다. 성공적으로 Replication을 마쳤다고 하더라도 개선할 점은 아직 남아 있다. 바로 Master 노드가 죽게되는 시나리오 이다.
  
  Master가 죽은경우에는 Slave는 마스터를 잃어버리고 Sync 에러를 낸다. 이상태에서는 쓰기는 불가능하고 읽기만 가능하다. 
  따라서 Slave를 Master로 승격 시켜야 한다. 이런 작업을 매번 장애마다 할 수는 없기 때문에 다양한 방법을 통해서 failover에 대응할 수 있다. 
  
  하나 예를 들면 DNS기반으로 failover에 대응 할 수 있다. Client는 Master 도메인을 계속 바라보게 한 후에, 만약 마스터에 장애가 발생하면 Master DNS를 Slave에 매핑한다. 그러면 Client는 특별한 작업 없이도 Slave쪽으로 붙게 된다.
  
  ## Redis Cluster
  
  Redis Cluster는 failover를 위한 대표적인 구성방식 중 하나이다. Redis Cluster는 여러 노드가 Hash 기반의 Slot을 나눠가지면서 클러스터를 구성하여
  사용하는 방식이다. 전체 slot은 16384이며 Hash 알고리즘은 CRC16을 사용한다. Key를 CRC16으로 해시한 후에 이를 16384로 나누면 해당 key가 저장될
  slot이 결정된다.
  
 ![image](https://user-images.githubusercontent.com/79154652/152170312-454f759b-e065-4bea-a6e6-8ab8ea7a249c.png)
  
  Cluster를 구성하는 각 노드들은 master노드로, 자신만의 특정 slot Range를 갖습니다. 다만 데이터를 이중화하기 위해서 위에서 설명한 Slave 노드를 가질
  수 있다. 
  
  즉, 하나의 클러스터는 여러 Master 노드로 구성할 수 있고, 한 Master 노드가 여러 Slave를 가지는 구조이다. 만약 특정 Mastser노드가 죽게되면, 해당 노드의 Slave중 하나가 Master로 승격하여 역할을 수행하게 된다.
  
  - Slave 가 죽어서 복제 노드가 없는 마스터가 생길 시 다른 마스터 노드에 여유분이 있다면 해당 노드로 빈자리를 채울 수 있다.
    - 사용자가 개입하지 않고 `Cluster`가 알아서 다 해준다.
  
![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Frz5QG%2Fbtq4yjBRrMO%2FKxGGhgnVxgUi6jNtRkVNU0%2Fimg.png)
  
 
     - Master 가 죽었을 시 Slave 가 master로 자동 failover 된다.
     - 최소 3개의 마스터 노드가 있어야 구성 가능하다.
     - 센티널이 노드를 감시했지만, Cluster 에서는 모든 노드가 서로 감시한다.
  
  
  # Redis 설치 및 구성
  
  필자가 구성할 아키텍쳐는 아래와 같다.
  
    
  ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F0vAII%2Fbtq4DhJYDuA%2FhGfHUmIO9FuXfGeKPkhSTK%2Fimg.png)
    
    
 1. Redis 설치
  
  ~~~java
  $ wget http://download.redis.io/releases/redis-5.0.5.tar.gz 
  $ tar xzf redis-5.0.5.tar.gz 
  $ cd redis-5.0.5 
  $ make // 소스 컴파일(=소스 파일을 실행 가능한 형태로 만들어 준다) 
  $ sudo make install // /usr/local/bin에 redis-server, redis-cli 등 실행 파일이 복사 // redis 서버 실행 (오류 안뜨고 잘되면 성공!) 
  $ redis-server redis.conf(redis.conf 파일이 있는 경로에서) 
  ~~~
  
 2. 설정
  
  3개의 마스터 노드와, 3개의 Slave 노드를 띄우기 위해 설정 파일을 복사 한다.
  
  ~~~java
  $ cp redis.conf redis_6379.conf
  $ cp redis.conf redis_6378.conf
  $ cp redis.conf redis_6377.conf
  $ cp redis.conf redis_7379.conf
  $ cp redis.conf redis_7378.conf
  $ cp redis.conf redis_7377.conf
  ~~~
  
  ~~~java
  //설정 수정(포트를 다르게 넣는 부분 빼고 모두 동일하게 수정한다)
  
  $ vi redis_[포트].conf
    port [각자포트]
  # 백그라운드에서 시작 설정
    daemonize yes
  # 클러스터를 사용한다.
    cluster-enabled yes
  # 클러스터 구성 내용을 저장하는 파일 명 지정(자동생성됨)
    cluster-config-file nodes-[포트].conf
  # 클러스터 노드가 다운되었는지 판단하는 시간 (3s)
    cluster-node-timeout 3000
  # Appendonly를 yes로 설정하면 rdb에 저장 안되고 aof에 저장됨
    appendonly yes
  # append only yes 시 해당 부분도 수정
    appendfilename appendonly_[각자포트].conf
  # 프로세스 아이디 저장 경로 설정
    pidfile /var/run/redis_[각자포트].pid
  # 로그파일 저장 경로 설정
    logfile logs/redis_[각자포트].log
  ~~~
  
 3. 서버 기동
  ~~~java
  $ redis-server redis_6379.conf
  $ redis-server redis_6378.conf
  $ redis-server redis_6377.conf
  $ redis-server redis_7379.conf
  $ redis-server redis_7378.conf
  $ redis-server redis_7377.conf
  ~~~
  
 4. 서버가 돌아가고 있는 프로세스 확인
  
  - 서버는 해당 서버에 올라가 있지만, Master와 Slave 설정을 하지않은 상태이다.
  
  ![image](https://user-images.githubusercontent.com/79154652/153111250-410c5487-1421-427e-afaf-9da6f7a60d35.png)

  
 5. 마스터 설정
  ~~~java
  $ redis-cli --cluster create 0.0.0.0:6379 0.0.0.0:6378 0.0.0.0:6377
  
  >>> Performing hash slots allocation on 3 nodes... 
  Master[0] -> Slots 0 - 5460 
  Master[1] -> Slots 5461 - 10922 
  Master[2] -> Slots 10923 - 16383 
  M: 1421ba67b8753514d8b4468bfba6691492600b15 127.0.0.1:6379 
  slots:[0-5460] (5461 slots) master 
  M: a9a9cf5addf2b403f9d7a19f2a7436a69b9ee29a 127.0.0.1:6378 
  slots:[5461-10922] (5462 slots) master 
  M: b6d509a5de3a12df80922d5cc9508a0e9031f4c9 127.0.0.1:6377 
  slots:[10923-16383] (5461 slots) master 
  Can I set the above configuration? (type 'yes' to accept): yes 
  >>> Nodes configuration updated 
  >>> Assign a different config epoch to each node 
  >>> Sending CLUSTER MEET messages to join the cluster 
  Waiting for the cluster to join . 
  
  >>> Performing Cluster Check (using node 127.0.0.1:6379) 
  M: 1421ba67b8753514d8b4468bfba6691492600b15 127.0.0.1:6379 
  slots:[0-5460] (5461 slots) master 
  M: b6d509a5de3a12df80922d5cc9508a0e9031f4c9 127.0.0.1:6378 
  slots:[10923-16383] (5461 slots) master 
  M: a9a9cf5addf2b403f9d7a19f2a7436a69b9ee29a 127.0.0.1:6377 
  slots:[5461-10922] (5462 slots) master 
  [OK] All nodes agree about slots configuration. 
  >>> Check for open slots... 
  >>> Check slots coverage... 
  [OK] All 16384 slots covered.
  
  ~~~
  
  
 6. Slave 등록
  ~~~java
  $ redis-cli --cluster add-node 127.0.0.1:7379 127.0.0.1:6379 --cluster-slave 
  $ redis-cli --cluster add-node 127.0.0.1:7378 127.0.0.1:6378 --cluster-slave 
  $ redis-cli --cluster add-node 127.0.0.1:7377 127.0.0.1:6377 --cluster-slave 
  >>> Adding node 127.0.0.1:7379 to cluster 127.0.0.1:6379 
  >>> Performing Cluster Check (using node 127.0.0.1:6379) 
  M: 1421ba67b8753514d8b4468bfba6691492600b15 127.0.0.1:6379 
  slots:[0-5460] (5461 slots) master 
  M: b6d509a5de3a12df80922d5cc9508a0e9031f4c9 127.0.0.1:6378 
  slots:[10923-16383] (5461 slots) master 
  M: a9a9cf5addf2b403f9d7a19f2a7436a69b9ee29a 127.0.0.1:6377 
  slots:[5461-10922] (5462 slots) master 
  [OK] All nodes agree about slots configuration. 
  >>> Check for open slots... 
  >>> Check slots coverage... 
  [OK] All 16384 slots covered. 
  Automatically selected master 127.0.0.1:6379 
  >>> Send CLUSTER MEET to node 127.0.0.1:7379 to make it join the cluster. 
  Waiting for the cluster to join 
  >>> Configure node as replica of 127.0.0.1:6379. 
  [OK] New node added correctly.

  ~~~

  
 7. 클러스터 확인
  
  ![image](https://user-images.githubusercontent.com/79154652/153112849-5758139d-60c9-454c-80a9-8914e7373003.png)
  
 8. Spring Bean 에 등록
  
 ~~~java
        @Bean
        public RedisConnectionFactory redisConnectionFactory(){
            LettuceClientConfiguration lettuceClientConfiguration = LettuceClientConfiguration.builder()
                    .readFrom(ReadFrom.REPLICA_PREFERRED)
                    .build();

            RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration()
                    .clusterNode("3.34.122.63", 6379)
                    .clusterNode("3.34.122.63", 6378)
                    .clusterNode("3.34.122.63", 6377)
                    .clusterNode("3.34.122.63", 7379)
                    .clusterNode("3.34.122.63", 7378)
                    .clusterNode("3.34.122.63", 7377);

            LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(redisClusterConfiguration, lettuceClientConfiguration);
            return lettuceConnectionFactory;
        }

        @Bean
        public RedisTemplate<String,Object> redisTemplate(){
            RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
            redisTemplate.setConnectionFactory(redisConnectionFactory());
            redisTemplate.setKeySerializer(new StringRedisSerializer());
            redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
            return redisTemplate;
        }

        @Bean
        public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory){
            RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                    .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                    .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer())).entryTtl(Duration.ofMinutes(30));

            return RedisCacheManager
                    .RedisCacheManagerBuilder
                    .fromConnectionFactory(redisConnectionFactory)
                    .cacheDefaults(redisCacheConfiguration)
                    .build();

        } 
  
 ~~~

Redis Cluster 마스터와 슬레이브 연동을 모두 끝냈고, 실제로 데이터를 넣어서 테스트를 해볼 예정이다.
  
이 글은 여기서 마치며 다음에는 실제 테스트를 진행해보고 포스팅 하겠다.



참고()
---
![https://co-de.tistory.com/24](https://co-de.tistory.com/24)
