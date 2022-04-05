

# Jenkins Tomcat에 War 파일 배포

  필자는 회사 프로젝트에서 Tomcat 에 war 파일을 배포하는 식으로 진행하였다.
  
  
  ## CI & CD
  
  먼저 CI(Github Repo에서 clone 하는 기록은 TIL에 이미 기록해 놓았다) 는 해당 TIL을 참고하면 된다.
  [여기](https://github.com/russell-seo/TIL/blob/main/Jenkins/CI%26CD.md)
  
  
  이제부터 Git에서 프로젝트를 받아와 Build 까지 성공했다고 하고 원격서버에 War파일 배포를 진행해볼려고 한다.
  
  필자는 원격서버에 Docker 로 Jenkins 설치하였고, 
  
  
  > Dashboard -> Project -> 빌드 후 조치
  
  ![image](https://user-images.githubusercontent.com/79154652/161709746-7c1138d4-33bc-4fed-8978-1f80920805df.png)
  
  
  
