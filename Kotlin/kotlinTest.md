

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
  
  결론 먼저 말씀 드리면, 해당 에러를 뱉는 이유는 `AuthService`를 빈 주입을 받지 못하는 에러였고 `Autowired constructor`를 추가하니 에러가 해결 되었다.
  
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
  
  - 해당이유에 대해 설명하자면 
  
  
