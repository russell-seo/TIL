# Spring Security

Spring Security의 동작에 대해 이해할려고 많은 문서들을 뒤져봤지만 쉽게 이해할 수 없어 직접 코드를 쓰면서 이해해 보기로 했다. 필자는 Spring Security 를 적용하여 간단한 로그인 인증관련 테스트를 진행할 예정이다.

개발 환경

  - IntelliJ IDEA
  - Spring Boot 2.6.3
  - Java 8
  - Gradle
  - H2

__Gradle 의존성 추가__

~~~java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}

~~~


__의존성을 추가 했으므로 본격적으로 SecurityConfig 를 작성해보자__

~~~java
  
@Configuration
@EnableWebSecurity  // 해당 어노테이션을 붙인 필터를 스프링 필터 체인에 등록
public class SercurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/").permitAll() // root 권한에 대한 요청은 허용한다.
                .anyRequest().authenticated() // 그 외의 요청에 대해서는 인증이 필요하다.
                  .and()
                .formLogin()
                  .loginPage("/login") //로그인 페이지로 이동
                .permitAll()
                 .and()
                  .logout()
                  .logoutSuccessUrl("/")
                .permitAll();
    }


    @Bean
    @Override
    public UserDetailsService userDetailsServiceBean() throws Exception {
        UserDetails build =
                User.withDefaultPasswordEncoder()
                .username("user")
                .password("password")
                .roles("USER")
                .build();

        return new InMemoryUserDetailsManager(build);
    }
}

~~~

Spring Security 의존성을 추가하면, 스프링 부트는 그 스프링 시큐리티 자동설정을 적용해줘서, 모든 요청에 대해서 인증을 필요로 하게 된다.
(일부 url을 허용하려면, WebSecurityConfigurerAdapter를 구현하여 설정하면 된다.)

  ## Architecture
  
  용어정리
  
  스프링 시큐리티에서는 `인증` 과 `권한`을 분리하여 체크할 수 잇도록 구조를 만들었다.
    
   - Authentication(인증) : A 라고 주장하는 주체(user, subject, principal) 가 A가 맞는지 확인 하는 것.
      - 코드에서 `Authenitication` : 인증 과정에서 사용되는 핵심 객체
         - ID/PASSWORD, JWT, OAuth 등 여러 방식으로 인증에 필요한 값이 전달되는데 이것을 하나의 인터페이스로 받아 수행하도록 추상화 하는 역할의 인터페이스 이다.
   - Authorization(권한) : 특정 자원에 대한 권한이 있는지 확인하는 것
      - `인증`(Authentication) 을 거치고 인증이 되었으면 `권한`(Authorization) 이 있는지 확인 후, 서버 자원에 대해서 접근 할 수 있게 되는 순서
   
   - Credential(증명서) : 인증 과정 중, 주체가 본인을 인증하기 위해 서버에 제공하는 것(ID/PASSWORD)
