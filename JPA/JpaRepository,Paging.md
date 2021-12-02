

## JpaRepository

  - 필자는 개인 프로젝트를 진행하면서 Spring Data JPA를 사용하여 프로젝트를 진행하고 있으며 JPA 에서 제공하는 기본적인 CRUD가 가능하도록 JpaRepository 인터페이스를 사용 및 자세히 알아보기 위해 이 글을 작성한다.
  
  - Spring Data JPA에서 제공하는 JpaRepository를 상속하기만 하면 따로 @Repository 어노테이션을 추가하지 않아도 JpaRepository 에서 자동으로 Bean으로 주입할 수 있다.
  
    
## CRUD 메소드

  아래 그림은 JpaRepository 에서 제공하는 기본 CRUD 메소드 이다.

  ![image](https://user-images.githubusercontent.com/79154652/144344118-90ad48b7-aebc-495f-9e4f-7b4bda8ab1df.png)

  - save(S) : 새로운 Entity는 저장하고 이미 있는 Entity는 merge 한다. -> Entity에 식별자 값이 없으면 em.persist() 호출, 식별자 값이 있으면 이미 있는 Entity라 판단하여 em.merge() 호출
  - delete(T) : Entity 하나를 삭제한다. 내부에서 EntityManager.remove() 호출
  - findByid(ID) : Entity 하나를 조회한다. 내부에서 EntityManager.find() 호출
  - getOne(ID) : Entity를 프록시로 조회한다. 내부에서 EntityManager.getReference() 호출
  - findAll(---) : 모든 Entity를 조회한다. Sort 나 Pageable 조건을 파라미터로 제공할 수 있다.
  
 

  추가로 아래 코드처럼 findByUsername(String username)처럼 관계에 맞게 메소드명 을 지어주면 자동으로 Spring Data Jpa가 메소드 이름을 분석하여 적절한 JPQL을 실행한다.

  ![image](https://user-images.githubusercontent.com/79154652/144343055-a4983a88-8fa7-4e96-94b8-a4be5e7df22e.png)
      
       실행되는 JPQL : `select m from Member m where username = :username`
  
  
  `Query 메소드에 포함할 수 있는 키워드는 아래 표와 같다.`
  
  ![image](https://user-images.githubusercontent.com/79154652/144347055-0c94dcd1-3c77-4a35-be1b-8e53feb84c88.png)





## JPA 설정
  
  
   ![image](https://user-images.githubusercontent.com/79154652/144344548-a08e290d-338e-4ec2-8c90-d0345caaccea.png)
   
   - Spring Boot를 이용하면 위와 같이 의존성을 추가하면 버전을 포함한 기본적인 설정을 모두 자동으로 해준다.
   
   `자동 설정에 의해 @SpringBootApplication 어노테이션이 붙은 클래스를 기점으로 모든 해당 패키지와 하위 패키지를 모두 스캔하여 Spring Data Jpa 관련 로직을 모두 빈으로 등록한다`
   
   - 따라서 Repository 인터페이스를 구현하면 해당 인터페이스를 구현한 클래스를 동적으로 생성한 다음 스프링 빈으로 등록한다.

    
    



   ![image](https://user-images.githubusercontent.com/79154652/144345258-b09da0a7-3b49-4e0e-a496-5b88430ecbe5.png)

    
   - org.springframework.data.repository.Repository 를 구현한 클래스는 스캔대상
   
      - MemberRespository 인터페이스가 동작한 이유
      
      - memeberRespository.getClass() class com.sun.proxy.$ProxyXXX
          -> Spring Data Jpa가 인터페이스 구현체를 프록시로 구성하여 빈 등록함
 
 
## JpaRepository Interface 계층 구조
  
  ![image](https://user-images.githubusercontent.com/79154652/144345499-288bca10-fde4-43c6-bdfe-1fba0b9f430f.png)
  
  - 제네릭 타입
  
      - T : Entity
      - ID : Entity 식별자 타입
      - S : Entity와 그 자식 타입

    

  
