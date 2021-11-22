
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
  
