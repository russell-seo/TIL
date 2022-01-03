
# JWT(Json Web Token)

  
  ## 서버 기반 인증 vs 토큰 기반 인증
  
  특정 사용자가 서버에 접근 했을 떄, 이 사용자가 인증된 사용자인지 구분하기 위해서는 여러 방법을 사용할 수 있다.
  
  대표적으로
  
  - 서버 기반 인증
  - 토큰 기반 인증

  JWT는 토큰 기반 인증에 속한다.

  ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbguaA1%2Fbtq6RHmor9l%2FC31oMkkgTWTdW79nrakP1K%2Fimg.png)
  
  - 위 그림은 서버와 클라이언트 간의 JWT 인증 방식이다.

  1. 클라이언트가 ID, Password로 서버에 로그인을 요청한다.
  2. 서버는 ID, Password를 통해 유효한 사용자 인지 검증하고, 유효한 사용자인 경우 `토큰을 생성해서 응답`한다.
  3. 클라이언트는 토큰을 저장해두었다가, `인증이 필요한 api에 요청할 때 토큰 정보와 함께 요청` 한다.
  4. 서버는 토큰이 유효한지 검증하고, 유효한 경우에는 응답 해준다.
