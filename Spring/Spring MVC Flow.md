
## Spring MVC 처리 과정

   - [참고][binghe819의 TIL](https://github.com/binghe819/TIL/blob/master/Spring/MVC/Spring%20MVC%20flow.md)
    
    
   - 필자는 Spring 공부를 하면서 어노테이션을 이해하지 못하고 써왔었고, 아래의 Flow 를 보면서 Spring DispathcerServlet을 이해하기 시작했다.
    
   ![image](https://user-images.githubusercontent.com/79154652/141027404-eb8ed302-4449-41a4-82b1-3596e5fd9da0.png)


  Spring MVC 는 핵심이 DispatcherServlet 이다.
  
  1.모든 요청에 대한 서블릿 필터가 실행 됨
  
  2.모든 요청은 DispatcherServlet 에게 전달 된다.
  
  3.DispatcherServlet은 HandlerMapping 에게 위임하여 요청을 처리할 Handler(Controller) 를 찾는다.
  
    -  HandlerMapping은 요청 URL을 보고 Handler를 찾아서 Handler의 이름과 함께 반환한다.
    
    -  이때 반환되는 것은 HandlerExecutionChain 타입이다.(handler과 인터셉터 관련 상태를 가지고 있다.)
