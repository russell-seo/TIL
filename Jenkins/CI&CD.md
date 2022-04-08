  
  # Jenkins CI&CD 

   필자는 Github의 Webhook 이벤트를 이용해 Repo 에 Push 이벤트가 발생하면 Git API 를 호출하여 Jenkins 로 해당 Repo의 소스코드를 Clone 및 Build 하는 CI/CD 환경을 적용해 볼 예정이다.
   
   
   
   1. Github 에서 특정 브랜치에 Push 하면 빌드 유발(Webhook)
   2. 젠킨스에서 자동 빌드하고 빌드된 내용을 SSH를 이용해 원격 서버로 전송
   
   
   
   ## Github Webhook, Github Plugin)
   
   __Webhook 이란?__
   
   - 특정 이벤트가 발생하였을 때 서비스나 응용프로그램으로 알림을 보내는 기능
   - 역방향 API
      - 일반적인 API는 클라이언트가 서버를 호출, 반면 Webhook은 Webhook을 호출하는 서버측에 등록하면 서버에서 특정 이벤트가 발생했을 때 클라이언트를 호출함.


  
   ### Github Webhook 추가
   
   > Github Repo -> Settings -> Webhooks -> add Webhook
   
   ![image](https://github.com/binghe819/TIL/blob/master/Infra%26DevOps/CI%3ACD/Jenkins/freestyle/image/webhook_create.png)
   
   
   - Payload URL : 설정한 이벤트가 발생시 내용을 보낼 URL
      - 젠킨스에 보낼 URL 주소를 입력 : http://Jenkins주소/github-webhook/
  
  
  ### Github Integration Plugin  설치
  
  > Jenkins 관리 --> 플러그인 관리 --> 설치가능 --> Github Integration 검색 및 설치
  
  프로젝트에 빌드 유발 관련 설정을 하기 위해 Jenkins 에서 Github Integration을 설치해야 한다.
  
  ![image](https://user-images.githubusercontent.com/79154652/161456765-c6bafc24-4dd9-4f02-a42b-b00afe6a286d.png)

  
  ### Jenkins Project 트리거 설정
  
  Jenkins 에서 프로젝트의 Github Hook 트리거를 설정해주면 된다.
  
  > DashBoard -> 프로젝트 -> 구성 -> 빌드유발 -> Github hook trigger for GITScm polling 클릭
  
  ![image](https://user-images.githubusercontent.com/79154652/161456970-eca03fea-b249-47a7-82d3-8fa4a0b01024.png)

  
  이제 아래 이미지와 같이 내가 push할 브랜치에 소스코드를 push하면 Jenkins에서 이를 받아 자동으로 빌드가 성공된다.
  
  ![image](https://user-images.githubusercontent.com/79154652/161457197-0f902cd4-f212-4965-95eb-a4c4ed04ca10.png)

  
  ![image](https://user-images.githubusercontent.com/79154652/161457317-5d81b1f1-0314-41e7-b690-71e97e42ec52.png)

  빌드 성공 하게 된다.
  
  여기까지다 CI를 적용 시켰다고 할 수 있다.
  
  ## SSH를 이용해 원격 서버에 배포
  
  이제 Github Push를 통해 빌드 유발이 되면, 젠킨스가 빌드한 후 원격 서버에 SSH를 통해 파일을 전송하는 것을 진행해볼려고 한다.
  
  
   ### Publish Over SSH 플러그인 설치 및 설정
   
   > Jenkins 관리 -> Plug in -> Publish Over SSH 설치 -> 프로젝트 구성


   ![image](https://user-images.githubusercontent.com/79154652/162342482-5ce6082f-8a8d-4fdd-a8a2-edb588d1c374.png)
    
   - Key
      - RSA 키가 필요하다
      - AWS EC2 같은 경우는 pem키를 넣으면된다.
      - 일반적으로 서버에서 직접 키를 생성할려면
      - `~/.ssh` 명령어를 통해 
    

  ![image](https://user-images.githubusercontent.com/79154652/162341992-032e714f-13dd-4745-9f9f-7ec5dfa2e261.png)

   
   - SSH Server
      - Name : 시스템 설정 -> SSH Server 등록되어 있는 SSH를 통해 보낼 원격 서버
      - Source files : Jenkins 파일 경로의 workspace `build/libs/*.jar` 빌드 후 해당 경로의 jar 파일을 지칭한다.
      - Remove prefix : 제거할 접두자를 의미, `bulid/libs/` 제거하고 jar 파일만 배포
      - Exec command : 배포 후 실행할 쉘 스크립트 입력
  
  
  
   위 설정들을 모두 마쳤으면 배포 준비가 완료 된 것이다.
   
   이제 Github 해당 Repo에 push를 하면 build 후 배포되는 것을 확인 할 수 있다.
   
   ![image](https://user-images.githubusercontent.com/79154652/162349341-05849313-ce0f-4507-aa9e-8f748d78c3a1.png)
   
   
    
   
   

  
