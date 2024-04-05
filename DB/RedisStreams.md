# Redis Stream Event-Driven

  ## 개요 및 문제점
 
  필자는 현재 게임 채팅서버를 개발하고 있으며 채팅 백오피스 -> 채팅 서버로 기존에 PUB/SUB 기반의 이벤트 처리방식으로 구현되어 있다.

  PUB/SUB 같은 경우 1개의 Channel을 Subscribe 하고 있는 N대의 서버가 모두 데이터를 받게 되어있다. 즉 1대의 서버에서 처리되어야할 이벤트가 N대의 서버에서 처리될 수 있다.

  기존에는 N대의 서버중 1대에만 Flag 값을 지정해놓고 해당 서버로만 이벤트를 발행했다. 만약 이 서버가 내려가게 된다면 백오피스 -> 채팅서버로 이벤트 처리가 불가능하며 DB의
Flag 값을 변경하거나 이 1대의 서버가 다시 올라올때 까지 이벤트 처리가 불가능하다는 문제점이 있다.

  ## Redis Streams

  최근 많이 사용되고 있는 Event-Driven 의 대표적인 라이브러리인 Kafka와 Redis Stream를 고민하다 Redis Stream를 적용해 보기로 하였다.

  그 이유는 현재 채팅서버에서 Redis가 사용되고 있으며, Kafka는 다시 라이브러리를 추가하고 학습해야하는 시간이 소요된다고 생각하여 Redis Stream를 간단히 테스트해보고 도입하기로 하였다. 이제 간단하게 Redis Stream에 대해서 알아보고 갈려고 한다.

  Redis Stream 은 Redis 5.0부터 추가된 자료 구조로, `log file` 처럼 `append only` 로 저장되는 구조를 가지고 있다.

  메시징 시스템인 Kafka와 비슷하게 동작하며 메시지를 읽는 Consumser와 여러 Conumser를관리하는 `Consumer Group`이 있다.

  Kafka와 비교하여 쉽게 보면 Kafka의 topic은 Redis 의 Stream, Kafka와 동일하게 Producer와 Consumer가 존재, 다중 Consumer를 관리하기 위해 Consumer Group을 지원

  Redis PUB/SUB 과 비교하면 , Redis Stream은 메시지 전달 후 사라지지 않고, 수신여부를 구분지을 수 있다, 

  ![image](https://github.com/russell-seo/TIL/assets/79154652/91fad988-c539-4550-8378-a21b2b216352)

  ---

  ## Redis Streams 기본 명령어

  
  - `XADD`
    -  Stream에 데이터 추가
    -  Append only 한 자료구조 답게 `XADD` 커맨드를 통해 Stream에 데이터를 추가한다.
    -  메시지의 id는 * 로 자동 생성가능하며 권장된다. 자동생성된 id는 <millisecondsTime>-<sequenceNumber> 의 조합으로 되어있다.
    -  time-series저장소로도 볼 수 있다.

    ~~~
     > XADD <stream-key> <message-id> <field> <name> ... <field> <name>
     > XADD FUNSDK * user-id jake tx-amount 1000
     1518951480106-0
    ~~~

  - `XRANGE`
    - Range를 통한 조회 `start`와 `end id`만 있으면 된다.
    - `-`는 최소 id, `+`는 최대 id를 의미하며, count 옵션을 통해 개수 제한을 두고 조회 가능하며 `ASC` 순으로 조회된다.
    - `XRANGE`의 시간복잡도는 `log(n)`이다.
   
  ~~~
  > XRANGE <stream-key> <start-id> <end-id> [COUNT <count-num>]
  > XRANGE FUNSDK - + 
  1)  1) ...
      2) 1)...
  ~~~


