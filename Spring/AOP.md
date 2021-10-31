## AOP란?

- Aspect Oriented Programming의 약자로 관점지향 프로그래밍 이라고 함
- 트랜잭션, 로깅, 인증과 관련된 공통화 할 수 있는 부분을 따로 배서 관리하는 것

### AOP 용어

- JoinPoint : 클라이언트가 호출하는 모든 비지니스 메소드, 조인포인트 중에서 포인트컷되기 때문에 포인트컷의 후보로 생각
- PointCut : 특정 조건에 의해 필터링 된 조인포인트, 수많은 조인포인트중 특정 메소드에서만 횡단 공톤기능을 수행시키기 위해 사용
- Advice : 횡단 관심에 해당하는 공통 기능의 코드, 독립된 클래스의 메소드로 작성한다.
    - 동작시점
        - Before : 메소드 실행 전에 동작
        - After : 메소드 실행 후에 동작
        - AfterReturning : 메소드가 정상적으로 실행된 후에 동작
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

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ee6b6b51-1dfc-41c5-9964-6c570f3d1d13/Untitled.png)

Advice가 적용되는 순서는 다음과 같다.

- @Before
- @Around(메소드 수행 전)
- 대상 메소드
- @Around(메소드 수행 후)
- @After
- @AfterReturning
