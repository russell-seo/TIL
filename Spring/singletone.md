## 싱글톤 방식의 주의점 - 스프링 빈 클래스는 항상 무상태로 설계 하자 

- 스프링 싱글톤 방식으로 설계시 주의해야할 사항
  - 스프링 빈은 항상 `무상태(Stateless)` 로 설계 하는 것 이다.
  - 스프링 빈으로 등록되는 클래스는 공유가 될 수 있는 전역 변수를 사용하지 말아야 한다.
  - 싱글톤으로 빈 객체를 관리하는 스프링 컨테이너의 특성상 빈에 전역변수를 두게 되는 경우 변수에 들어간 값이 다른 곳에서 `상태를 유지(Stateful)`한 채 사용되는게 문제이다.
  - 서로 다른 요청에 따라 다른 스레드 별로 각자의 스택 메모리 영역을 차지한다 하더라도 스프링 컨테이너가 동작함에 따라 이미 `힙 영역`에 Bean 객체가 로딩되어 공유되기 때문에 Bean 객체를 Stateful 한 상태로 설계 할 경우
    서로 다른 요청임에도 불구하고 값이 꼬여버리는 일이 생길 수 있다. 
  - Bean 객체는 메소드만 공유하는 역할을 해야하며 Bean 객체의 상태값을 유지하게 만드는 필드 값이 없는 Stateless로 설계 해야한다.

>> 추가로 스프링 컨테이너의 경우 컨테이너가 동작되는 시점에 빈으로 설정해둔 객체를 메모리에 Loading 하는 Pre-Loading 방식을 Default 로 취하기 때문에 메모리에서 하나의 Bean을 공유해서 여러 요청을 처리한다.

문제 상황을 코드로 보고 알아보자


### 문제상황

__JoinService__
 ~~~Kotlin
 @Service
class JoinService(
    private val memberJoinPort: MemberJoinPort,
    ) : JoinUseCase {

    private lateinit var name : String // JoinService 싱글톤 객체에서 `name` 이라는 변수가 공유되는 상황

    @Transactional
    override fun joinMember(dto: JoinDto): Member {
        name = dto.name //name 이라는 변수가 공유되는 상황을 만들기 위해 임의로 name 에 할당

        val member =
                Member.create(
                userId = dto.userId,
                name = name,
                password = dto.password,
                phone = dto.phone,
                email = dto.email,
        )
        return memberJoinPort.registerMember(member);
    }
    
    override fun getName(): String {
        return name
    }


}
~~~


- Member 를 저장하는 Service 클래스를 만들고 이를 스프링 빈으로 등록하여 사용한다.
- 위 코드로 테스트 코드를 작성해서 `name`이라는 변수가 공유되는 상황을 직접 확인 해 보자.
- 필자는 스레드 2개를 병렬로 돌리기 위해 Kotlin 의 코루틴을 사용해서 테스트 해 보겠다.

~~~Kotlin
@SpringBootTest
class KotlinPlayGroundApplicationTests {

    @Autowired
    private lateinit var joinService: JoinService


    @Test
    fun `싱글톤 에서 무상태 변수`() = runBlocking{
        val thread1 = async(Dispatchers.Default){
            println("Thread 1: Running")
            val dto = JoinDto(
                userId = "kiaofk1",
                password = "123",
                email = "kiaofk1@naver.com",
                name = "서상원",
                phone = "1"
            )
            joinService.joinMember(dto).id!!
            println("Thread 1: Finish")
        }

        val thread2  = async(Dispatchers.Default){
            println("Thread 2: Running")
            val dto = JoinDto(
                userId = "kiaofk2",
                password = "123",
                email = "kiaofk2@naver.com",
                name = "빅뱅",
                phone = "1"
            )
            joinService.joinMember(dto).id!!
            println("Thread 2: Finished")
        }

        thread1.await()
        thread2.await()
        println("joinService = ${joinService.getName()}")
    }

~~~

- JoinService Bean 객체를 주입받아서 사용한다.(싱글톤 객체로 1개만 생성됨)
- `thread1`과 `thread2`가 병렬적으로 실행된다.

![image](https://github.com/russell-seo/TIL/assets/79154652/afa1220b-e4db-4d80-aa32-00e4b0b34f3a)

- 위 이미지와 같이 DB에 2번의 SQL이 날라간 것을 볼 수 있다.
- 아래 그림을 보면 name -> 서상원, 빅뱅 2개의 데이터가 들어가야 하는 상황
- But 서상원, 서상원으로 SQL이 날라간 것을 볼 수 있다.

![image](https://github.com/russell-seo/TIL/assets/79154652/4fdff4c1-601a-41c7-a3a0-61eeeacc1957)


- 즉 여러요청이 동시에 들어오는 경우 `name`이라는 클래스 변수가 공유되면서 사용자 의도와 다르게 값이 저장되는 것을 볼 수 있다.

## 마무리
- 하나의 객체를 공유하는 싱글톤 패턴의 특성상 상태가 공유되는 변수는 문제가 될 수 있다.
- 싱글톤 패턴을 사용한다면 반드시 무상태(Stateless)로 설계 해야한다.
- 스프링 컨테이너는 싱글톤 패턴을 사용한다. 즉 무상태 설계가 필수이다.
