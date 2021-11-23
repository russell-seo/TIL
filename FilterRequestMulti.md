

 ## 개요
 
  - 필자는 회사에서 Spring 프로젝트를 진행하면서 클라이언트의 Request를 예외가 발생 했을 시 모두 로그로 찍어야 했다.
    하지만 모든 Controller에 try/catch문을 다루기에는 코드가 너무 난잡하여 Filter에서 request를 커스텀하여 Interceptor로 넘겨주어
    처리 할 수 있었다.
    
  - 클라이언트로 부터의 요청 정보를 담은 HttpServletRequest 객체에는 요청 Body 반환하는 getInputStream()메소드가 존재한다.
    이 메소드는 최초 1회 호출 후 재호출 시 `java.io.IOException:Stream closed` 예외를 발생시킨다.
    
    
    
 ## CustomHttpServletRequest 작성
 
  -  Custom의 목적은 요청 바디를 바이트의 배열로 보관하다가 매번 getInputStream() 요청이 올 때 마다 보관된 데이터를 반환하는 것

  ![image](https://user-images.githubusercontent.com/79154652/142986507-61d0dd81-5a56-44e8-943d-b3031f664f5c.png)
  
  
  - 위의 코드는 HttpServletRequestWrapper 를 상속 받아서 구현한다.
  
    - 먼저 부모 클래스의 생성자를 호출 한다.
    - Filter에서 받은 요청을 `request.getInputStream()`으로 처음 호출 하여 `is` 라는 변수에 저장한다.
    - 그 후 Apache 라이브러리를 사용한 IOUtils 클래스의 toByteArray()메소드를 사용해 모든 데이터를 바이트 배열로 가져온다.


![image](https://user-images.githubusercontent.com/79154652/142986539-69b53668-c8c9-4f16-8ed9-dcd721f20b35.png)


 
## Filter에 Custom한 Filter 적용하기

  - OncePerRequestFilter를 CustomFilter에 상속하여 `doFilterInternal`을 Override 하여 사용하면 된다.
 
 ![image](https://user-images.githubusercontent.com/79154652/142991235-77c00b76-c8da-4da9-8be7-d4b0400ca5ef.png)


## 인터셉터 에서 CustomFilter의 Body값 가져오기

  - 아래 코드의 IOUtils.toString 메소드를 통해서 CustomFilter를 통해 request를 인터셉터에서 다시 호출 할 수 있게 되었다.
  - request를 `param`이라는 속성으로 지정하여 DispatcherServlet 안의 비지니스 로직에서 예외가 발생하면 ExceptionHanlder에서
    처음 클라이언트가 요청한 Request 값을 로깅할 수 있게 되었다.

![image](https://user-images.githubusercontent.com/79154652/142991300-c1a880d0-ea7f-4656-8194-cd9f8d730e8a.png)
