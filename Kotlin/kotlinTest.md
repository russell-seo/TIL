

# JUnit 5 + Kotlin 테스트 클래스에서 생성자 주입 이슈

  ## 문제 상황
  
 ~~~kotlin  

@SpringBootTest
@ActiveProfiles("dev")
class AuthServiceTest(private val authService: AuthService) {


    @Test
    fun `패스워드_유효성_테스트`() {
        //given
        val isNotLengthPassword = "fM1@"
        val isNotSpecialCharacterPassword = "fM12ekol"
        val isNotUpperCasePassword = "fm3@kpowqs"
        val isNotNumberPassword = "fm@Mkpe#erkgj"
        val isSatisPassword = "qwedfM1!@"
        //when

        val isNotLength = authService.isValidPassword(isNotLengthPassword)
        val isNotSpecial = authService.isValidPassword(isNotSpecialCharacterPassword)
        val isNotUpperCase = authService.isValidPassword(isNotUpperCasePassword)
        val isNotNumber = authService.isValidPassword(isNotNumberPassword)
        val canPass = authService.isValidPassword(isSatisPassword)

        //then
        Assertions.assertThat(isNotLength).isEqualTo(false)
        Assertions.assertThat(isNotSpecial).isEqualTo(false)
        Assertions.assertThat(isNotUpperCase).isEqualTo(false)
        Assertions.assertThat(isNotNumber).isEqualTo(false)
        Assertions.assertThat(canPass).isEqualTo(true)
    }
  
  
  ~~~
  
  - Kotlin으로 테스트 코드를 작성하고 테스트를 돌리는 중에 `org.junit.jupiter.api.extension.ParameterResolutionException: No ParameterResolver registered for parameter ~~`
  에러 코드를 만나게 된다.
  
 > 결론 먼저 말씀 드리면, 해당 에러를 뱉는 이유는 `AuthService`를 빈 주입을 받지 못하는 에러였고 `Autowired constructor`를 추가하니 에러가 해결 되었다.
  
  ~~~kotlin
@SpringBootTest
@ActiveProfiles("dev")
class AuthServiceTest @Autowired constructor(private val authService: AuthService) {


    @Test
    fun `패스워드_유효성_테스트`() {
        //given
        val isNotLengthPassword = "fM1@"
        val isNotSpecialCharacterPassword = "fM12ekol"
        val isNotUpperCasePassword = "fm3@kpowqs"
        val isNotNumberPassword = "fm@Mkpe#erkgj"
        val isSatisPassword = "qwedfM1!@"
        //when

        val isNotLength = authService.isValidPassword(isNotLengthPassword)
        val isNotSpecial = authService.isValidPassword(isNotSpecialCharacterPassword)
        val isNotUpperCase = authService.isValidPassword(isNotUpperCasePassword)
        val isNotNumber = authService.isValidPassword(isNotNumberPassword)
        val canPass = authService.isValidPassword(isSatisPassword)

        //then
        Assertions.assertThat(isNotLength).isEqualTo(false)
        Assertions.assertThat(isNotSpecial).isEqualTo(false)
        Assertions.assertThat(isNotUpperCase).isEqualTo(false)
        Assertions.assertThat(isNotNumber).isEqualTo(false)
        Assertions.assertThat(canPass).isEqualTo(true)
    }
  
  
  ~~~
  
 ### Juptier가 ParameterResolver를 몾찾는 이유 

- `테스트가 아닌 메인`에서는 생성자의 매개변수를 Spring IOC 컨테이너 에서 관리합니다. 스프링 프레임워크의 Application Context는 등록할 빈들을 찾아서 저장하고 관리하다가 빈을 런타임 시점에 주입해주는 방식이다.
  
  그렇기 때문에 Kotlin의 생성자는 `@Autowired`없이도 스프링 컨테이너가 잘 주입해주었던 것이다.
  
  하지만 테스트 코드의 경우는 조금 다르다.
  
  테스트의 경우에는 생성자 매개변수를 Spring IOC 컨테이너가 관리하는 것이 아니라 `Jupiter`에서 관리하기 때문에 `Jupiter`가 Spring IOC 컨테이너에게 요청하는 것이기 때문이다.
  
 > `@Autowired`로 명시적으로 선언을 해주어야 `ParameterResolver`에서 해당 어노테이션이 붙은 Bean을 찾아서 주입해주는 것이다.


 
  
  
