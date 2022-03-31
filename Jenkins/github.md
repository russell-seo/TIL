
# Jenkins Github Webhooks 연동


  Jenkins를 사용해서 CI/CD 환경을 구축하기 전에 젠킨스와 Github를 연동하는 것을 먼저 해보고자 한다.
  
  이는 다음 단계에서 Github 에서 Push가 발생하면 이벤트를 캐치해서 CI/CD를 하기 위한 사전 작업 이기도 하다.
  
  Github와 연동해서 Jenkins에서 `build now`를 누르면, 특정 repo 에서 코드를 가져와 빌드하는 것 까지 해보려고 한다.
  
  
  
  
  ## Github 토큰 발급
  
  Github에 로그인 하고 아래와 같이 Github 토큰을 발급 하자.
  
  > Settings -> Developer Settings > Personal access tokens > Generate new token


  ![image](https://user-images.githubusercontent.com/79154652/160955681-80e1d609-075e-4b6a-bf26-eefc19850b20.png)


  
  repo의 권한을 설정해 주자.
  
  
  ![image](https://user-images.githubusercontent.com/79154652/160955905-5d2d67a1-c74d-4f62-8b67-0d44e5cfa3ae.png)
  
  
  발급받은 Token을 복사해 놓아야 한다. 아래에서 진행할 Jenkins에 등록하여야 한다.

  
  
  ## Jenkins Credentials 설정
  
  위에서 발급한 AccessToken을 Jenkins에 등록해준다.
  
  > Jenkins 관리 -> Manage Credentials -> Stores scoped to Jenkins 의 Jenkins 클릭 -> Domain의 Global credentials


  ![image](https://user-images.githubusercontent.com/79154652/160956546-375ddd39-c2e9-4a9c-968b-6237b3444752.png)


  
  > 아래 이미지와 같이 입력해 준다.

  ![image](https://user-images.githubusercontent.com/79154652/160956778-4f20869f-ae5c-49eb-b8a0-f3df600a4f45.png)
  
  
  위와 같이 Secret 에 Github Token을 넣어준다.
  
  - `Kind` : 인증 종류
  - `Secret` : Github 에서 발급 받은 토큰
  - `ID` : Jenkins 인증 정보에 대한 식별 ID
  
  Kind를 `Secret text`로 설정해주어야 한다.
  
  __Username with Password__
  
  ![image](https://github.com/binghe819/TIL/blob/master/Infra%26DevOps/CI%3ACD/Jenkins/freestyle/image/configure_credentials_withuser.png)
  
  Username with Password 형식의 인증정보도 추가해 준다.
  
  아래에서 해당 프로젝트 Repo 설정하는 곳에서 사용된다.
  
  ## Jenkins 에 Github Server 설정
  
  위에서 설정한 인증정보를 바탕으로 Jenkins 설정에 Github Server를 해준다.
  
  > Jenkins 관리 -> 시스템 설정 -> 아래로 내려가면 아래 이미지와 같이 Github Servers 등록하는 란이 있다.

  ![image](https://user-images.githubusercontent.com/79154652/160958192-eba6000a-f9a1-4716-bcca-208f315a7a90.png)

  - Name 을 먼저 입력해 준다.
  - `Credentials`에서 위에서 등록한 Credentials를 선택해 준다.
  - Test connection을 해주고 저장한다.


  ## 신규 프로젝트 생성(Freestyle)
  
  신규 프로젝트(아이템)을 생성해준다.
  
  > 새로운 Item -> FreeStyle project -> name을 입력하고 OK를 눌러 생성한다.
  
  - 이제 Github Project 에 관한 설정을 입력한다.

  ![image](https://user-images.githubusercontent.com/79154652/160958735-9cacc83a-92c5-4bc4-ae3e-2d3e1ed3eba8.png)
  
  - Github 프로젝트의 URL을 입력해준다.


  ![image](https://user-images.githubusercontent.com/79154652/160958907-699e685a-dd9d-4fc6-8541-61d92133ed69.png)
  
  - Repository URL : 해당 프로젝트에서 사용한 Repo 의 URL을 입력
  - Credentials : 위에서 정의한 인증정보 (Username with Password)
  - Branch : 프로젝트에서 사용될 브랜치 정보

  
  
  ## Build now
  
  위의 과정을 모두 수행했다면 Jenkis 프로젝트에서 Build Now를 누른다.
  
  아래와 같이 Console OutPut에 Success라고 출력이 된다.
  
  등록한 Github 브랜치에서 코드를 가져와 빌드가 성공된 것이다.
  
  ![image](https://user-images.githubusercontent.com/79154652/160960499-70cd4744-eae0-48bb-ad77-e715118fc099.png)


  ## 마치며
  
  다음에는 빌드 후 배포하는 내용을 기록할 예정이다.
  
  To be Continue
