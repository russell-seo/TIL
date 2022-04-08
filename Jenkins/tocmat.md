

# Jenkins Tomcat에 War 파일 배포

  필자는 회사 프로젝트에서 Tomcat 에 war 파일을 배포하는 식으로 진행하였다.
  
  
  ## CI & CD
  
  먼저 CI(Github Repo에서 clone 하는 기록은 TIL에 이미 기록해 놓았다) 는 해당 TIL을 참고하면 된다.
  [여기](https://github.com/russell-seo/TIL/blob/main/Jenkins/CI%26CD.md)
  
  
  이제부터 Git에서 프로젝트를 받아와 Build 까지 성공했다고 하고 원격서버에 War파일 배포를 진행해볼려고 한다.
  
  필자는 원격서버에 Docker 로 Jenkins 설치하였고, 아래와 같은 순서로 CI&CD 가 진행된다.
  
  __Github push -> (Docker) Jenkins Build -> Docker 원격서버로 배포__
  
  - 먼저 DashBoard -> Jenkins 관리 -> PlugIn -> Deploy to container 를 설치한다.
  
  > Dashboard -> Project -> 빌드 후 조치
  
  
  
  
  ![image](https://user-images.githubusercontent.com/79154652/161716131-f0eedfe6-9809-4df1-ab58-89714889ddda.png)


  - `WAR/EAR files` : **/*.war 로 설정해 준다.
  - `Context path` : 서버에 올라갈 Path를 입력한다. 예로 들면 http://주소:8080/test/~~ 로 API를 호출하고 싶으면 `/test` 라고 입력해 준다.
  - `Credentials` : 먼저 원격 서버에 올라가 있는 Tomcat의 버전을 선택한다.
      - Add를 클릭하고 가장 중요한 원격서버의 ../tomcat9/conf/tomcat-users.xml 에 있는 username 과 password를 입력해주고 이를 Credentials로 사용한다.
      
      ![image](https://user-images.githubusercontent.com/79154652/161718496-23988a35-f0c1-4ac9-8f21-b5d0c4c738a8.png)
      
      - 해당 xml 파일에서 username="설정" password="설정" 해주고 저장 하고 나오면 된다.
  - `TomcatURL` : 원격서버의 IP 주소를 입력해 준다. 
  
  __여기 까지 하면 원격 서버에 war 파일 배포 준비가 끝이다__
  
  이제 Github 에서 push 후 배포 되는 것을 보면 된다.
  
  하지만 필자는 계속 아래와 같은 Error가 난다.
  
  ![image](https://user-images.githubusercontent.com/79154652/161720059-9dca2e56-8186-472c-bfb5-106c3828de83.png)


  ## 해결법
  
  > 원격서버의 ../tomcat9/webapps/manager/META-INF 경로의 `context.xml` 에서 수정해주어야 한다.
    
    
        <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
      <Valve className="org.apache.catalina.valves.RemoteAddrValve"
             allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1"  />
      <Manager sessionAttributeValueClassNameFilter="java\.lang\.           (?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>


  위와 같은 코드를 주석으로 처리해 주어야한다.
  
  ![image](https://user-images.githubusercontent.com/79154652/161720551-2bf4fba9-e8a7-4873-9844-7b2bd5b7af4e.png)

  - 로컬에서만 허용한다는 코드를 모두 주석 처리해 주면 정상적으로 빌드 및 배포되는 것을 볼 수 있다.
  - 혹은 아래와 같이 특정 IP만 접속하게 끔 IP를 등록해준다.

        <Context antiResourceLocking="false" privileged="true" >
        <Valve className="org.apache.catalina.valves.RemoteAddrValve"
                 allow="특정ip|127.0.0.1"/>
        </Context>


  ![image](https://user-images.githubusercontent.com/79154652/161720949-5cf34088-0518-49a3-98ed-98fc5e56f973.png)
  
  
  
  마치며
  ---
  
  일단 테스트용으로 Spring 프로젝트의 db.properties 도 함께 git에 올려서 Build를 진행했지만.
  
  DB 정보들은 추가로 따로 빼내어서 진행해 볼려고 한다. 

  
