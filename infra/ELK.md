  # ELK Stack(ElasticSearch, Logstash, Kibana + Filebeat)
  
   필자는 회사 프로젝트를 진행하면서 APM을 어플리케이션에 붙여서 모니터링을 진행해야 했기에 오픈 소스인 ELK Stack을 이용해 보았다.
   
   ELK Stack을 알아보기 전에 ElasticSearch에 대해 알아보자.
   
   1. __ElasticSearch__ 란?
   
   ElasticSearch는 Apache Lucene 기반의 Java 오픈소스 분산형 RESTful 검색 및 분석 엔진 입니다. 방대한 양의 데이터에 대해 실시간으로 저장과 검색 및 분석등의 작업을 수행할 수 있습니다.
   
   특히 정형데이터, 비정형 데이터, 지리 데이터등 모든 타입의 데이터를 처리할 수 있습니다. ElasticSearch는 `JSON 문서로 데이터를 저장하기 때문입니다.`
   
   
  2. __ELK Stack__ 란?
  
  - 기존 `ELK(ElasticSearch + Logstash + Kibana)에 FileBeats`가 들어간 형태를 말한다.

      - `ElasticSearch` 는 검색 및 분석 엔진 입니다.
      - `Logstash`는 여러 소스에서 동시에 데이터를 수집하여 변환한 후 ElasticSearch 같은 'stash'로 전송하는 서버 사이드 데이터 처리 파이프 라인
      - `Kibana`는 사용자가 ElasticSearch에서 차트와 그래프를 이용해 데이터를 시각화할 수 있게 해줍니다.
      - `Beats`는 서버의 에이전트로 설치하여 다양한 유형의 데이터를 ElasticSearch 나 Logstatsh 로 전송하는 오픈 소스 데이터 발송자입니다.
  
  기본적인 시스템 아키텍처는 아래와 같다.
  
  ![image](https://mblogthumb-phinf.pstatic.net/MjAxOTEyMDZfMTY0/MDAxNTc1NjIwNDM3MTky.9-noY7toMpRb3LJ1GH3o8Zpvp_ji2eL4vA75tkOwuhEg.jbqByZnJNqyjNlhqX-GQPOz9lE9OXJhUJqeZJPR2Le0g.PNG.ksh60706/image.png?type=w800)
  
  - Beats에는 PacketBeat, FileBeat, MetricBeat, WinlogBeat 등 여러가지 Beat가 있다. 해당 Beat들이 각자의 역할로 Metric 정보나 윈도우 이벤트, 로그파일 정보들을 수집하여 ElasticSearch나 Logstash로 전달하게 됩니다.
    
    필요 하다면 Logstash를 통해 가공 작업을 거쳐 ElasticSearch로 보내주고 이 결과를 Kibana로 보이게 됩니다.
  
  - Filebeats는 `파일에 저장된 로그 파일을 실시간으로 수집`하여 Logstash로 전달해주는 서비스 입니다.(즉, 이는 데이터가 변경되면 실시간으로 전달해준다고 볼 수 있다.)
  - 그러나 꼭 Logstash로 안보내고 Elasticsearch로 바로 보낼수도 있다.

  > file -logstatsh로 바로 읽을 수 있으면 굳이 왜 FileBeat를 쓰나?
    - filebeat는 logstash에서 파일 읽고 전송하는 기능만 있는 subset이라고 보면 되는데
    - 데이터 출처(서버)가 여러 대인 경우 각각의 서버에 logstash를 설치해 운용하기에는 낭비가 심하기 때문
    - 따라서 각 서버에는 경량 수집기 Filebeat만 설치하고, logstash는 소수의 장비에서 운용하기 위함
  
  ## ELK 구현 및 프로젝트 적용
  
  필자는 아래와 같은 아키텍처를 구현하였다.
  
   
  ![image](https://user-images.githubusercontent.com/80693904/123505606-16756780-d69b-11eb-87bc-762379c3312e.jpg)
  
  1. Spring Application 에서 log를 해당 폴더로 파일을 생성하고
  2. FileBeats에서 파일에 저장된 로그 파일을 수집합니다
  3. FileBeats -> Logstash 로 로그 파일을 전달
  4. Logstash에서 전달받은 log 파일들을 ElasticSearch로 전달
  5. Kibana에서 해당 log 파일을 모니터링 할 수 있다.

  
  
  ### 테스트 환경
  - Windows10
  - Spring
  - elasticsearch-7.10.0
  - filebeat-7.10.0
  - kibana-7.10.0
  - logstash-7.10.0
  
  
  
  1. 먼저 ElasticSearch, fileBeat, kibana, logstatsh를 설치한다.
      - [elasitcSearch 다운로드 링크](https://www.elastic.co/kr/downloads/past-releases/elasticsearch-7-10-0)
      - [filebeat 다운로드 링크](https://www.elastic.co/kr/downloads/past-releases/filebeat-7-10-0)
      - [kibana 다운로드 링크](https://www.elastic.co/kr/downloads/past-releases/kibana-7-10-0)
      - [logstash 다운로드 링크](https://www.elastic.co/kr/downloads/past-releases/logstash-7-10-0)

  2. logstash config 폴더의 logstash.conf 파일을 수정한다.
    
      - input, filter, output으로 구성된다.
      - `input` 은 filebeat에서 받을 포트를 지정한다.
      - `output`은 logstash에서 받은 파일을 elasticsearch로 parse하기 위해 설정, index는 Kibana 인덱스에서 생성될 이름이다.\
      
     ~~~java
     input{
        beats{
          port => 5044
        }
     }
    
     output{
        elasticsearch {
          hosts => ["localhost:9200"]
          index => "logstash-%{+YYYY.MM.dd}"
        }
     }
     ~~~
     
     
     
  3. Spring Application log를 수집할 filebeat의 filebeat.yml 파일 수정
    
    ~~~
    # ============================== Filebeat modules ==============================
    filebeat.inputs:
      - type: log
        enabled: true
        paths:
          - "C:/home/jhc/logs/logenLog/info/*.log"  //어플리케이션 로그가 찍히는 Path
    processors:
      - decode_json_fields:
          fields: ["message"]
          max_depth: 1
          target: "filebeat"
    # ------------------------------ Logstash Output -------------------------------
    output.logstash:
        hosts: ["localhost:5044"] //Logstash로 parse할 포트
    ~~~

  4. filebeat log를 받을 `logstash`먼저 실행한다. 명령어는 bin 폴더에서 `logstash -f ../config/logstash.conf` 를 입력
  
![image](https://user-images.githubusercontent.com/79154652/155271965-9901028b-43bc-43c0-86e1-923038ed2f02.png)
   
   console 창에 쭉 실행 되면서 위 이미지 마지막 줄 처럼 성공적으로 Logstash API 가 실행됬다고 출력된다.
   
  5. logstash를 실행한 후 filebeat도 cmd창을 열어서 실행한다. 명령어는 `filebeat -e` 이다.
  
  ![image](https://user-images.githubusercontent.com/79154652/155272389-82241933-cbc6-41d8-852b-427b98370d68.png)
  
  정상적으로 실행이 되었다면 위의 이미지 처럼 실시간으로 log를 모니터링 한다.
  
  6. 차례로 elasticsearch를 실행, bin 폴더에서 cmd창을 키고 `elasticsearch` 입력 정상적으로 실행됬는지 확인 위해 `localhost:9200` 입력
     아래와 같은 Json 데이터 출력 됨
  
  ![image](https://user-images.githubusercontent.com/79154652/155273945-802bbe47-b447-4bde-91f5-2c7305a5021b.png)

  
  7. kibana도 실행 해준다, bin 폴더에서 kibana.bat 실행
  
  8. 위 과정을 모두 수행하면 cmd창 4개가 실행되고 있으며 localhost:5601로 접속하면 아래와 같은 Elastic 홈페이지로 이동
  
  ![image](https://user-images.githubusercontent.com/79154652/155273826-05f1db83-b103-44b1-9564-95be0caf8ca2.png)

  9. 이제 Elastic의 index를 설정을 위해 메뉴에서 Management의 Stack Management 클릭
  
  10. Kibana의 index Patterns 로 진입한다. 아래와 같은 index patterns을 등록 할 수 있다.
   
   ![image](https://user-images.githubusercontent.com/79154652/155274303-eb9e74ec-8626-43c9-a964-e3c5d08c4a7d.png)

  11. 여기서 logstash 의 index 설정한  `logstash-*` 라고 입력하고 해당 날짜의 index를 생성합니다.
  
  12. Kibana의 Discover로 진입하면 아래와 같은 log를 볼 수 있다.
  
  ![image](https://user-images.githubusercontent.com/79154652/155274567-64d0efc5-64ef-4623-bfe6-d841035fb1d5.png)

  
  ### 마치며
  
  ELK Stack을 Spring 프로젝트와 연동 하였는데 현재는 info 폴더에 대한 log만 수집하고 있다. 추가로 error와 login 폴더에 있는 log도 함께 출력 할 예정이다.
  
  데이터 시각화를 위해 조금 Filter와 데이터 가공을 통해 조금 더 공부 해 볼 예정이다.
  
  참고
  ---
  [https://umbum.dev/1144](https://umbum.dev/1144)
  
  [https://parkcheolu.tistory.com/135](https://parkcheolu.tistory.com/135)
  
  [https://has3ong.github.io/spring/springaopelk2/](https://has3ong.github.io/spring/springaopelk2/)
  
