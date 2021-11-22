
## AOP, Filter, Interceptor 의 차이

![image](https://user-images.githubusercontent.com/79154652/142804318-ca040b1a-5d80-4851-a423-a9015b56e025.png)


- Spring 으로 개발을 하다가 보면 `공통적으로 처리해야 할 업무`가 많다. 로그인, 토큰처리, 로그처리등 많은 업무들이 공통적 모든 프로젝트에 적용 되어야 한다.
  하지만 모든 코드에 적용하다보면 코드가 난잡하고 가독성이 떨어지는 코드가 될 가능성이 높다.
  
  그래서 Tomcat, Spring은 공통 업무를 따로 분리하여 처리하는 기능을 제공한다.
  
  1. Filter - WAS(Tomcat) : Servlet 단위에서 실행
  2. Interceptor - Spring : Servlet 단위에서 실행
  3. AOP - Spring : 메소드 앞에 Proxy 패턴의 형태로 실행


  ### AOP, Filter, Intetceptor 흐름
  
  - 서버를 실행시켜 Servlet이 올라오는 동안 init()이 실행되고, 그 후 doFilter가 실행된다.
  - Controller로 들어가기전에 preHandler가 실행된다.
  - Controller에서 나와 postHandler, after Completion, doFilter 순으로 진행 된다.
  - Servlet 종료 시 destory가 실행된다.
  
  
  ## Filter
  
    - WAS로 들어온 요청을 Servlet으로 보내고 또 Servlet이 작성한 응답을 클라이언트로 보내기 전에 특별한 처리가 필요한 경우에 사용되는 필터.
    - 체인 형태의 구조
    - 클라이언트 요청 정보를 제공하는 SerlvetRequest 객체를  정의함.
    - Servlet 컨테이너는 ServletRequest 객체를 생성하고 이를 Servlet service() 메소드에 인수로 전달.


  1. 사용하는 이유
      
      - Servlet 실행 전, 후 에 어떠한 작업을 하기 위해 사용된다.
      - Servlet에서 반복적으로 수행해야 하는 작업을 공통으로 처리할 수 있다는 장점.
        ex)데이터 암호화, 복호화(JWT), 문자 인코딩, 디코딩, 로그
        
  2. 메소드
      
      - init() : 필터 객체가 생성되고 준비 작업을 위해 딱 한번 호출된다.
      - doFilter() : 필터와 매핑된 URL에 요청이 들어올때 마다 doFilter()가 호출 된다.
          - filterChain 은 다음 필터를 가리키고 `filterChain.doFilter()`는 다음 필터를 호출한다.
          - 다음 필터가 없으면 내부적으로 서블릿의 service()가 호출된다.
      - destory() - WAS가 종료되기 전에 딱 한번 호출된다.

      $`서블릿이 실행되기 전의 작업은 filterChain.doFilter() 이전 코드에 써야한다.`
      
      $`서블릿이 실행된 후 응답하는 작업은 filterChain.doFilter() 이후 코드에 써야한다.`
      
  3. Filter 배치 
    - 필터를 배치 하는 방법은, web.xml, 자바 어노테이션 두가지 이다.

![image](https://user-images.githubusercontent.com/79154652/142807185-c7750b00-b491-480e-8822-a55b0497e562.png)

![image](https://user-images.githubusercontent.com/79154652/142807206-cbfaa8ad-54bc-4685-9298-d8e445d644ff.png)



## Intetceptor


  - 요청에 대한 작업 전/후를 가로챈다. Filter는 스프링 컨텍스트 외부에 존재, Interceptor는 스프링의 DispatcherServlet이 컨트롤러를 호출하기 전, 후로 끼어들기 때문에 
    내부에서 Controller에 관한 요청과 응답에 대한 처리를 하며 스프링의 모든 빈 객체에 접근 가능.
    `Interceptor`는 여러 개를 사용할 수 있다.
    
     1.preHandler() - 컨트롤러 메소드가 실행되기 전
     2.postHandler() - 컨트롤러 메소드 실행 직 후 view 페이지 렌더링 되기 전
     3.afterCompletion() - view 페이지가 렌더링 되고 난 후
    
![image](https://user-images.githubusercontent.com/79154652/142807737-bfacede4-cb51-41a8-991f-c8b2bab3307b.png)

![image](https://user-images.githubusercontent.com/79154652/142807771-7d6da361-cd7e-4e11-9a75-b5db6a9780b8.png)



## AOP

  - OOP를 보완하기 위해 나온 개념
  - 객체 지향의 프로그래밍을 했을 때 중복을 줄이는 기능을 제공.
  - `비지니스 단의 메서드에서 조금 더 세밀하게 조정 할 때 사용한다.`
  
  Interceptor와 Filter와 달리 메소드 전 후 의 지점에 자유롭게 설정이 가능하며 주소, 파라미터, 어노테이션 등 다양한 방법으로 대상을 지정 할 수 있다.
  반면 HandlerInterceptor는 Filter와 유사하게 HttpServletRequest, HttpServletResposne를 파라미터로 사용한다.
  
      - @Before : 대상 메소드의 수행 전
      - @After : 대상 메소드의 수행 후
      - @After-returning : 대상 메소드의 정상적인 수행 후
      - @After-throwing : 예외 발생 후
      - @Around : 대상 메소드의 수행 전/후
   
   
# 결론
  
#### Filter
    - 전체적인 request단 에서 처리가 필요 할 떄.
    - URL 및 기타 정보를 캐시하는 Filter
    
#### Interceptor
    - 세션 및 쿠키 체크하는 Http프로토콜 단위로 처리해야 하는 업무.
    - 로그인 토큰 체크 등..
    
#### AOP
    - 비지니스 단에서 세밀하게 조정하고 싶을 떄
    - 로깅, 트랜잭션, 에러처리 
