
# Spring Security 인증처리 흐름과 구조

스프링 시큐리티는 수 많은 클래스 및 인터페이스들로 이루어져 있다. 이 클래스들은 인증환경을 구성하기 위한 각각의 역할이 있으며, 인증처리 역시 관련 시큐리티 클래스들이 적절하게 호출되어 구현된 것이다.

## Spring Security 란?

  __스프링 시큐리티에서는 `인증` 과 `권한`을 분리하여 체크할 수 잇도록 구조를 만들었다.__
    
   - Authentication(인증) : A 라고 주장하는 주체(user, subject, principal) 가 A가 맞는지 확인 하는 것.
      - 코드에서 `Authenitication` : 인증 과정에서 사용되는 핵심 객체
         - ID/PASSWORD, JWT, OAuth 등 여러 방식으로 인증에 필요한 값이 전달되는데 이것을 하나의 인터페이스로 받아 수행하도록 추상화 하는 역할의 인터페이스 이다.
   - Authorization(인가) : 특정 자원에 대한 권한이 있는지 확인하는 것
      - `인증`(Authentication) 을 거치고 인증이 되었으면 `권한/인가`(Authorization) 이 있는지 확인 후, 서버 자원에 대해서 접근 할 수 있게 되는 순서
   
   - Credential(증명서) : 인증 과정 중, 주체가 본인을 인증하기 위해 서버에 제공하는 것(ID/PASSWORD)


  <p align = center>
  <img src = https://media.vlpt.us/images/qotndus43/post/8d342792-848f-4078-9c74-b3acd32e8c91/image.png>
  </p>
  
  Spring Security는 `서블릿 필터`를 기반으로 동작한다.
  
  서플릿 필터는 서블릿이 호출 되기 전 전처리를 하거나 서블릿 응답 이후 후처리 작업을 할 수 있다.
  
  그러나 `서블릿 컨테이너`는 빈을 인식하지 못하기 때문에 스프링 빈으로 등록된 `Filter Bean`들을 실행할 수 없다.
  
  <p align = center>
  <img src= https://media.vlpt.us/images/qotndus43/post/df134816-a636-4a1a-b37f-41d4fd69b01e/image.png>
  </p>
  
  따라서 스프링 프레임워크는 서블릿 필터의 구현체 `DelegatingFilterProxy`를 제공합니다. 업무를 위임하다 라는 뜻을 가진 Delegate의미 처럼 직접 처리를 하는 것이 아닌 해당 필터를 등록하면 `ApplicationContext`에서 `Filter Bean`들을 찾아 실행합니다.
  
  <p align = center>
  <img src = https://media.vlpt.us/images/qotndus43/post/75720d89-0234-4bc8-8057-8c3710e3a16f/image.png>
  </p>
  
  `FilterChainProxy`는 Spring Security 가 제공하는 Filter로 `Spring Security의 중심점` 이라고 할 수 있다. FilterChainProxy 또한 Bean 이기 때문에 DelegatingFilterProxy로 감싸져 있다.
  
  `FilterChainProxy`는 SecurityFilterChain을 통해 여러 Filter 인스턴스로 작업을 위임할 수 있다.
  
  고유한 설정을 가진 여러개의 SecurityFilterChain을 두고 FilterCHainProxy 설정을 통해 URL마다 SecurityFilterChain을 맵핑할 수 있습니다. 특정 요청에 대해 스프링 시큐리티가 무시하길 바란다면, SecurityFilterChain에 보안 Filter를 0개 설정할 수 있습니다.
  
  
  ## SecurityContextHolder
  
  <p align = center>
  <img src = https://media.vlpt.us/images/franc/post/94ae1f24-267e-4ba9-b461-10ed9514bf27/sec2-2.png>
  </p>
  
  __1. SecurityContextHolder__
      - Security가 최종적으로 제공하는 객체
         - 인증에 대한 정보는 `Authentication`객체에 있다.
         - 하지만 Security는 최종적으로 `SecurityContextHodler`를 통해 이를 제공한다.
      - `Thread-local`
          - `SecurityContextHodler`는 `SecurityContext`객체를 Thread-local로 제공, 같은 쓰레드 에서는 매개로 주고 받지 않아도 언제든지 인증 정보에 접근 할 수 있다.
           
  
  
  __2. Authentication__
      - 실질적으로 인증정보를 담고 있는 객체
      - Authentication 객체의 구성
      
          - `Principal`
              - 내가 누구인지에 대한 정보를 담은 객체
              - 로그인_ID에 해당하는 값을 담고 있다.
              
          - `Credentials`
              - 인증 자격에 대한 정보를 담은 객체
              - 비밀번호와 같은 암호 값을 담고 있다.
              
          - `Authorities`
              - 현재 유저의 권한(ROLE)정보를 담은 객체
              
              
  ~~~java
  @Service
