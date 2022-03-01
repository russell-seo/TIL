
  # PSA 란?
  
  Spring의 대표적인 핵심가치 3가지로 IoC, AOP, PSA 가 있다. 그 중 이 글은 PSA(Portable Service Abstraction) 에 대해 기록해 보기로 하였다.
  
  
  __PSA(Portable Service Abstraction)__
  
  우리는 Spring의 AOP가 Proxy 패턴을 발전시켜 만들어 졌다. 그리고 FactoryBean을 통해 Proxy가 Bean이 생성될때 자동으로 생성 되는 것 또한 배울 수 있었다.
  
  여기에 우리가 간과하고 있던 사실이 있다. `@Transactional` 어노테이션을 선언하는 것 만으로 별도의 코드 추가 없이 트랜잭션 서비스를 사용할 수 있다는 사실이다.
  그리고 내부적으로 트랜잭션 코드가 추상화되어 숨겨져 있는 것 입니다.
  `이렇게 추상화 계층을 사용하여 어떤 기술을 내부에 숨기고 개발자에게 편의성을 제공해주는 것이 서비스 추상화(Service Abstraction)`입니다.
  
  그리고 아시다시피 DB에 접근하는 방법은 여러가지가 있다. 기본적으로 JDBC를 통해 접근(DatasourceTransactionManger)할 수 있으며 ORM을 이용하고자 하면 JPA(JpaTranscationalManager)를 통해서 접근할 수도 있습니다. 신기하게도 어떠한 경우라도 @Transactional 어노테이션을 이용하면 트랜잭션을 유지하는 기능을 추가할 수 있다.
  이렇게 __하나의 추상화로 여러 서비스를 묶어둔 것을 Spring 에서 Portable Service Abstraction__ 이라고 한다. 또 Service Abstraction으로 제공되는 기술을 다른 기술 스택으로 간편하게 바꿀 수 있는 확장성을 가지고 있는 것이 `PSA(Portable Service Abstraction)` 이다.
  
  > Spring은 Spring Web MVC, Spring Transaction, Spring Cache등의 다양한 PSA를 제공한다.


## PSA의 원리

JAVA로 DB와 통신을 구현하기 위해서는 먼저 DB Conncetion을 맺어야 한다. 그리고 트랜잭션을 시작한다. 쿼리를 실행하고 결과에 따라 Commit, Rollback을 하게 된다. 마지막으로 DB Connection을 종료하며 마무리하게 된다. 이를 JAVA 코드로 나타내면 아래와 같다.

~~~java
public void method_name() throw Exception {
    // 1. DB Connection 생성
    // 2. 트랜잭션(Transaction) 시작
    try {
        // 3. DB 쿼리 실행
        // 4. 트랜잭션 커밋
    } catch(Exception e) {
        // 5. 트랜잭션 롤백
        throw e;
    } finally {
        // 6. DB Connection 종료
    }
}

~~~
  
- 위의 3번 쿼리 실행을 제외하고는 `@Transcational`에서 제어해주는 부분이다. 3번은 우리가 직접구현하는 비니지스 메소드가 될 것이다. 3번을 제외하면 AOP를 통해 구현되어 진다는 사실.

만약 JDBC로 `@Transactional` 이 되어 있다면 아래와 같은 코드가 될 것이다. 이것은 JDBC에 특화되어 있는 코드이다. 이 코드는 JPATransactionManager는 이용할 수 없다. 왜냐하면 JPA는 Connection을 직접 관리하지 않고 EntityManager로 간접으로 관리한다. 어떻게 해야 `@Transactional` 단일 어노테이션으로 JPATranscationManager 도 사용 할 수 있을 까?

~~~java
public void method_name() throw Exception {
    TransactionalSynchronizationManager.initSunchronization();
    Connection c = DataSourceUtils.getConnection(dataSource);
    try {
        // 3. DB 쿼리 실행
        c.commit();
    } catch(Exception e) {
        c.rollback();
        throw e;
    } finally {
        DatasourceUtils.releaseConnection(c, dataSource);
        TransactionSynchronizationManager.unbindResource(dataSource);
        TransactionSynchronizationManager.clearSynchronization();
    }
}
~~~
  
- 바로 추상화에 있다. Spring의 TranscationManager의 관계를 아래사진에서 보여주고 있다. 즉, Spring의 `@Transactional`의 각 TranscationManager를 각각 구현하고 있는 것이 아니라 최상위 `PlatformTransactionManager`를 이용하고 필요한 TransactionManager를 DI로 주입받아 사용하는 사실을 알 수 있다.

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbOLvvy%2FbtqN9MtzKKO%2FmkJukSC9T4xy74ZoOGJmu0%2Fimg.png)


