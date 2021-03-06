
#  Redis 운영 이슈와 효율적인 관리

   [Redis Windows 다운로드](https://github.com/microsoftarchive/redis/releases)

   Redis 운영 이슈는 4가지가 있다.
   
   - 메모리 관리를 잘하자.
   - O(n) 관련 명령어를 주의하자(keys *등) -> Single Thread라 소요 시간이 큰 명령어가 들어오면 Block 된다.
   - Replication
   - 설정


   ## 그래서 Redis 운영을 위한 설정 팁 먼저 기록하겠다.
   
   - Maxclient 설정을 `50000`으로 해놓자. 기본값은 `10000` (설정한 값의 수 만큼 Client 접속 가능)
   - RDB/AOF 설정 off(persistance 기능을 끄는게 성능, 안정성이 높음. 혹시 persistance를 유지해야할 경우 Master에서는 off 시켜놓고
     Slave에서만 설정을 켜두도록 하자.
   - 특정 command는 disable 시키자. 다른건 몰라도 `Keys`를 disable 시켜야 한다.
   - maxmemory를 사용할만큼 지정한다. 1G로 놓으면 1GB만 사용하게 된다.
   - maxmemory-policy, 즉 메모리가 꽉 찼을 때의 정책을 성정한다. 사실 이럴 일이 없게 60~70% 정도 차면 migration을 해주는 것이 맞다. allkeys-lru로 설정하면 가장 최근에 입력 받은 값을 사용하고, 전의 값은 지운다.
   - 적절하게 ziplist 자료구조를 사용하도록 하자.


   ## Redis 설정
   
   설정 파일을 변경하는 방법은 redis 설치 디렉토리 안의 redis.conf 파일을 변경하고 재구동 하면 된다.
   
   
   1. 기본 설정
      
      레디스의 가장 기본적인 설정들인 네트워크, 로그 및 프로세스 관련 설정을 할 수 있다.
      
      
      - `port` : 인스턴스가 사용할 포트, 기본 값은 6379
      - `bind` : 인스턴스가 사용할 네트워크 설정, 사용할 아이피 설정 (bind 0.0.0.0 -> 모든 ip 허용)
      - `timeout` : 연결된 클라이언트의 idle 대기 시간 설정을 초 단위로 한다. 해당 시간동안 송 수신이 발생하지 않으면 클라이언트의 연결을 끊난다. `0` 으로 설정하면 사용하지 않음
      - `loglevel` : 인스턴스 동작 중에 출력하는 로그 레벨을 지정함(debug/verbose,notice,warning 중 선택 가능)
      - `logfile` : 로그가 저장되는 경로와 파일명을 지정함, logfile ./log/redis.log
      - `database` : redis에서 제공하는 논리적으로 분리된 데이터 저장 공간이다. RDBMS의 스키마 개념이라고 보면 된다. 각각의 데이터베이스는 숫자로 구분 된다. 이 값을 저장하지 않으면, 기본값으로 16이 사용되며, 16개로 분리된 데이터 저장 공간을 사용한다.
         `config get databases` 명령어 입력 시 DBIndex가 출력되고 접속중인 데이터 베이스를 확인 할 수 있다.
         
   2. 스냅 샷 설정
      
      레디스는 메모리의 데이터를 영구 저장하기 위해 dump.rdb 파일에 메모리의 내용을 기록한다.
      
      스냅샷 이벤트가 발생하면 메모리의 내용을 dump.rdb에 저장한다.
      
      스냅샷 이벤트에 의해서 dump.rdb 파일에 저장 중에 오류가 발생하면 redis는 모든 쓰기 요청을 거부한다.
      
      - `save` : 스냅샷 이벤트가 발생 하는 주기를 설정한다. save 단위시간(초) 키 변경회수로 저장한다. ex) save 300 10 ( 300초 또는 키 변경이 10개 발생하면 스냅샷 이벤트 발생), 만약 save 300 10, save 900 1 이런 식으로 여러개 설정 시에는 그 중 하나라도 만족 하면 스냅샷 이벤트가 발생한다.
      - `stop-writes-on-bgsave-error` : 스냅샷 요청에 의해서 dump.rdb에 내용을 저장하는 중 쓰기 오류가 발생 할 경우, 쓰기 요청을 에러 없이 처리하고 싶다면 이 파라미터를 NO로 설정하면 된다. `NO로 설정 할 경우 쓰기 오류가 발생하는 것을 감지 할 수 없으므로 yes로 사용하는 것이 좋다`
      - `rdbcompression` : 스냅샷 파일을 저장 할 때 파일의 압축 여부를 설정. 압축을 하게 되면 dump.rdb 파일의 크기는 줄어 들지만, 작업 시 CPU 사용률을 높아진다.
      - `dir` : 레디스의 작업 디렉토리를 지정 한다. redis가 생성하는 파일의 기본 위치 사용됨.
      - `dbfilename` : 스냅샷 파일의 파일명을 지정하는 설정. 기본은 dump.rdb로 되어 있으나, 서비스 포트 또는 노드이름.rdb 같은 형식으로 사용하고자 할 경우 지정해 주면 된다.
   
   3. AOF(Append Only File) 설정
   
      AOF 파일은 Redis에서 발생 한 모든 데이터의 변경 이력을 저장한다.
      
      쓰기 명령어가 실행 될 때 마다 AOF 파일에 명령과 데이터를 기록하여 장애에서 데이터를 복원하는데 사용한다.
      
      사용자가 잘못해서 flushall 명령을 수행했을 경우, Redis를 종료 후 AOF 파일을 열어서 flushall 명령을 삭제하고 레디스를 재 기동 시키기만 하면 된다.
      
      - `appendonly` : AOF 기능의 사용 여부를 설정한다. 기본값은 NO 이며 필요한 경우 yes로 설정한다.
      - `appendfilename` : AOF파일명을 지정 함. 필요 시 경로까지 설정해 줄 수 있다. 해당 위치에 파일이 없거나 생성 실패 시에 AOF 파일이 없다는 오류 메시지를 출력하고 인스턴스 시작이 실패 한다.
      - `appendfsync` : 데이터를 dump.rdb 또는 AOF 파일에 쓸 때, fsync() 함수를 이용해서 버퍼의 내용을 디스크에 즉시 기록하도록 할지 여부를 결정 한다. 쓰기가 필요 할 때 마다 fsync() 함수가 호출되면 데이터 유실 할 위험은 줄어들지만, 상대적으로 파일 기록 성능이 저하 될 수 있다.
           - no : fsync() 함수를 호출 하지 않는다. OS로 쓰기 명령을 전달하고, O/S에서 알아서 파일을 저장하도록 한다. 가장 빠른 성능을 제공하지만, O/S에 쓰기 명령을 전달하고 다 쓰기 전에 서버에 문제가 생기면 쓰여지지 않는 부분에 대한 손실이 발생할 수 있다.
            
           - always : 각 명령어를 AOF 파일에 기록하고 나서 항상 fsync()함수를 호출한다. 가장 느린 성능을 제공하지만 데이터 안정성 측면에서는 가장 좋다.
           
           - everysec : 매 초마다 fsync() 함수를 호출한다. 성능과 안정성의 타협점으로 볼 수 있으면 __레디스의 기본 설정이다__
     
   4. Replication 설정
      
      마스터 노드에서는 requirepass부분만 설정 하면 되고, 아래 부분은 Slave에서 설정 한다.
      
      - `slaveof` : 마스터 노드의 네트워크 위치를 지정한다. slaveof <마스터노드IP> <마스터노드 PORT>
      - `masterauth` : 마스터 노드가 requirepass 설정에 의해서 패스워드로 보호되고 있을 때, 슬레이브 노드에서 설정하며, 마스터 노드의 requirepass 설정에 지정된 패스워드를 설정 한다.
      - `slav-serve-stale-data` : Slave 노드에서 Master 노드에 대한 네트워크 연결이 끊어지거나, Master 노드의 전체 데이터를 복제 중 일때 Slave 노드로 들어오는 요청을 어떻게 처리할 지 결정
               - yes : Slave 노드에서 읽거나 쓰기 요청을 모두 처리함.
               - no : 에러 메시지를 전달 함
      - `slave-read-only` : Slave를 read-only 로 운영 함, 쓰기 명령만 에러가 발생하고, slaveof나 config 같은 관리자 명령을 사용 가능
      - `repl-ping-slave-period` : Slave 노드에서 설정된 간격으로 Master 노드에 ping 명령을 수행한다. 기본은 10초 이다.
  
  5. 제한 설정
     
     레디스 서버가 사용 할 수 있는 최대 값들 설정, 주로 메모리에 관한 설정 들이며 복제나 스냅샷 사용시 숙지 해야 한다.
     
     - `maxclient` : Redis 인스턴스에 접속 할 수 있는 클라이언트 수를 지정한다. 기본값은 1024로 된다.
     - `maxmemory` : Redis 인스턴스가 데이터를 저장하기 위하여 사용할 메모리 크기를 지정한다. 이 값보다 많은 데이터를 저장하면 maxmemory-policy 설정에 지정된 값에 따라서 레디스의 동작이 달라 진다.
     - `maxmemory-policy` : 레디스에서 저장한 데이터가 maxmemory를 넘을 경우, 메모리 정리를 어떻게 할지에 대한 정책을 지정해 준다.
         - volatile-lru : (기본값) 만기시각이 설정된 key 들 중에서 LRU 알고리즘에 의해 key를 골라 삭제
         - allkeys-lru : LRU algorithm 에 의해 key를 골라 삭제
         - volatile-random : 만기시각이 설정된 key 들 중에서 랜덤하게 key를 골라 삭제
         - allkeys-random : 랜덤하게 key를 골라 삭제
         - volatile-ttl : 만기시각이 설정된 key들 중에서 만기시각이 가장 가까운 key를 골라 삭제
         - noeviction : 어떤 key 도 삭제하지 않고 error on write operations 를 돌려준다.
     - `maxmemory-samples` : maxmemory-policy 를 적용하기 위해서 레디스가 조회할 키의 개수를 지정함
        전체 키를 읽어서 삭제 policy에 해당하는 키를 찾는게 아니라, 지정한 값 만큼의 임의의 키를 읽어서 그 중에서 삭제 대상인 키가 있는지 확인한다. 이때 임의로 읽어 들일 키의 개수를 지정 함.
        
        

