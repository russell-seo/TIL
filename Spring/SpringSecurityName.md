

## Architecture
  
  용어정리
  
  스프링 시큐리티에서는 `인증` 과 `권한`을 분리하여 체크할 수 잇도록 구조를 만들었다.
    
   - Authentication(인증) : A 라고 주장하는 주체(user, subject, principal) 가 A가 맞는지 확인 하는 것.
      - 코드에서 `Authenitication` : 인증 과정에서 사용되는 핵심 객체
         - ID/PASSWORD, JWT, OAuth 등 여러 방식으로 인증에 필요한 값이 전달되는데 이것을 하나의 인터페이스로 받아 수행하도록 추상화 하는 역할의 인터페이스 이다.
   - Authorization(권한) : 특정 자원에 대한 권한이 있는지 확인하는 것
      - `인증`(Authentication) 을 거치고 인증이 되었으면 `권한`(Authorization) 이 있는지 확인 후, 서버 자원에 대해서 접근 할 수 있게 되는 순서
   
   - Credential(증명서) : 인증 과정 중, 주체가 본인을 인증하기 위해 서버에 제공하는 것(ID/PASSWORD)
