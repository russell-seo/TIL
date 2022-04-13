  
  
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
  
  
  
  참고
  ---
  [https://velog.io/@sihyung92/how-does-springboot-handle-multiple-requests](https://velog.io/@sihyung92/how-does-springboot-handle-multiple-requests)
  
  
  
