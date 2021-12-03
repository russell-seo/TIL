## AOP란?

- Aspect Oriented Programming의 약자로 관점지향 프로그래밍 이라고 함
- 트랜잭션, 로깅, 인증과 관련된 공통화 할 수 있는 부분을 따로 배서 관리하는 것

![이미지](https://t1.daumcdn.net/cfile/tistory/2155894358C6091209)

### AOP 용어

- JoinPoint : 클라이언트가 호출하는 모든 비지니스 메소드, 조인포인트 중에서 포인트컷되기 때문에 포인트컷의 후보로 생각
- PointCut : 특정 조건에 의해 필터링 된 조인포인트, 수많은 조인포인트중 특정 메소드에서만 횡단 공톤기능을 수행시키기 위해 사용
- Advice : 횡단 관심에 해당하는 공통 기능의 코드, 독립된 클래스의 메소드로 작성한다.
    - 동작시점
        - Before : 메소드 실행 전에 동작
        - After : 메소드 실행 후에 동작
        - AfterReturning : 메소드가 정상적으로 실행된 후에 동작
                ![image](https://user-images.githubusercontent.com/79154652/144564850-dbbac712-baef-4f8e-854d-8c5514cf2aed.png)
                 @AfterReturning의 returning = "returnValue" 값으로 Controller에서 넘어오는 response 값을 받을 수 있다.
        - AfterThrowing : 예외가 발생한 후에 동작
        - Around : 메소드 호출 이전, 이후, 예외 발생등 모든 시점에서 동작
- Weaving : 포인트 컷으로 지정한 관심 메소드가 호출될 때, 어드바이스에 해당하는 횡단 관심 메소드가 삽입되는 과정을 의미함. 이를 통해 비지니스 메소드를 수정하지 않고도 횡단 관심에 해당하는 기능을 추가하거나 변경 가능.
- Aspect : 포인트컷과 어드바이스의 결합. 어떤 포인트컷 메소드에 대해 어떤 어드바이스메소드를 실행할지 결정.

### 포인트컷 표현식

```java
@Pointcut("execution(public *.*(..))")
private void test(){}

@Pointcut("within(com.zxc,someapp.trading..*)")
private void test2(){}

@Pointcut("test() && test2()")
private void test3(){}
```

Advice가 적용되는 순서는 다음과 같다.

- @Before
- @Around(메소드 수행 전)
- 대상 메소드
- @Around(메소드 수행 후)
- @After
- @AfterReturning

### 스프링 AOP 특징

- 프록시 패턴 기반의 AOP 구현체, 프록시 객체를 쓰는 이유는 접근 제어 및 부가기능을 추가하기 위해서 임
- 스프링 Bean 에만 AOP 적용 가능.
- 모든 AOP 기능을 제공하는 것이 아닌 스프링 IoC와 연동하여 프로젝트 전역의 중복코드 등을 해결하기 위한 것이 목적

아래 코드와 같이 AOP를 사용하려면 @Aspect, @Component 어노테이션을 붙여 클래스가 Aspect를 나타내는 것을 명시하고 스프링 Bean으로 등록한다.

아래 코드는 Controller에 요청이 들어오면 Token 값을 체크하는 AOP를 구현한 코드이다.

![image](https://user-images.githubusercontent.com/79154652/142518487-4c93132a-8e0c-4215-95a9-ccbf42afbed3.png)

@Around 어노테이션은 타겟 메서드를 감싸서 특정 Advice를 실행한다는 의미이다.

또 특정 @어노테이션을 생성해 Aspect를 실행하는 기능도 제공한다.

![image](https://user-images.githubusercontent.com/79154652/142518815-8ed67f65-62d0-41ac-af18-23eb9bbe3f33.png)


AOP를 구현하면서 대상 객체에 대한 정보 및 파라미터 값을 전달 해야 할 때가 있다. 이때 이러한 정보에 접근할 수 있도록 `ProceedingJoinPoint` 인터페이스를 제공한다.

`ProceedingJoinPoint Method`

- Signature getSignature() : 호출되는 메서드에 대한 정보를 구한다.

- Object getTarget() : 대상 객체를 구한다.

- Object[] getArgs() : 파라미터 목록을 구한다.

