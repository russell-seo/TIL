

# DispatcherServlet

  ## Why?
  Dispatch의 의미는 파견, 급파하다의 의미로 해석해보면, 받은 요청을 어딘가로 빨리빨리 보내는 서블릿 이라는 뜻이다.
  
  Spring이 없는 JAVA 런타임에서는 컨트롤러가 존재하지 않는다. 따라서 우리는 서블릿 객체를 생성하고 그것을 web.xml에 일일히 다 등록해줘야 했다. 아래 코드와 같이
  
  ![image](https://user-images.githubusercontent.com/79154652/144779432-ba644478-d2d1-4df3-b283-2395092527ff.png)

  웹 사이트를 이용해 봤다면 우리가 접속하는 페이지(경로)는 한 두개가 아니다. 프로젝트 문서는 온통 서블릿 객체로 넘쳐 날 것이며, Servlet 객체
  
  는 HttpServlet을 확장한 객체이다. 이렇게 되면 HttpServlet 기능을 필수로 Override 해야하고 더이상 일반 객체로 사용할 수 없다.
  
  이러한 불편함은 DispatcherServlet이 나타남에 따라 Spring MVC 모든 요청을 받아 처리를 하게 됨.
  
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
  
  `mappedHandler = getHandler(processedRequest);`
  
  `=> 요청에 맞는 적절한 핸들러를 찾아 반환해준다.`
  
  `noHandlerFound(processedRequest, response);`
  
  `=> 적절한 핸들러를 찾지 못한경우 404 에러코드를 반환해준다.`
  
  ![image](https://user-images.githubusercontent.com/79154652/144574187-93bf0014-1db7-4de5-abbc-b608367a2a4f.png)
   
   
  2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
  
  `HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());`
  
  `=> 찾은 핸들러를 처리할 수 있는 핸들러 어댑터를 찾아준다.`
  
  `=> 만약 찾지 못할 경우 ServletException 발생`
  
  
  `mv = ha.handle(processedRequest, response, mappedHandler.getHandler());`
  
  `=> 찾은 핸들러 어댑터를 이용해 로직을 수행하는 handle 메소드를 호출한다.`
  
  `=> 결과로 ModelAndView 를 반환받고 이를 이용해 렌더링 까지 수행된다.`
  
  ![image](https://user-images.githubusercontent.com/79154652/144574879-8816761c-7c34-46d3-8a9d-70fce920e11d.png)

  
   
   
  3. 뷰 렌더링 호출

  `processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);`
  
  `=> 실제 코드는 복잡하게 되있는데 결국 render() 메소드를 호출한다.`
  
  `=> render()에서 ModelAndView에서 View를 찾아 ViewResolver를 이용해 뷰의 물리적 이름을 완성해서 forward 해준다.`
  
  ![image](https://user-images.githubusercontent.com/79154652/144770712-4f9b0426-a5a1-4d08-aad8-868b77685695.png)

  
  ![image](https://user-images.githubusercontent.com/79154652/144770768-9947b820-9f11-4eab-bea5-c43b81e4a0f3.png)

  
  ![image](https://user-images.githubusercontent.com/79154652/144771311-c00c1952-74e4-4cd8-93ad-7f401fdaf750.png)

   
  ## DispatcherServlet 구성요소
  
   ![이미지](https://github.com/binghe819/TIL/raw/master/Spring/MVC/image/image-20200924010336407.png)
   
   
   ### MultipartResolver
   
   - 사용자의 파일 업로드 요청에 대한 처리를 하는 인터페이스
   
   - HttpServletRequest 를 MultipartHttpServletRequest 로 변환해주어 요청을 담고 있는 File을 꺼낼 수 있는 API 제공
  
  
   ### LocaleResolver
   
   - 요청하는 클라이언트의 위치(Locale)정보를 파악하는 인터페이스.
      
      - 지역 정보를 읽어서 적절한 언어를 선택할 때 사용된다.
      - 헤더값의 `accept-language`를 읽고 처리하는 것이 기본 값

   - 기본 전략으로는 요청의 accept-language를 보고 판단하는 `AcceptHeaderLocaleResolver` 가 사용된다.


  ### ThemeResolver
  
   - 애플리케이션에 설정된 테마를 파악하고 변경할 수 있는 인터페이스
      
        - 웹 브라우저에서 버튼 누르면 테마를 바꾸는 것을 Spring 에서 자체적으로 해주는 것.
        - 즉, View에게 css 파일을 넘겨줘서 테마를 바꿀 수 있게 해준다.

  ### HandlerMapping
  
   - 요청을 처리할 핸들러를 찾는 인터페이스
   
      - 즉, 요청을 처리할 Controller를 찾는 인터페이스
      
      ![이미지](https://github.com/binghe819/TIL/raw/master/Spring/MVC/image/image-20200924142551245.png)
      
   - `BeanNameUrlHandlerMapping` - 빈의 이름을 기반으로 핸들러를 찾아준다.(클래스가 핸들러가 됨)
   - `RequestMappingHandlerMapping` - 어노테이션 기반 핸들러를 찾아준다.(메세드가 핸들러가 됨)
   
  
  ### HandlerAdapter
  
   - `HandlerMapping` 이 찾아낸 핸들러를 처리하는 인터페이스
   - 스프링 MVC 확장력의 핵심
        - 개발자가 핸들러를 커스텀하여 사용 가능
        
           ![이미지](https://github.com/binghe819/TIL/raw/master/Spring/MVC/image/image-20200924143043743.png)
        
        - `RequestMappingHandlerAdapter`  - 어노테이션 기반으로 찾아낸 핸들러를 처리해준다.

  ### HandlerExceptionResolvers
  
  - 요청 처리중에 발생한 에러 처리하는 인터페이스

  ### RequestToViewNameTranslator
  
   - 핸들러에게 뷰 이름을 명시적으로 리턴하지 않는 경우, 요청을 기반으로 뷰 이름을 판단하는 인터페이스
  
  
  ### ViewResolver
    
   - 뷰 이름에 해당하는 뷰를 찾아내는 인터페이스
  
  ### FlashMapManager
  
   - `FlashMap` 인스턴스를 가져오고 저장하는 인터페이스
   - `FlashMap` 은 주로 리다이렉션을 사용할 때 요청 매개변수를 사용하지 않고 데이터를 전달하고 정리할 때 사용한다.
   
      - URL에 매개변수를 사용하지 않고 보낼 수 있게 해준다.
      - 즉, 리다이렉션을 할 때 서버에서 클라이언트에게 파라미터를 보낼 때 URL 파라미터를 사용하지 않아도 파라미터를 보낼 수 있게 해준다.


 ## 마치며
 DispathcerServlet은 Spring 을 사용 할떄 가장 중요한 핵심 Front Controller이며 그냥 막연하게 쓰는게 아니라 `왜` 쓰는지에 대해
 학습하고 공부하게 된다면 문제가 생겼을 때 빠른 대처가 가능할 것이라고 생각한다.
