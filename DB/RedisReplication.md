
# Redis Replication

  - Redis를 구성하는 방법 중에서 Read 분산과 데이터 이중화를 위한 Master/Slave 구조가 있습니다. Master 노드는 쓰기/읽기를 전부 수행하고,
    Slave는 읽기만 가능합니다. 이렇게 하려면 Slave는 Master 의 데이터를 전부 가지고 있어야 한다. 이럴 때 발생하는 것이 Replication 입니다.
    
    `Replication은 마스터에 있는 데이터를 복제해서 Slave로 옮기는 작업입니다. Slave가 싱크를 받는 작업은 다음과 같다.`
    
    ### Master-Slave간 Replication 작업 순서
    1. Slave Configuration 쪽에 "replicaof<master IP><master PORT>" 설정을 하거나 REPLICAOF 명령어를 통해 마스터에 데이터를 Sync 요청한다.
    
    2. Master는 백그라운드에서 RDB파일(현재 메모리 상태를 담은 파일) 생성을 위한 프로세스를 진행합니다. 이 때 Master는 fork를 통해 메모리를 복사한다.
      이후에 fork한 프로세스에서 현재 메모리 정보를 디스크에 덤프 뜨는 작업을 진행한다.
    
    3. 2번 작업과 동시에 Master는 이후부터 들어오는 쓰기 명령들을 Buffer에 저장해 놓는다.
    4. 덤프 작업이 완료되면 Master는 Slave에 해당 RDB파일을 전달해주고, Slave는 디스크에 저장한 후에 메모리로 로드한다.
    5. 3번에서 모아두었던 쓰기 명령들을 전부 Slave로 보내준다.
  
    > Master가 fork하는 부분에서 자신이 쓰고 있는 메모리만큼 추가로 필요해진다. 따라서 Replication 을 할 때 OOM(Out of Memory)이 발생하지
      않도록 주의하는 것이 중요하다. 성공적으로 Replication을 마쳤다고 하더라도 개선할 점은 아직 남아 있다. 바로 Master 노드가 죽게되는 시나리오 이다.
  
    Master가 죽은경우에는 Slave는 마스터를 잃어버리고 Sync 에러를 낸다. 이상태에서는 쓰기는 불가능하고 읽기만 가능하다. 따라서 Slave를 Master로
  승격 시켜야 한다. 이런 작업을 매번 장애마다 할 수는 없기 때문에 다양한 방법을 통해서 failover에 대응할 수 있다. 
  하나 예를 들면 DNS기반으로 failover에
  대응 할 수 있다. Client는 Master 도메인을 계속 바라보게 한 후에, 만약 마스터에 장애가 발생하면 Master DNS를 Slave에 매핑한다. 그러면 Client는 특별한
  작업 없이도 Slave쪽으로 붙게 된다.
  
  ## Redis Cluster
  
  Redis Cluster는 failover를 위한 대표적인 구성방식 중 하나이다. Redis Cluster는 여러 노드가 Hash 기반의 Slot을 나눠가지면서 클러스터를 구성하여
  사용하는 방식이다. 전체 slot은 16384이며 Hash 알고리즘은 CRC16을 사용한다. Key를 CRC16으로 해시한 후에 이를 16384로 나누면 해당 key가 저장될
  slot이 결정된다.
  
 ![image](https://user-images.githubusercontent.com/79154652/152170312-454f759b-e065-4bea-a6e6-8ab8ea7a249c.png)
  
  Cluster를 구성하는 각 노드들은 master노드로, 자신만의 특정 slot Range를 갖습니다. 다만 데이터를 이중화하기 위해서 위에서 설명한 Slave 노드를 가질
  수 있다. 즉, 하나의 클러스터는 여러 Master 노드로 구성할 수 있고, 한 Master 노드가 여러 Slave를 가지는 구조이다. 만약 특정 Mastser노드가
  죽게되면, 해당 노드의 Slave중 하나가 Master로 승격하여 역할을 수행하게 된다.
  
  
