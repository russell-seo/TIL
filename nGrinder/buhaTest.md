
# nGrinder API 서버 부하 테스트 


  회사 프로젝트 막바지에 우리 서버가 얼마만큼의 부하를 견딜 수 있는지 부하테스트를 진행하는 것이 중요하다고 했다. 그래서 필자는 네이버 오픈소스인 nGrinder를 통해 부하테스트를 진행
  하려고 한다.
  
  
  ## nGrinder 란?
  
   nGrinder 는 네이버에서 성능 측정 목적으로 개발된 오픈 소스 프로젝트 이다.
   
   nGrinder 는 서버에 대한 부하를 테스트하는 것이므로 서버의 성능 측정이라고 할 수 있다. 성능 측정이란 실제 서비스에 투입되기 전, 실제와 같은 환경을 만들어 놓고 서버가 사용자를 얼만큼 수용할 수 있는지를 실험할 때 사용한다.
   
   - nGrinder 구성요소
      - Controller
          - 웹 기반의 GUI 시스템
          - 유저 관리 - 다른 컴퓨터를 유저로 관리
          - 에이전트 관리
          - 부하테스트 실시 & 모니터링
          - 시나리오 작성


   - Agent
      - 부하를 발생시키는 대상이다.
      - Controller의 지휘를 받는다.
      - 복수의 머신에 설치해서 Controller의 신호에 따라서 일시에 부하를 발생시킨다.
    
  
  ## nGrinder 설치
  
  nGrinder 는 기본적으로 jdk 가 설치되어있어야 동작한다.
  
  필자의 로컬에는 이미 jdk가 설치되어 있어 이 부분은 생략하고 넘어간다.
  
  - nGrinder Controller 설치
      - [다운로드 경로](https://github.com/naver/ngrinder/releases)
      - 위의 경로에서 Controller를 설치해준다.(다운로드 OS 가 윈도우가 아니라면 `wget https://github.com/naver/ngrinder/releases/download/ngrinder-3.5.5-20210430/ngrinder-controller-3.5.5.war`)
      - 설치가 완료되면 `java -XX:MaxPermSize=200m -jar ngrinder-controller-3.5.5-p1.war -p 8009` 해당 명령어로 실행한다.
      - 설정한 포트로 nGrinder 접속(초기 ID/PW : admin) 

      ![image](https://user-images.githubusercontent.com/79154652/163957909-8a5ae756-74f3-4e12-ba78-5746ae68305c.png)

  
  - nGrinder Agent 설치
      
      ![image](https://user-images.githubusercontent.com/79154652/163958172-5ca318d2-c3a9-4c22-87c8-7bfb8dae7aff.png)
      
      - 위 이미지에서 보는바와 같이 Download Agent를 찾아서 다운로드 한다.
      - 해당 폴더로 들어가서 agent.conf 파일을 생성한다.
      ~~~
      common.start_mode=agent 
      agent.controller_host= Controller IP주소 
      agent.controller_port=16001 
      agent.subregion= 
      agent.owner=
      ~~~
      
      - 설정이 완료되었으면 ./run_agent.sh 혹은 run_agent.bat으로 실행한다.
      
      ![image](https://user-images.githubusercontent.com/79154652/163958766-4678ff42-def8-4daf-9b37-d47e9084c45a.png)
      
      - Agent 가 잘 실행되었다면 위와 같이 Max : agent 갯수 가 나타나게 된다.
