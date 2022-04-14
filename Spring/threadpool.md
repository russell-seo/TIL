  
  
  # Spring Thread Pool
  
  스프링 부트 다중 처리 요청에 대해 너무 잘 정리되어있는 블로그를 보고 나 또한 기록해 놓기로 하였다.
  
  
  ![image](https://user-images.githubusercontent.com/79154652/163090670-c12289be-1996-49ca-bc84-d6b0b903d5a3.png)



  __스프링 부트가 다중 처리 요청을 처리하는 것이 아닌, 내장되어있는 Servlet Container(Tomcat)에서 다중요청을 처리한다.__
  
  - 스프링 부트는 내장 서블릿 컨테이너인 `Tomcat`을 사용한다.
  - Tomcat은 다중 요청을 처리하기 위해서, 부팅할 때 Thread의 컬렉션인 Thread Pool을 생성한다.
  - 유저 요청(HttpServletRequest)가 들어오면 Thread Pool에서 하나씩 Thread를 할당한다. 해당 Thread에서 스프링부트에서 작성한 Dispathcer Servlet을 거쳐 유저 요청 을 처리한다.
  - 작업을 모두 수행하고 나면 Thread는 Thread Pool로 반환한다.
 
 
 
 ## Spring Boot와 내장 Tomcat
 
 Spring Boott는 `application.yml` 혹은 `application.properties`에 설정을 주는 것만으로 간편하게 Tomcat의 설정을 바꾸어 줄수 있다.
 
 
    # application.yml (적어놓은 값은 default)
    server:
      tomcat:
        threads:
          max: 200 # 생성할 수 있는 thread의 총 개수
          min-spare: 10 # 항상 활성화 되어있는(idle) thread의 개수
        max-connections: 8192 # 수립가능한 connection의 총 개수
        accept-count: 100 # 작업큐의 사이즈
        connection-timeout: 20000 # timeout 판단 기준 시간, 20초
      port: 8080 # 서버를 띄울 포트번호
 
 
 해당 디폴트 값은 `org.springframework.boot.autoconfigure.web.ServerProperties` 클래스에서 확인 할 수 있다.
 
 
 ## Thread Pool 설정
 
 위의 예시로 들어둔 yml 파일에 적어놓은 값들이, Tomcat `ThreadPoolExcutor` 와 `Connector`에 줄수 있는 옵션들 이다.
 
 __Thread Pool__ 이란?
 
 프로그램 실행에 필요한 Thread 들을 미리 생성해놓는다는 개념, CPU의 자원을 이용하여 코드를 실행하는 하나의 단위이다.
 
 Tomcat 3.2 이전 버전에는 유저의 요청이 들어올 때 마다 Servlet을 실행할 Thread를 하나씩 생성, 요청이 끝나면 destory 했고 이 방침은
 두가지 문제를 야기 했다.
 
 > 1. 모든 요청에 대해 Thread를 생성하고 소멸하는 것은 OS와 JVM에 대해 많은 부담을 안겨준다.
 > 2. 동시에 일정 이상의 다수 요청이 들어올 경우 리소스(CPU와 메모리 자원) 소모에 대한 억제가 어렵다. 순간적으로 서버가 다운되거나 동시다발적
    인 요청을 처리하지 못해서 생기는 문제가 야기 된다.
    
 `해당 문제 해결을 위해 Tomcat 은 Thread Pool을 사용하기 시작`
 
 ![image](https://user-images.githubusercontent.com/79154652/163099709-1833646c-b7a6-4584-b652-d786f7247f6d.png)


  Thread Pool Flow
  
  1. 첫 작업이 들어오면 `core size`만큼의 쓰레드를 생성한다.
  2. 유저 요청(Connection, Server socket에서 accept 한 소캣 객체)이 들어올 떄 마다 작업 큐(queue)에 담아둔다.
  3. core size의 Thread 중, 유휴상태(idle)인 Thread가 있다면 작업 큐(queue)에서 작업을 꺼내 Thread에 작업을 할당하여 작업을 처리한다.
      3-1. 만약 유휴상태인 Thread 가 없다면 작업은 큐(queue)에서 대기한다.
      3-2. 그 상태가 지속되어 작업 큐(queue)가 꽉 찬다면 Thread를 새로 생성한다.
      3-3. 3번의 과정을 반복하다 `Thread Max size`에 도달하고 작업큐(queue)도 꽉 차게 되면 추가요청에 대해선 connection-refused 오류를 반환한다.
  4. Task가 완료되면 Thread 는 다시 유휴상태로 돌아간다.
      4-1. 작업큐(queue)가 비어있고 core size이상의 Thread가 생성되어있다면 Thread를 Destory한다.
      
 `한줄요약 : Thread를 미리 만들어 놓고 필요한 작업에 할당했다가 돌려 받는다`
 
 > Thread는 많으면 너무 많은 Thread 가 cpu의 자원을 두고 경합하게 되므로 처리속도가 느려질 수 있고, 적으면 cpu자원을 최적으로 활용하지
    못해 마찬가지로 처리속도가 느려질 수 있다. Thread는 적절한 수로 유지되는 것ㅅ이 가장 좋다.
  
  
  
  ## ThreadPoolExecutor
  
  위에 설명한 ThreadPool을 자바에서 구현한 구현체가 ThreadPoolExecutor 이다. 앞서 application.yml에서 주었던 설정중 일부이다.
  
  
      server:
        tomcat:
         threads:
          max: 200 # 생성할 수 있는 thread의 총 개수
          min-spare: 10 # 항상 활성화 되어있는(idle) thread의 개수
         accept-count: 100 # 작업 큐의 사이즈
  
  이 두가지 설정은 `Thread Max size 및 core size`를 변경할 수 있도록 해준다. Tomcat9.0의 디폴트 옵션은 각각 200개, 25개 인데 스프링
  부트에선 200개, 10개를 디폴트 값으로 잡았다.
  
  accept-count는 작업큐의 사이즈 이다. Spring boot에선 아무 옵션을 안주면 Integer.max, 즉 21억 블라블라를 줬다. 이는 무한 대기열 전략으로
  아무리 요청이 많이 들어와도 core size를 늘리지 않는다는 정책이다. 무한 대기열 전략에선 작업큐가 꽉 찰 일이 없으므로, Thread pool Max 사이즈가
  의미가 없다.
  
  > 지금까지 알아본 바에 의하면, 유저 요청이 들어올 때(Connection) 마다 쓰레드가 하나씩 할당될것 이고, 작업큐가 가득차면 쓰레드가 늘어날 것이고, 쓰레드도 가득차면
    유저 요청이 거절된다. 하지만 이는 `BIO Connector(Blocking I/O)` 일떄 유효한 이야기 이다. 그러나 톰캣 8.0부터 `NIO(NonBlocking I/O) Connector`이 기본으로 채택되고
    9.0 부터는 BIO Connector가 deprecate 됨 으로써 위의 설며과는 다른 방식으로 진행되게 된다.


  ## Connector
  
  Connector는 소켓 연결을 수입하고 데이터 패킷을 획득하여 HttpServletRequest 객체로 변환하고, Servlet 객체에 전달하는 역할을 한다.
  
  1. Acceptor에서 while문으로 대기하며 port listen을 통해 Socket Connection을 얻게 된다.
  2. Socket Connection으로부터 데이터를 획득한다. 데이터 패킷을 파싱해서 HttpServletRequest 객체를 생성한다.
  3. Servlet Container 에 해당 요청객체를 전달합니다. ServletContainer는 알맞은 서블릿을 찾아 요청을 처리한다.


  ### BIO Connector
  
  BIO Connector는 Socket Connection을 처리할 때 Java의 기본적인 I/O 기술을 사용한다. Thread Pool에 의해 관리되는 Thread는 소켓 연결을 받고 요청을 처리하고 요청에 대해 응답한 후
  소켓 연결이 종료되면 Pool에 다시 돌아오게 된다.
  즉, Connection이 닫힐 때 까지 하나의 Thread는 특정 Connection에 계속 할당되어 있을 것입니다.
  
  이러한 방식으로 Thread를 할당하여 사용 할 경우, 동시에 사용되는 Thread 수가 동시 접속할 수 있는 사용자의 수가 될 것 입니다. 그리고 이러한 방식을 채택해서 사용할 경우
  Thread들이 충분히 사용되지 않고 idle(아무것도하지않는) 상태로 낭비되는 시간이 많이 발생합니다. 이러한 문제점을 해결하고 리소스를 효율적으로 사용하기위해 `NIO Connector`가 등장했다.
  
  
  ### NIO Connector
  
  NIO Connector는 I/O가 아니라 Http11NioProtocol을 사용하는데 해당 내용에 대해 이해하기 위해선 NIO에 대해 이해해야 한다.
  
  ![image](https://user-images.githubusercontent.com/79154652/163323263-4859051b-9a84-44b2-a269-693d097e2bf0.png)

  
  NIO Connector에선 Poller 라고 하는 별도의 쓰레드가 커넥션을 처리한다. Poller는 Socket들을 캐시로 들고 있다가 해당 Socket에서 data에 대한 처리가 가능한 순간에만
  Thread를 할당하는 방식을 사용해서 Thread가 idle 상태로 낭비되는 시간을 줄여준다.
  
  ![image](https://user-images.githubusercontent.com/79154652/163324086-4c3847fb-d2ec-4d35-86e4-125a4b2c419d.png)

  Acceptor는 이름 그대로 Socket Connection을 accept합니다. ServerSocket.accept() 방식을 사용하고 있습니다. 소켓에서 Socket Channel 객체를 얻어서 톰캣의 NioChannel 객체로
  변환합니다. 그리고 추가로 NioChannel 객체를 PollerEvent라는 객체로 한번 더 캡슐화해서 event queue에 넣게 됩니다. Acceptor는 event Queue 공급자, Poller Thread는 event Queue의 사용자입니다.
  
  ![image](https://user-images.githubusercontent.com/79154652/163324358-c89cf583-8b6c-4ace-85f3-d15256cf5101.png)

  Poller는 NIO의 Selector를 가지고 있습니다. Selector에는 다수의 채널이 등록되어 있고, Select 동작을 수행하여 데이터를 읽을 수 있는 소켓을 얻습니다. 그리고 Worker Thread Pool에서 이용할 수 있는 Worker Thread를 얻어서 해당 소켓을 worker Thread에게 넘기게 된다.
  
  Java Nio Selector를 사용해서 data 처리가 가능할 때만 Thread를 사용하기 때문에 idle상태로 낭비되는 Thread가 줄어들게 된다.
  
  Poller에선 Max Connection 까지 연결을 수락하고, 셀렉터를 통해 채널을 관리하므로 작업큐 사이즈와 관계없이 추가로 커넥션을 refuse하지 않고 받아놓을 수 있습니다.
  
  요약
  ---
  NIO 기반의 Connector는 하나의 Connection이 하나의 쓰레드를 할당받는 BIO Connector에 비해, Selector를 활용해 Socket을 관리하므로 더 적은 쓰레드를 사용합니다. 또한 Max-Connections 값까지 접속을 유지하고, 쓰레드가 모자라면 max 사이즈 까지 쓰레드를 추가한다. time-wait 시간 안에 처리가 가능하다면 처리할 수 있습니다.
  
  참고
  ---
  [https://velog.io/@sihyung92/how-does-springboot-handle-multiple-requests](https://velog.io/@sihyung92/how-does-springboot-handle-multiple-requests)
  
  
  
