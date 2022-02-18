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

__추가로 테스트용 웹 MvcConfig 작성__

~~~java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/home").setViewName("home");
        registry.addViewController("/").setViewName("home");
        registry.addViewController("/hello").setViewName("hello");
        registry.addViewController("/login").setViewName("login");
    }
}
~~~


__저장할 User 도메인__

~~~java
@Entity
@Getter
@Setter
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private String password;
    private String ROLE;
    

}
~~~

__테스트용 Controller__
~~~java
@RestController
public class UserController {


    @Autowired
    CustomUserService userService;

    @GetMapping("/create")
    public User create(){
        User user = new User();
        user.setName("russell");
        user.setPassword("password");
        user.setROLE("USER");
        return userService.save(user);
    }
}
~~~
- `/create` 호출 시 DB에 User 저장

__UserDetailService 를 구현한 CustomUserService__

~~~java
@Service
@RequiredArgsConstructor
public class CustomUserService implements UserDetailsService {


    private final UserRepository userRepository;

    private final BCryptPasswordEncoder bCryptPasswordEncoder;


    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByName(username);
        List<GrantedAuthority> authorityList = new ArrayList<>();
        authorityList.add(new SimpleGrantedAuthority("ROLE_USER"));
        System.out.println("user = " + user);
        return new org.springframework.security.core.userdetails.User(user.getName(),user.getPassword(), authorityList); // User에 해당하는 Model에 UserDetails 구현하여 SpringSecurity 가 이해할 수 있는 형태의 User로 만들어 주어야 한다.
    }


    public User save(User user) {
        user.setPassword(bCryptPasswordEncoder.encode(user.getPassword()));
        return userRepository.save(user);
    }
}
~~~

- `UserDetailService` 는 DB로 부터 사용자를 가져와 로그인처리 로직을 구현하는 역할을 한다.
  
  - 필자는 loadUserByUsername()을 구현하고, 당연히 return 값으로 `UserDetails`를 구현한 객체를 return 했으나, SecurityConfig 의 UserDetailService 와 충돌이 있는 듯 하여 계속 인증이 실패하였다.
  - SecurityConfig 의 UserDetailService를 주석처리하니 정상적으로 작동하였다.

__UserRepository__
~~~java
public interface UserRepository extends JpaRepository<User, Long> {

    User findByName(String username);

}
~~~

1. root로 접속한다.

![image](https://user-images.githubusercontent.com/79154652/154649150-cbe4b941-c3d4-46ee-a803-f9ddebeaf55e.png)

2. /hello로 접속을 요청했지만 허가되지 않은 경로 이므로 /login 페이지로 redirect

![image](https://user-images.githubusercontent.com/79154652/154649286-102fd975-f392-4d0f-bbd1-8256f31cab1e.png)

3. /create 로 접속해 미리 Controller에 입력해 놓은 User를 저장한다.
  
  ![image](https://user-images.githubusercontent.com/79154652/154649420-93387882-e426-44be-b594-ca46b4bcab9a.png)

4. 저장한 username 과 password를 입력
  
  ![image](https://user-images.githubusercontent.com/79154652/154649501-738a88a9-3587-49bc-b1fd-05892321e064.png)
  
5. 정상적으로 로그인이 되었다.

![image](https://user-images.githubusercontent.com/79154652/154649783-fd764c41-b347-45ca-bd81-a7ad42aeeb04.png)


__마치며__

간단한 Spring Security 이용한 로그인을 구현하였으며, 다음으로는 프로젝트에 붙일 Oauth2 Security를 공부하고 적용해 볼 예정이다.


참고
--
[https://galid1.tistory.com/576](https://galid1.tistory.com/576)
[백기선 님의 유튜브 영상](https://www.youtube.com/watch?v=fG21HKnYt6g&t=1562s)

  

