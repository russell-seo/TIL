
## @ControllerAdvice & @RestControllerAdvice

  - Controller 전역에서 예외처리 하기.
    
    최근 API 서버 개발을 하다보니 개발해야 하는 API수가 너무 많고 모든 메소드에 try/catch를 쓰기에는
    코드가 너무 난잡하여 컨트롤러 전역에서 Exception을 handling 하기 위해 @ControllerAdvice를 사용 하였다.
    
    @ControllerAdvice 는 AOP 방식이고 Application 전역에서 발생하는 모든 컨트롤러의 예외를 한 곳에서 처리하게 할 수 있다.
    ![image](https://user-images.githubusercontent.com/79154652/140484482-34c174c5-b63d-4081-b2f8-cae597611610.png)
    
    위의 코드에서 @RestControllerAdvice를 사용했는데 이는 return 값이 Json으로 반환되며 괄호안에 범위를 지정 할 수 있다.
