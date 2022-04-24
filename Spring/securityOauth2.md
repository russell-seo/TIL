# Spring Security OAuth2

  Spring Security OAuth2를 이용하여 구글, 네이버, 애플 소셜 로그인을 구현했으며, 워낙 오랜기간 동안 고생했고 많이 찾아봐서 기록해놓고 추후에 보기 위해 이 글을 쓸려고 한다.
  
  
  필자는 먼저 구글 소셜 로그인에 대한 흐름을 이해하기 위해 블로그에서 Flow의 흐름을 잘 정리 해놓은 그림을 가져왔다. 아래 이미지를 보면 Spring Security OAuth2 에 대해 이해하기 더 쉬웠다.
  
  ![image](https://blog.kakaocdn.net/dn/bhP40i/btqIKqwlE9g/5RLgQYGe2zM79ex2Td7iRK/img.png)
  
  
  ## Spring Security
  
   - 스프링 시큐리티에서 애플리케이션 보안을 구성하는 두 가지 영역에 대해 간단하게 설명해보려 한다. 인증(Authentication)과 인가(Authorization). 두 영역은 사실상 스프링 시큐리티의 핵심이다.


  