public class xxxService {

    /**
     * SecurityContextHolder를 통해 인증정보가 공유되는 지 테스트
     */
    public void user() {
    	// SecurityContextHolder를 통해 인증정보 가져오기
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        // authentication의 인증정보 가져오기 (principal, authorities)
        Object principal = authentication.getPrincipal();
        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();

        // 인증여부 가져오기
        boolean authenticated = authentication.isAuthenticated();
    }
  
  ~~~
    
   Security는 인증에 대한 요청/결과 값을 `Authentication` 객체에 담아 처리한다. 하지만 인증에 대한 결과정보는 최종적으로 `SecurityContextHolder` 객체로 감싸져 제공된다.
   
   `SecurityContextHodler`는 내부의 `SecurityContext(Authentication)`객체를 Thread-local로 제공, 같은 쓰레드 에서는 `SecurityContextHodler`를 통해 어디서든 인증정보에 접근이 가능하다.
   
   
   ## Spring Security Structure
   
   우선은 소셜 로그인이 아닌 기본적인 Form Login 의 구조를 살펴보자
   
   <p align = center>
   <img src = https://media.vlpt.us/images/tmdgh0221/post/943d8964-04c2-4310-8b32-caaf0aac8b65/security-aritchtecture.PNG>
   </p>
   
   1.사용자가 로그인 정보와 함께 인증 요청(HttpRequest)
   
   2.`AuthenticationFilter` 가 요청을 가로챔, 이때 가로챈 정보를 통해 `UsernamePasswordAuthenticationToken`객체(현재 미검증 Authentication) 생성
   
   3.`ProviderManager` 구현체인 `AuthenticationManager`에게 `UsernamePasswordAuthenticationToken`객체를 전달
   
   4.`AuthenticationProvider`에 `UsernamePasswordAuthenticationToken` 객체를 전달
   
   5.실제 DB로부터 사용자 인증 정보를 가져오는 `UserDetailsService`에 사용자 정보를 넘겨줌
   
   6.넘겨받은 정보를 통해 DB에서 찾은 사용자 정보인 `UserDetails`객체를 생성
   
   7.`AuthenticationProvider`는 `UserDetials`를 넘겨 받고 사용자 정보를 비교
   
   8.인증이 완료되면, 사용자 정보를 담은 Authentication 객체를 반환
   
   9.최초의 `AuthenticationFitler`에 `Authentication`객체가 반환됨
   
   10.`Authentication` 객체를 `SecurityContext`에 저장
   
   > 여기서 주의깊게 봐야할 부분은 `UserDetailsService`와 `UserDetails`이다. 실질적인 인증 과정은 사용자가 입력한 데이터(ID,PW등) 와 `UserDetailsService`의 `loadUSerByUsername()` 메소드가 반환하는 `UserDetails`객체를 비교함으로써 동작한다. 따라서 `UserDetailsService`와 `UserDetails` 구현을 어떻게 하느냐에 따라 인증의 세부 과정이 달라진다.
   
   만약 OAuth 2.0 로그인을 사용한다면, `UsernamePasswordAuthenticationFilter` 대신 `OAuth2LoginAuthenticationFilter` 가 호출된다. 
   
   두 필터의 상위 클래스는 `AbstractAuthenticationProcessingFilter`이다. 사실 스프링 시큐리티는 `AbstractAuthenticationProcessingFilter`를 호출하고, 로그인 방식에 따라 구현체인 `UsernamePasswordAuthenticationFilter` 와 `OAuth2LoginAuthenticationFilter` 가 동작하는 방식이다.
