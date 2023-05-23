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

 ~~~Kotlin
 @Service
class JoinService(
    private val memberJoinPort: MemberJoinPort,
    ) : JoinUseCase {

    private lateinit var name : String // JoinService 싱글톤 객체에서 `name` 이라는 변수가 공유되는 상황

    @Transactional
    override fun joinMember(dto: JoinDto): Member {
        name = dto.name

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

    override fun findMember(email: String) : Member? {
        return memberJoinPort.findMember(email)
    }

    override fun getName(): String {
        return name
    }


}
~~~



