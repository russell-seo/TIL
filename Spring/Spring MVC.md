
Servlet Application 에 Spring 적용 의미

  - Spring이 제공하는 IoC컨테이너를 활용
  - Spring이 제공하는 서블렛 구현체 DispatcherServlet을 사용하겠다.
  - web.xml 파일 내 Listener 변경(기등록된 리스너를 제거하고, Spring에서 제공하는 ContextLoaderListener 등록)
  - ContextLoaderListener는 Spring IoC Container(즉, Application Context)를 Servlet Applicaiton 생명 주기에 맞춰서 바인딩 해준다.
  - Applicaiton Context를 Web Application에 등록되어있는 Servlet들이 사용할 수 있도록 Application Context를 만들어서 이 Application Context를 Servlet Context에 ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE라는 이름으로 등록해주고 Servlet Context가 종료될시점에 제거(Servlet Context : Servlet들이 사용할 수 있는 공용의 정보들을 모아두는 공간)


스프링 MVC 구조

![image](https://user-images.githubusercontent.com/79154652/138597373-ca8967ce-fee1-4f27-8c8e-06ce6510727c.png)




1. DispathcerServlet
    - 가장 앞서 요청을 받아들여 FrontController 라고 불림
    - 가장 핵심이며 모든 요청은 DispatcherServlet에게 전달된다.
    - DispathcerServlet은 HandlerMapping 에게 위임하여 요청을 처리할 Handler(Controller)를 찾는다.
    - 각 Controller에 요청을 전달하고 컨트롤러가 반환한 결과값을 view에 전달해 응답.

2. HandlerMapping
    - 클라이언트의 요청 URL을 처리할 컨트롤러를 찾아서 DispatcherServlet에 반환.
    - @Controller 어노테이션이 적용된 객체의 @RequestMapping 값을 이용해 요청을 처리할 컨트롤러 탐색


3. HandlerAdapter
    - DispatcherServlet의 처리요청을 변환해서 컨트롤러에 전달.
    - Controller의 응답 결과를 DispatcherServlet이 요구하는 형식으로 변환.