- `XREAD`
  - Subscribe 형식으로 접근할 때는 `XREAD` 커맨드를 사용한다.
  - `XREAD`를 통한 조회 + Redis STreams의 특징은 여러 consumer들을 가질 수 있고, 새로운 아이템은 데이터를 기다리는 모든 consumer에게 전달
  - Redis PUB/SUB은 fire & forget으로 메시지 발생 후 저장되지 않는 형태, Blocking List를 사용할땐 client 가 메시지를 받으면 popped, 하지만 Redis Streams은 모든 메시지는 append되는 형태로, 다른 Consumer들은 마지막으로 받은 메시지 id를 기억하는걸로 새로운 메시지가 온걸 알 수 있다.
  - Stream Consumer Group은 PUB/SUB 혹은 Blocking List가 가질수 없는 여러 레벨을 제공한다.
    - `ACK`를 통해 해당 Consumer가 해당 메시지를 처리한다는 것을 명시
    - `ACK`가 오지 않은 메시지들을 `Pending`아이템을 관리할 수 있다.
    - Pending 된 아이템들을 `CLAIM`을 통해 다른 Consumer가 처리할 수 있다.


  ~~~
  > XREAD <COUNT> <count-num> STREAMS <stream-key> <start-id>
  > XREAD STREAMS FUNSDK 0
  ~~~

  - Non-Blocking 형태의 XREAD STREAMS FUNSDK 0 은 `0-0`보다 큰 id를 가진 모든 메시지를 가져온다.

  ![image](https://github.com/russell-seo/TIL/assets/79154652/45a7544d-58d7-47ae-a439-d9a55ce67e62)


- `XDEL`
  - `XDEL`을 통해 특정 메시지를 삭제할 수 있다. 발생한 데이터를 저장하고 time-series 형태로 관리할 수 있는 것이 Redis Stream 의 장점이고, 후에 나오는 `XACK`와 `XCLAIM`을 통해 메시지를 관리할 수 있다.
 
  ~~~
  > XDEL <stream-key> <message-id>
  > XDEL FUNSDK 1711944192112-0
  ~~~

  ## Consumer Groups

  - 필자가 Redis Streams 를 사용할려는 이유는 바로 이 Consumer Groups 때문이다. 보통 PUB/SUB 같은 경우 fan-out 의 형태로 모든 Consumer 들이 동일한 메시지를 받게 받게 되어 있는데 이 `Consumer Groups은 여러 Consumer를 가진 그룹이지만 이론상 하나의 Consumer를 등록하여 데이터를 가져가게 할 것 이다.
 
  - Consumer가 메시지를 Consuming 했으면 `ack`를 통해 명시해줘야 한다. Redis 는 `ack`된 메시지를 처리 되었다고 생각하여 Consumer Group 으로 부터 제외 시킨다.
  - Consumer Group은 Pending 중인 모든 메시지를 찾을 수 있다. Pending 된 메시지는 아직 처리되지 않은 `ack` 되지 않은 메시지를 의미한다.
 
### Redis Stream 메시지 상태

- `DELIVERED` : Consumer Group 이 Stream으로부터 메시지를 읽어와 그룹 내 Consumer에게 전달된 상태
- `PENDING`: Delivered 되었지만 Consumer가 아직 메시지를 처리하지 못한 대기 상태
- `ACK`: Consumer가 메시지를 처리하고 `XACK`커맨드를 통해 메시지의 처리완료를 명시한 상태


### Consumer Group 만들기


~~~
> XGROUP CREATE <stream-key> <group-name> <start-id> [MKSTREAM]
> XGROUP CREATE FUNSDK FUNSDK_GROUP $ MKSTREAM
~~~

`XGROUP CREATE`를 통해 Consumer Group 을 생성할 수 있으며, 동시에 `MKSTREAM`옵션을 통해 스트림이 없으면 스트림 생성도 가능하다.

`start-id`설정 또한 조정가능하다.


### Redis Stream 조회

Consumer를 설정하고 Subscription 이 자동으로 돌아가는 시스템을 구축했다면, Redis Stream 세부정보를 `XINFO`로 알아볼 수 있다.

~~~
> XINFO STREAM <stream-key>
> XINFO STREAM FUNSDK
~~~

![image](https://github.com/russell-seo/TIL/assets/79154652/b28549ba-a6bb-4c9d-a451-a54bb89ec409)


### Consuemr Group 조회
~~~
> XINFO GROUPS <stream-key>
> XINFO GROUPS FUNSDK
~~~

![image](https://github.com/russell-seo/TIL/assets/79154652/ad5b1714-5036-4ea7-a90c-2b9d524ae9ce)



---


## Spring Boot 에서 Redis Streams 구현

Spring Boot 에서 Redis Streams 사용하기 위한 설정

- Spring Boot 2.4.0 이상
- Spring Boot starter data redis 2.4.0 이상
- Redis 5.0 이상
- Client로 Lettuce 사용


 ## 
