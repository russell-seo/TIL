# Spring Security OAuth2

  Spring Security OAuth2를 이용하여 구글, 네이버, 애플 소셜 로그인을 구현했으며, 워낙 오랜기간 동안 고생했고 많이 찾아봐서 기록해놓고 추후에 보기 위해 이 글을 쓸려고 한다.
  
  
  필자는 먼저 구글 소셜 로그인에 대한 흐름을 이해하기 위해 블로그에서 Flow의 흐름을 잘 정리 해놓은 그림을 가져왔다. 아래 이미지를 보면 Spring Security OAuth2 에 대해 이해하기 더 쉬웠다.
  
  ![image](https://blog.kakaocdn.net/dn/bhP40i/btqIKqwlE9g/5RLgQYGe2zM79ex2Td7iRK/img.png)
  
  
  ## OAuth 2.0
  
   - 기본적으로 OAuth(OpenID Authentication)란, 타사의 사이트에 대한 접근 권한을 얻고 그 권한을 이용하여 개발할 수 있도록 도와주는 프레임워크이다. 구글, 카카오, 네이버등과 같은 사이트에서
    로그인을 하면 직접 구현한 사이트에서도 로그인 인증을 받을 수 있도록 되는 구조이다.
    
   물론 구글에서 로그인을 했다고 해서 개발한 웹 사이트에 구글 ID와 PW를 그대로 전달해주면 안되므로 Access Token을 발급받고, 그 토큰을 기반으로 원하는 기능을 구현해야 한다.
   
   Access Token은 로그인을 하지 않고 인증을 할 수 있도록 해주는 인증 토큰 정도의 개념이다. 유저 A가 직접 개발한 웹사이트 에서 자신의 구글 캘린더에 대한 접근을 허용해준 다면, Access Token을 통해
   해당 정보 권한을 받아올 수 있어서 그정보를 토대로 캘린더에 글을 작성하고 삭제하는 등의 작업을 할수 있게 된다.
   
   ### 용어
   
   여기서 Access Token을 발급 받기 위한 일련의 과정들을 인터페이스로 정의해둔 것이 바로 OAuth 이다. OAuth에서 중요한 용어는 크게 세 가지다.
   
   - `Resource Owner` : 개인 정보의 소유자를 가리킨다. 유저 A가 이에 해당한다.
   - `Client` : 제 3의 서비스로부터 인증을 받고자 하는 서버다. 직접 개발한 웹사이트가 이에 해당한다.
   - `Resource Server` : 개인정보를 저장하고 있는 서버를 의미한다. 구글, 카카오 등이 이에 해당한다.


  유저가 구글에서 제공해주는 서비스를 이용하는 셈으로 타 사의 서비스를 이용하기 위해서는 신청해야 한다.
  
  - `Client ID` : `Resource Server`에서 발급해주는 ID. 웹 사이트 에 구글이 할당한 ID를 알려주는 것이다.
  - `Client Secret` : `Resource Server` 에서 발급해주는 PW. 웹 사이트 에 구글이 할당한 PW를 알려주는 것이다.
  - `Authorized Redirect Uri` : `Client` 측에서 등록하는 Url. 만약 이 Uri 로 부터 인증을 요구하는 것이 아니라면, `Resource Server`는 해당 요청을 무시한다.
  
  
 __동작 설명__
 
 Client 즉 직접 개발한 웹 사이트 가 `Resource Server`에 등록이 완료 되었다면, 이제 Access Token을 발급 받을 수 있다. 구글을 예시로 설명하자면 
 유저가 웹 사이트 에서 특정기능을 이용하기 위해서 로그인이 요구 되고 구글 인증 Access Token을 받기 위해서 구글 로그인 링크로 연결된다.
 
 예시 링크(https://accounts.google.com/?client_id=123&scope=profile,email&redirect_uri=http://localhost) 의 쿼리 스트링을 살펴보면 client_id는 123, scope는 profile과 email, redirect_uri는 http://localhost임을 알 수 있다. 
 
 유저가 구글 로그인을 정상적으로 했다면, 구글은 이전에 등록되었던 `client_id = 123`인 서버의 redirect_uri와 동일한지 확인한다.
 
 일치하는 경우, 유저 에게 `scope=profile, email`기능을 넘겨 줄 것인지에 대한 승인 여부를 물어보고, 동의한다면 구글은 이에 해당하는 `Authorization_code`라는 임시 PW를 발급한다.
 
 이후 http://localhost/?authorization_code=2 로 리다이렉트 되며, 웹사이트의 서버는 이 `authorization_code`를 가지고 구글에게 Access_token을 요청한다. 그리고 유저의 인증이 필요할 떄 마다 Access Token을 이용하여 접근한다.
 
 
__Spring Security Config__

~~~java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
@Override
    protected void configure(HttpSecurity http) throws Exception {

            http.httpBasic()
                .disable()
                .cors().and()
                .csrf()
                .disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .anonymous()
                .disable()
                .authorizeRequests()
                .antMatchers( "/oauth2/**, /login/**").permitAll()
                .antMatchers("/test/**").permitAll()
                .antMatchers("/token").permitAll()
                .anyRequest().authenticated()
                .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
                .oauth2Login()
                .successHandler(successHandler)
                .failureHandler(failureHandler)
                .userInfoEndpoint()
                .userService(customOAuthService);
    }
    
}
~~~

- `@EnableWebSecurity` 는 스프링 시큐리티 설정들을 활성화 시켜준다.
- `http.authorizeRequest()` 로 URL별 권한 관리를 설정할 수 있다.
    - `antMatcher()`는 권한 관리 대상을 지정한다. `permitAll()`, `hasRole()` 등으로 권한 설정이 가능하다.
    - `anyRequest()`는 설정된 값들 이외 나머지 Url들을 나타낸다. `authenticated()`로 인증된 사용자만 접근 가능하도록 설정 가능하다.
    - `addFilterBefore`는 설정한 UsernamePasswordAuthenticationFilter 전에 Token을 검증하는 Filter를 먼저 검증하게 끔 설정가능하다.

- `oauth2Login()`로 OAuth2 로그인 관련 처리를 설정할 수 있다.
    - `userService()`는 OAuth2 인증 과정에서 `Authentication`을 생성에 필요한 OAuth2USer를 반환하는 클래스를 지정한다.
    - `successHandler()` 는 인증 성공 후 처리할 클래스를 지정한다.
    - `faulureHandler()` 는 인증을 실패할 경우 예외처리를 할 클래스를 지정한다.



__OAuth2 인증 과정__

1. 사용자가 소셜 로그인을 정상적으로 완료

2. `AbstractAuthenticationProcessingFilter` 에서 OAuth2 로그인 과정을 호출

3. `Resource Server`에서 넘겨주는 정보를 토대로 `OAuth2LoginAuthenticationFilter`의 `attemptAuthentication()` 에서 인증 과정을 수행

4. `attemptAuthentication()` 처리 과정에서 `OAuth2AuthenticationToken`을 생성하기 위해 `OAuth2LoginAuthenticationProvider`의 `authenticate()`를 호출

5. `authenticate()` 처리 과정에서 `OAuth2USer`를 생성하기 위해 `OAuth2UserService`의 `loadUser()`를 호출

    
   `OAuth2UserService`의 기본 구현체는 `DefaultOAuth2UserService`이지만, 커스텀한 `OAuth2User`를 반환하도록 구현하고 싶었으므로 직접 구현한 `CustomOAuth2UserService`의 `loadUser()`가 호출됨
  
  
 ~~~java
@RequiredArgsConstructor
@Service
public class CustomOAuthService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {

    private final UserJpaRepository userJpaRepository;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService<OAuth2UserRequest, OAuth2User>  delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration()
                .getProviderDetails()
                .getUserInfoEndpoint()
                .getUserNameAttributeName();
        


        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());
        User user = userJpaRepository.findByEmail(attributes.getEmail());
        if(user == null){
            saveNewUser(attributes);
        }

        Map<String, Object> stringObjectMap = attributes.convertToMap();


        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority("ROLE_USER")),
                stringObjectMap,
                "email");

    }
 
 ~~~
