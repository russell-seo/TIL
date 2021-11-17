
  ## Spring Exception 처리 과정
  
   - 스프링 처리과정을 보면 예외가 발생하는 부분은 크게 두가지로 나뉜다.
      
      1. Dispatcher Servlet 내에서 발생하는 예외(Controller, Service, Repository 등)
      2. Dispathcer Servlet 전의 서블릿(Filter)에서 발생하는 예외


  ### HandlerInterCeptor
  
  ![image](https://user-images.githubusercontent.com/79154652/141954385-dbc4b2d7-493b-44d6-9e03-f49c2aa9c868.png)

  
   - HandlerInterceptor 단 은 Handler(Controller) 수행 전에 DispatcherServlet 에게 Request를 위임하여 처리하는 객체.
    
   - @ControllerAdvice 는 Handler(Controller) 단에서 발생하는 Exception을 @ExceptionHandler Anntation으로 처리하는 로직 이므로 Exception Handling 이 불가능 한 상황
   
   - HandlerInterceptor 단 Exception Handling
     
     1. Handler(Controller)에서 발생한 Exception을 Handling 하기 위해서는 HandlerExceptionResolver의 Exception Handling 전략을 사용 해야 함.
  
 #### HandlerExceptionResolver 활용
 
  - Handler(Controller) 나 그 뒤 계층에서 throw 된 Exception은 DispathcerServlet 이 전달 받은 뒤 다시 servlet 밖의 Servlet container가 처리하게 된다.
  
  - HandlerExceptionResolver가 등록되어 있지 않으면 HTTP Status 500 내부 서버 오류 와 같은 메시지가 브라우저에 노출 됨. 하지만 HandlerExceptionResolver가 등록 되어 있으면
    DispatcherServlet은 먼저 HandlerExcpetionResolver가 해당 Exception을 Handling 할 수 있는 지 확인하고 Exception Handling 이 가능한 경우 HandlerExceptionResolver에게 
    `Exception Handling` 을 위임한다.
