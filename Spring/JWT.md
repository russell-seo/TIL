
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


 ## 토큰 사용 방식의 특징
 
  1. 무상태성
      - 사용자의 인증 정보가 담겨있는 토큰을 클라이언트에 저장하기 때문에 서버에서 별도의 저장소가 필요 없다, 완전한 무상태를 가질 수 있다.
        이로 인해 서버 확장에 용이하다.
        
  2. 확장성
      - 토큰 기반 인증을 사용하는 다른 시스템에 접근이 가능하다.(ex. 카카오로그인, 구글로그인등)

  3. 무결성
      - HMAC(Hash-based Message Authenication) 기법 이라고도 불리며 발급후 토큰 정보를 변경하는 행위가 불가능 하다. 즉 토큰이 변조되면 바로 알아 차릴 수 있다.


  4. 보안성
      - 클라이언트가 서버에 요청을 보낼 때, 쿠키를 전달하지 않기 때문에 쿠키의 취약점은 사라진다.



  ## JWT 란?
  
  토큰 기반 인증 시스템의 대표적인 구현체이다.
  ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbE1ajd%2Fbtq6SvTtY7B%2FdfBjF0kq3E7GJg4LB2gi90%2Fimg.png)
  
  `JWT`는 `.`를 기준으로 헤더(Header), 내용(payload), 서명(signature)로 이루어져 있다.
  
  
  ### 헤더(Header)
  
  `헤더는 토큰의 타입과 해싱 알고리즘을 지정하는 정보를 포함한다`
  
  - typ : 토큰 타입을 지정한다. `JWT`라는 문자열이 들어가게 된다.
  - alg : 해싱 알고리즘을 지정한다.
  ~~~java
  {
    "typ" : "JWT"
    "alg" : "HS256"
  
  }
  ~~~
  위 코드는 , JWT 토큰으로 이루어져 있고, 해당 토큰은 HS256으로 해싱 알고리즘으로 사용되었다.
  
  ### 정보(payload)
  
  `토큰에 담을 정보가 들어간다`, `정보의 한 덩어리를 클레임(claim)이라고 부르며, 클레임은 key-value 의 한 쌍으로 이루어져 있다` 클레임 종류는 세 종류로 나눌 수 있다.
  
  - 등록된(registered) 클레임
     - 토큰에 대한 정보를 담기 위한 클레임들이며, 이미 이름이 등록되어 있는 클레임
        - `iss` : 토큰 발급자(issuer)
        - `sub` : 토큰 제목(subject)
        - `aud` : 토큰 대상자(audience)
        - `exp` : 토큰의 만료시간(expiraton), 시간은 NumericDate 형식으로 되어있어야 하며, 항상 현재시간 보다 이후로 설정되어있어야 한다.
        - `nbf` : Not Before 을 의미하며, 토큰 활성 날짜와 비슷한 개념, NumericDate 형식으로 날짜를 지정하며, 이 날짜가 지나기 전 까지는 토큰 처리가 되지 않는다.
        - `iat` : 토큰이 발급된 시간(issued at)
        - `jti` : JWT의 고유 식별자로서, 주로 일회용 토큰에 사용한다.
  
  - 공개(public) 클레임
     - 말 그대로 공개된 클레임, 충돌을 방지할 수 있는 이름을 가져야 하며 , 보통 클레임 이름을 URI로 짓는다.
  
  - 비공개(private) 클레임
     - 클라이언트 -> 서버간 통신을 위해 사용되는 클레임

  -> 예제 Payload
   
  ~~~java
  {
      "iss" : "kiaofk@naver.com", //등록된 클레임
      "iat" : 1622370878, //등록된 클레임
      "exp" : 1622372678, //등록된 클레임
      "https://XXX.com/jwt_claims/id_admin" : true, //공개 클레임
      "email" : "kiaofk@naver.com", //비공개 클레임
  }
  
  ~~~
  
  ### 서명(signature)
  
  해당 토큰이 조작되었거나 변경되지 않았음을 확인하는 용도로 사용, `헤더(Header) 의 인코딩 값과 정보(payload) 의 인코딩 값` 을 합친 후에 주어진 비밀키를 통해 해쉬값을 생선한다.
  
  ~~~java
  HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
  ~~~
  
 
