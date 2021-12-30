
# OSIV 와 성능 최적화

  - Open Session in view : 하이버네이트
  - Open EntityManager in View : JPA
    (관례상 OSIV라 한다)
    
    
  ## OSIV ON  
   
   ![image](https://user-images.githubusercontent.com/79154652/147721153-9d78d3f1-89e5-4b18-a745-170e230cfc3b.png)
   
   - `spring.jpa.open-in-view` : true 기본값
    
     이 기본값을 Spring boot는 뿌리면서 애플리케이션 시작 시점에 warn 로그를 남기는 것은 이유가 있다.
     
     ![image](https://user-images.githubusercontent.com/79154652/147721325-7f114e1a-e8b9-4c18-9432-c3d7325bf8de.png)

  위 사진과 같이 warn 로그를 남긴다.
  
  - OSIV 전략은 트랜잭션 시작처럼 최초 데이터베이스 커넥션 시작 시점부터(@Transaction 어노테이션이 붙은 시점) API 응답이 끝날때 까지
    
    영속성 컨텍스트와 DB 커넥션을 유지한다. `즉, Request 시작부터 클라이언트에게 Response 가 나갈 때 까지 영속성 컨텍스트가 유지됨`
    
    그래서 지금까지 View Template 나 API 컨트롤러에서 지연 로딩이 가능했던 것이다.
    
    `지연로딩은 영속성 컨텍스트가 살아있어야 가능하고, 영속성 컨텍스트는 기본적으로 DB 커넥션을 유지한다. 이것 자체가 큰 장점`
    
    
  - 그런데 이 전략은 너무 오랜시간동안 DB 커넥션 리소스를 사용하기 때문, 실시간 트래픽이 중요한 어플리케이션 에서는 `커넥션이 모자랄 수가 있다.`
    이것이 결국 장애로 이어진다.
    
    > 예를 들어 컨트롤러에서 외부 API를 호출하면 외부 API 대기 시간만큼 커넥션 리소스를 반환하지 못하고 유지해야 한다.

  
  ## OSIV OFF
  
  ![image](https://user-images.githubusercontent.com/79154652/147721593-76b34d71-323a-4249-8958-7f583c94866e.png)

  - `spring.jpa.open-in-view : false` OSIV 종료
  
  OSIV를 끄면 트랙잭션을 종료할 때 영속성 컨텍스트를 닫고 DB 커넥션도 반환한다. 따라서 커넥션 리소스를 낭비하지 않는다.
  OSIV를 끄면 `모든 지연로딩을 트랙잭션 안에서(Service, Repository) 처리해야 한다. 그리고 view template에서 지연로딩이 동작하지 않는다.`
  __결론적으로 트랜잭션이 끝나기 전에 지연 로딩을 강제로 호출해 두어야 한다.
  
  ## 해결책
  
  __커맨드와 쿼리 분리__
  
  실무에서 `OSIV를 끈 상태`로 복잡성을 관리하는 좋은 방법이 있다. 바로 Command 와 Query를 분리하는 것이다.
  
  보통 비지니스 로직은 특정 Entity 몇개를 등록하거나 수정하는 것이므로 성능이 크게 문제가 되지 않는다. 그런데 복잡한 화면을 출력하기 위한
  쿼리는 화면에 맞추어 성능을 최적화 하는 것이 중요하다. 하지만 그 복잡성에 비해 핵심 비지니스에 큰 영향을 주는 것은 아니다.
  그래서 크고 복잡한 애플리케이션을 개발한다면, 이 둘의 관심사를 명확하게 분리하는 선택은 `유지보수 관점`에서 충분히 의미있다.
  
  - OrderService
      - OrderService : 핵심 비즈니스 로직
      - OrderQueryService : 화면이나 API에 맞춘 서비스(주로 읽기 전용 트랜잭션 사용)

  보통 서비스 계층에서 트랜잭션을 유지한다. 두 서비스 모두 트랜잭션을 유지하면서 지연로딩을 사용할 수 있다.
  
  > 참고 : 고객 서비스의 실시간 API는 OSIV를 끄고, ADMIN 처럼 커넥션을 많이 사용하지 않는 곳에서는 OSIV를 켠다.
  
