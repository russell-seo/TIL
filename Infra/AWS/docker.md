# <img src="https://t1.daumcdn.net/cfile/tistory/9975EB375B055B7519" width="70" height="40"/> Docker(도커)
  
  
  `컨테이너`,`이미지`
  
  `도커는 개발환경 구축뿐만 아니라 개발 후 운영 환경에 대한 배포나 애플리케이션 플랫폼으로 가능할 수 있다는 점`에서 기존 가상 머신보다 뛰어나다
  
  `도커는 컨테이너 정보를 DokcerFile(이미지) 코드로 관리할 수 있다. 이 코드를 기반으로 복제 및 배포가 이루어지기 때문에 재현성이 높은 것이 특징`

  ## [1] VM vs Docker 
  
  - VM(가상머신)
    - 가상 머신은 하드웨어 스택을 가상화 한다.
    - 전체 OS를 가상화 하는 방식

  - Docker
    - 컨테이너는 이와 달리 `운영체제 수준`에서 가상화를 실시하여 다수의 컨테이너를 OS 커널에서 직접 구동한다.
    - 운영체제 커널을 공유하며 시작이 훨씬 빠르고 운영체제 전체 부팅보다 메모리를 훨씬 적게 차지한다.
  
  
         ![이미지](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fzb2zd%2FbtqFnv1zOjt%2F6ocg67efhRGZA12v9KPHKk%2Fimg.png)
  
  
  
  ## [2] 이미지(Image)
  
   - 컨테이너 실행에 필요한 팡리과 설정값 등을 포함하고 있는 것으로 상태값을 가지지 않고 변하지 않는다.
   - 도커 이미지는 컨테이너 실행하기 위한 모든 정보를 가지고 있기 때문에 보통 용량이 수백메가MB에 이릅니다. 도커는 효율적인 이미지 관리를 위해
     `layer`라는 개념을 사용하고 여러개의 읽기 전용 read only 레이어로 구성되고 파일이 추가 되거나 수정되면 새로운 레이어가 생성된다.
     
     ![이미지](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/image-layer.png)
     
  ## [3] __Docker의 구조__
  
  - 우리는 도커를 사용할 때 docker라는 명령어를 맨 앞에 붙여서 사용한다. 그리고 실제 `docker는 /usr/bin/docker`에 위치하고 있다. 이처럼 명령어 `/usr/bin/docker` 에 위치한 파일을 통해
    사용되고 있다. 하지만 실제 도커 엔진의 프로세스를 확인해보면 `/usr/bin/dockerd` 파일로 실행 된다.
    ~~~Linux
    root        1046  0.0  1.1 1611252 94504 ?       Ssl  Dec14   0:28 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
    ~~~
    
    컨테이너나 이미지를 다루는 명령어는 `/usr/bin/docker`에서 실행되지만 도커 엔진의 프로세스는 `/usr/bin/dockered` 파일로 실행되고 있다. `이는 docker의 명령어가 실제 도커엔진이 아닌
    클라이언트로서의 도구 이기 때문이다.`
    
    
    
    
  **도커의 구조는 크게 두가지로 나누어진다**
  
  1. `클라이언트 도커`
      
      - 도커 데몬은 API 입력을 받아 도커 엔진의 기능을 수행하는데, 이 API를 사용할 수 있도록 CLI(Commnad Line Interface)를 제공하는 것이 `도커 클라이언트`이다.
      
  2. `서버 도커`
  
      - 실제로 컨테이너를 생성하고 실행하며 이미지를 관리하는 주체는 서버도커, 이는 dockered 프로세스로서 동작합니다.
      - 도커 엔진은 외부에서 API 입력을 받아 도커 엔진의 기능을 수행하는데 도커 프로세스가 실행되어 서버로서 입력을 받을 준비가 된 상태를 `도커 데몬` 이라고 한다.


  ## [4] Docker Flow
  
  사용자가 docker로 시작하는 명령어를 입력하면 `도커 클라이언트`를 사용하는 것이며, `도커 클라이언트`는 입력된 명령어를 로컬에 존재하는 __도커 데몬에게 API 로서 전달합니다__
  이때 `도커 클라이언트`는 `/var/run/docker.sock` 에 위치한 `유닉스 소켓을 통해 도커 데몬의 API를 호출합니다.` 도커 클라이언트가 사용하는 유닉스 소켓은 같은 호스트 내에 있는 도커 데몬에게
  명령을 전달할 때 사용 되며, tcp로 원격으로 도커 데몬을 제어하는 방법도 있다.
  
  즉, 터미널이나 Putty 등으로 도커가 설치된 호스트에 접속해 docker 명령어를 입력하면 아래와 같은 과정으로 도커가 제어된다.
  1. 사용자가 docker ps 같은 명령어를 입력
  2. /usr/bin/docker는 /var/run/docker.sock 유닉스 소켓을 사용해 `도커 데몬` 에게 명령어 전달
  3. 도커 데몬은 이 명령어를 파싱하고 명령어에 해당하는 작업을 수행
  4. 수행 결과를 도커 클라이언트에게 반환하고 사용자에게 결과 출력
  
  <p align= "center">
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbP7uqe%2FbtrdMIHeLcy%2F9qoFR5HkiEyQCJumLkMFA1%2Fimg.png" width="500" height="200"/>
  </p>
  
  
  ## [5] 도커 클라이언트
  
   - 도커는 Restful API로 소통하는 Client-Server 체계
   - Client가 요청하는 build, run, pull, push 등의 작업을 받아 HTTP API Request 로 변환하여 Server에 요청


  ## [6] Docker Registry
  
   - 도커 내부에서 도커 이미지들을 보관하는 저장소
   - 클라우드 호스팅 업체에서도 컨테이너 레지스트리


  ## [7] Docker Hub
  
  - 도커에서 운여하는 도커 레지스트리
  - Git처럼 로그인 후 도커 이미지를 업로드(push)/다운로드(pull)하여 사용할 수 있다.
  <p align= "center">
  <img src = "https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FA5muh%2FbtrdMJ0oNE1%2FeuUAF1GCuVswSXWNpm9S7K%2Fimg.png" width="600" height="300"/>
  </p>
  
  ## 참고
  ---
  [https://junstar92.tistory.com/169](https://junstar92.tistory.com/169)
  
  [https://12bme.tistory.com/585](https://12bme.tistory.com/585)
  
