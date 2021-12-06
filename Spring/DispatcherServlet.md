

# DispatcherServlet

  ## Why?
  
  
  
  ## DispathcerServlet 구조
  
  이 서블릿도 부모 클래스에서 HttpServlet 을 상속받아 사용 하며 `서블릿` 으로 동작한다.
  
     DispathcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
  
  스프링 부트 구동시 DispathcerServlet 을 서블릿으로 자동등록하며 모든 경로(urlPatterns="/")에 대해 매핑한다. 즉 Spring MVC 역시 FrontController 
  패턴으로 구현되어 있고 DispatcherServlet 이 FrontController 의 역할을 한다고 생각하면 된다.
  
   ![이미지](https://oopy.lazyrockets.com/api/v2/notion/image?src=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F01763a91-c0a1-4688-b15e-573a3458b5f4%2FUntitled.png&blockId=3da6bc6e-44b2-4ad6-b00c-60402a5c507d)
                                                          DispathcerServlet 을 보면 결국 HttpServlet을 상속받아 사용 한다.
                                                          
                                                          
   
   ## 요청 흐름
   
   1. Servlet이 호출되면 HttpServlet이 제공하는 service() 메소드가 호출된다.
   2. 스프링 MVC는 DispathcerServlet 부모인 FrameworkServlet에서 service()를 오버라이드 해뒀다.
   3. FrameworkServlet.service()를 시작으로 여러 메소드가 실행되며 DispathcerServlet.doDispatch()가 호출된다.


  ## doDispatch 메소드
  1. 핸들러 조회 - 아래 코드는 doDispatch의 `핸들러 조회` 코드이다.
  
  `mappedHandler = getHandler(processedRequest);
  
  => 요청에 맞는 적절한 핸들러를 찾아 반환해준다.`
  
  `noHandlerFound(processedRequest, response);
  
  => 적절한 핸들러를 찾지 못한경우 404 에러코드를 반환해준다.`
  
  ![image](https://user-images.githubusercontent.com/79154652/144574187-93bf0014-1db7-4de5-abbc-b608367a2a4f.png)
   
   
  2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
  
  `HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
  
  => 찾은 핸들러를 처리할 수 있는 핸들러 어댑터를 찾아준다.
  
  => 만약 찾지 못할 경우 ServletException 발생`
  
  
  `mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
  
  => 찾은 핸들러 어댑터를 이용해 로직을 수행하는 handle 메소드를 호출한다.
  
  => 결과로 ModelAndView 를 반환받고 이를 이용해 렌더링 까지 수행된다.`
  
  ![image](https://user-images.githubusercontent.com/79154652/144574879-8816761c-7c34-46d3-8a9d-70fce920e11d.png)

  
   
   
  3. 뷰 렌더링 호출

  `processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
  
  => 실제 코드는 복잡하게 되있는데 결국 render() 메소드를 호출한다.
  
  => render()에서 ModelAndView에서 View를 찾아 ViewResolver를 이용해 뷰의 물리적 이름을 완성해서 forward 해준다.`
  
  ![image](https://user-images.githubusercontent.com/79154652/144770712-4f9b0426-a5a1-4d08-aad8-868b77685695.png)

  
  ![image](https://user-images.githubusercontent.com/79154652/144770768-9947b820-9f11-4eab-bea5-c43b81e4a0f3.png)

  
  ![image](https://user-images.githubusercontent.com/79154652/144771311-c00c1952-74e4-4cd8-93ad-7f401fdaf750.png)

   
  
