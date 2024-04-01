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
