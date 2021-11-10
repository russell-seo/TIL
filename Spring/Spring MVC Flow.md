
## Spring MVC 처리 과정

   - [참고][binghe819의 TIL](https://github.com/binghe819/TIL/blob/master/Spring/MVC/Spring%20MVC%20flow.md)
    
    
   - 필자는 Spring 공부를 하면서 어노테이션을 이해하지 못하고 써왔었고, 아래의 Flow 를 보면서 Spring DispathcerServlet을 이해하기 시작했다.
    
   ![image](https://user-images.githubusercontent.com/79154652/141032269-7edf5021-44da-4059-bdff-0071f64f1719.png)



  Spring MVC 는 핵심이 DispatcherServlet 이다.
  
  1.모든 요청에 대한 서블릿 필터가 실행 됨
  
  2.모든 요청은 DispatcherServlet 에게 전달 된다.
  
  3.DispatcherServlet은 HandlerMapping 에게 위임하여 요청을 처리할 Handler(Controller) 를 찾는다.
  
    -  HandlerMapping은 요청 URL을 보고 Handler를 찾아서 Handler의 이름과 함께 반환한다.
    
    -  이때 반환되는 것은 HandlerExecutionChain 타입이다.(handler과 인터셉터 관련 상태를 가지고 있다.)
  
  4. DispathcerServlet은 찾아낸 Handler를 실행할 수 있는 HanderAdapter를 찾는다.
  
  5. 찾아낸 HandlerAdapter를 사용 Handler를 실행(invokeHandlerMethod)
      - Handler를 실행하면서 Service, Repository, DB 등 비즈니스 로직을 수행.
      - Handler의 리턴 값 : view(뷰 파일명), Model(데이터)


  6. DispatcherSevlet은 ViewResolver에게 view 의 이름을 전달하고 view 객체를 얻는다.
      
      - view 이름에 해당하는 view 를 찾음.
      
      - view Resolver는 전략 객체이며 view name 뿐 아니라 헤더 정보도 전달.
      
      - view Resolver는 전달된 정보를 바탕으로 사용자에게 보여줄 view 가 무엇인지 결정.
  
  
  7. DispatcherServlet 은 View 객체에게 Model 과 함께 화면 표시를 의뢰한다.

  8. View 는 해당하는 뷰를(ex. JSP, Thymleaf..) 호출하며, Model 객체에서 화면 표시에 필요한 정보를 가져와 화면 표시를 처리한다.
      
      - 찾은 뷰에 모델 데이터를 랜더링하고 요청의 응답값을 생성한다.
 
  9. DispatcherServlet 은 View 로부터 받은 결과를 클라이언트에게 반환한다.

 
 중요 Point : 
 
 DispathcerServlet의 doDispatch 메소드를 보면 try-catch 구문이 있음.
   
   - Controller로 들어오는 예외는 catch로 잡아주기 때문에 Controller에서 try-catch문을 쓰는 것은 불필요한 코드.
   
   - 추가로 Service, Repository 에서 발생하는 예외는 Spring에서 모두 Controller로 위임하여 처리하게 됨
   
   - ExceptionHandler를 만들어서 Controller에서 전역으로 예외처리 하는 것이 코드 가독성에 좋음.


![image](https://user-images.githubusercontent.com/79154652/141032689-989e00fe-2aa3-45c4-81b0-7fbbb84f5c4c.png)
![image](https://user-images.githubusercontent.com/79154652/141032746-ea23fb29-af5e-4b81-bdee-cc6f20aabcf8.png)



