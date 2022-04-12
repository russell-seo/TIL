
# Jenkins CI/CD, Pipeline 이해하기

  __개요__
  
  - CI/CD 파이프라인 개념 이해
  - Jenkins 기본 개념에 대한 이해
  - Jenkins 토해 기본 배포 파이프라인 구축



  ## CI/CD란 무엇인가?
  
  CI란?
  
  - Continuous Integration을 말한다 -> 무엇을 통합한다.
  - 기본적으로 코드
  - 여러명의 많은 개발자들이 코드 베이스를 계속해서 통합하는 것
  - 여러 개발자들의 코드를 각각 가능한 빠르게 배포하는 것을 의미한다.
  - 코드를 통합한다는 것 이다.


  CD란?
  
  - Continuous Delivery -> 무엇을 배달하다.
  - 내부 사용자(QA, 마케터, 기확자등)든, 사용자들 서비스를 지속적으로 배달한다.
  - 즉 코드 베이스가 항상 배포 가능한 상태를 유지하는 것을 의미한다. -> Continuous Deployment
  - 코드 베이스를 사용자가 사용가능한 환경에 배포하는 자동화하는 것을 말한다.


  > 즉 CI/CD 란 각각의 개발자들이 개발을 하는 개발환경을 사용자가 사용 가능한 서비스로 전달하는 모든 과정을 지속 가능한 형태로 또 가능하다면 자동으로 해서 개발자와 사용자 사이의 격차를 없애는 것이다. 이러한 과정에는 코드를 빌드하고, 테스트 하고 배포하는 활동이 있다.
  
  ![image](https://user-images.githubusercontent.com/79154652/162870192-8bf95d7c-2204-4678-80a6-101ac97dfde0.png)



## Jenkins란 무엇인가?

Jenkins 기본 개념

- Java Runtime Environment에서 동작
- 다양한 플러그인들을 활용해서 각종 자동화 작업을 처리할 수 있다.
- AWS배포, 테스트, 도커 빌드 등 할게 너무 많으니 각각의 컴포넌트들을 하나의 플러그인으로 모듈화를 해놓았다.
- 가장 핵심적인 파이프라인, 시크릿 키 마저도 플러그인으로 동작 시킬 수 있다.
- 즉 일련의 자동화 작업의 순서들의 집합인 Pipeline을 통해 CI/CD 파이프 라인을 구축한다.
- 플러그인들을 잘 조합해서 돌아가게 하는게 Pipeline이라고 할 수 있다.

__Jenkins 대표 PlugIn__

- Credentials PlugIn
    - Jenkins는 그냥 단지 서버이기 때문에 배포에 필요한 각종 리소스에 접근하기 위해서는 여러가지 중요 정보들을 저장하고 있어야 한다.
    - 리소스에는 클라우드 리소스 혹은 베어메탈에 대한 ssh 접근 등을 의미한다.
    - 베어메탈 이란 어떠한 소프트웨어도 담겨 있지 않은 하드웨어를 가르킨다.
    - AWS token, Git access token, secret key, ssh 등의 정보들을 저장할때 사용한다.

- Git Plugin
    - Jenkins 에서 Git 소스코드를 Clone 하여 빌드할 수 있도록 도와줌.

- Pipeline
    - 핵심 기능인 파이프라인도 플러그인이다.

- Docker plugin and Docker Pipeline
     - Docker agent를 사용하고 Jenkins 에서 도커를 사용하기 위한 플러그인 


## Jenkins Pipeline

![image](https://user-images.githubusercontent.com/79154652/162896066-5d477425-7fdc-4e9b-9885-a4cb55e0e771.png)

- Pipeline
    
    - Pipeline 이란 CI/CD 파이프라인을 젠킨스에 구현하기 위한 일련의 플러그인들의 집합이자 구성.
    - 즉 여러 플러그인들을 이 파이프라인에서 용도에 맞게 사용하고 정의 함 으로써 파이프라인을 통한 서비스가 배포된다.
    - Pipeline DSL(Domain Specific Language) 로 작성됨
    - Jenkins 가 동작되기 위해 여러 플러그인들이 파이프라인을 통해 흘러가는 과정이라고 할 수 있다.


- Pipeline 구성요소

    - 두 가지 형태의 Pipeline Syntax 가 존재
        - Declarative
        - Scripted Pipeline
    
    - Section 구성
      - Sections
        - Agent section
        - Post section
        - State section
        - Steps section

- Sections 이해하기
    - Agent section
      - 젠킨스는 많은 일을 해야하기 때문에 혼자하기 버겁다.
      - 여러 Slave node를 두고 일을 시킬 수 있는데, 이처럼 젠킨스가 어떤 일을 하게 할것인지를 지정한다.
      - 젠킨스 노드 관리에서 새로 노드를 띄우거나 혹은 docker이미지를 통해 처리할 수 있다.
      - 쉽게 말하면 젠킨스를 이요하여 시종을 여러명 둘 수 있는데 어떤 시종에게 일을 시킬 것이냐 하는것을 결정하는 것이다.
      - 예를 들어 젠킨스 인스턴스가 서버 2대에 각각 떠 있는 경우, 마스터에서 시킬 것인지 slave에서 시킬 것 인지를 정할 수 있다.
      - 제닌스 노드만 넣을 수 있는 것이 아니라 젠킨스 안에 있는 docker container에 들어가서 일을 시킬 수도 있다.
    
    - Post section
      - 스테이지가 끝난 이후의 결과에 따라 후속 조치를 취할 수 있다.
      - 각각의 단계별로 구별하면 다음과 같다.
      - 성공 시에 성공 이메일, 실패하면 중단 혹은 건너뛰기 등등

    - Stage section
      - 어떤 일들을 처리할 것 인지 일련의 stage를 정의한다.
      - 일종의 카테고리 라고 보면 된다.
    
    - Steps section
      - 한 스테이지 안에서의 단계로 일련의 스텝을 보여줌
      - Steps 내부의 여러가지 스텝들로 구성되며 여러 작업들을 실행 가능
      - 플러그인을 깔면 사용할 수 있는 스텝들이 생겨남
      - 빌드를 할 때 dir 을 옭겨서 빌드를 한다던가, 다른 플러그인을 깔아서 해당 플러그인의 메소드를 활용해서 일을 처리한다
      - 플러그인을 설치하면 쓸 수 있는 Steps들이 많아진다.


마치며
---

Jenkins 에 대한 기본적인 개념과 파이프라인에 대해 이해하기 위해 기록해 놓았다. 이제부터는 실습 및 실무에 적용해보기 위해 기록해 보겠다.

TO BE CONTINUED
