  
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
  
