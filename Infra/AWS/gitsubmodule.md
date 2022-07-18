# Git SubModule vs Subtree

## Git SubModule

- Git 저장소에 다른 Git 저장소를 추가하는 방법이다.
- SubModule로 추가된 저장소(Repository)는 상위 저장소에서 1개의 폴더로 취급되며 일반적인 파일처럼 접근해서 사용 가능하다.
- 그러나 상위 저장소는 서브모듈의 정보 중에서 체크아웃한 커밋의 SHA 값만 기록한다. 즉 만약 서브모듈의 원격에 새로운 내용이 푸쉬 되어서 서브모듈을 업데이트 해야 하는 상황이 생기면 어떻게 될까?
    1. 서브모듈 B를 추가한 저장소 A에서 서브모듈을 직접 수정한 후 커밋을 했다.
    2. 서브모듈 B의 원격에 새로운 내용이 푸시되었다.
        - 저장소 A에 있는 서브모듈의 내용과 원격에 있는 서브모듈의 내용은 서로 다르다. 충돌이 발생할 수 있다.
    3. 서브모듈의 업데이트를 해야 한다. git submodule update 명령어로 사용한다.
- 위와 같은 시나리오에서 서브모듈의 병합은 제대로 이루어 지지 않는다. 상위 저장소에서 서브모듈을 SHA값, 하나의 바이너리처럼 취급하기 때문이다. SHA 값이 다르므로 그냥 최신 커밋의 내용으로 교체해버린다.

## Git SubTree

- Git Subtree는 SHA값만 저장하는 서브모듈과 달리 상위 저장소에 파일을 직접 추가하고 트래킹 한다. 자연히 서브트리의 파일 및 변경사항도 상위 저장소에 기록된다.
- 그리고 서브트리의 원격에 있는 소스와 서브트리를 추가한 저장소의 소스가 서로 달라도 “subtree merge” 기능을 사용해서 양쪽의 변경사항을 모두 반영할 수 있다.
- **이처럼 상위 저장소에서 서브트리를 직접 수정하고 원격에 푸시할 수 있다는 것이 서브모듈과 큰 차이점이다**.
- 예를 들어 서브트리의 컴포넌트를 상위 저장소의 프로젝트에서 사용할때 버그를 발견 했다면 즉시 수정해서 커밋한 후 버그를 해결한 상태로 서브트리의 원격에 반영할 수 있다.

![image](https://user-images.githubusercontent.com/79154652/179464239-efbf53ec-d466-4799-9208-36b090fecc5e.png)
![image](https://user-images.githubusercontent.com/79154652/179464342-3ddd3d57-0610-435d-ad1c-866c8ae70c52.png)


### Git SubModule 이란 ?

→ 만약 두 개의 프로젝트에서 공통으로 필요한 모듈이 있다면 이러한 공통모듈을 하나의 저장소로 관리 한다면? 

→ 이러한 공통모듈을 원하는 프로젝트의 원하는 이름(디렉토리)으로 존재할 수 있다.

→ 서브모듈을 적용하면 하나의 디렉토리가 해당 서브모듈의 Repository로 변경된다.

### SubModule 이 포함된 슈퍼 프로젝트 Clone

→

```bash
git clone (복제할 슈퍼프로젝트 url)
-> 슈퍼프로젝트를 clone 했을 시 포함된 submodule 코드는 가져오지 않는다.

git submodule init

git submodule update --remote(원격 저장소에 있는 submodule 커밋기반으로 코드를 가져옴)

```

아래 그림과 같이 실제로 Git SubModule을 적용해본 사례 입니다.

- 먼저 Git Repository를 두 개 생성합니다.

![image](https://user-images.githubusercontent.com/79154652/179464492-8770899a-bd0f-40ea-884d-9b1e992fba3d.png)

- 그리고 각 Repository 에서 간단한 작업 후 해당 코드를 Remote 저장소에 Push 했다고 가정하고 진행합니다.
- 아래와 같이 submodule 해당 프로젝트로 이동합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a3dd7251-12ab-40f8-a5af-c5b2205f8d4a/Untitled.png)

```bash
git submodule add (서브모듈로 지정할 Repo 주소) 
https://github.com/russell-seo/submodule2.git

(만약 해당 프로젝트 디렉토리의 이름을 정하고 싶으면)
-> 
git submodule add (서브모듈로 지정할 Repo 주소) (디렉토리이름) 
```

- 서브모듈로 사용하고 싶은 Repo 의 주소를 복사하여 위의 명령어를 실행하면
    
    아래와 같이 해당 프로젝트에 submodule2 라는 모듈이 복사된다.
    

![image](https://user-images.githubusercontent.com/79154652/179464559-acbef12d-b913-452d-ab5e-aeaed53d065b.png)
- 현재는 submodule 저장소 로컬에만 해당 submodule2 라는 모듈이 추가된 것이고
    
    remote 저장소에 반영해야 git 에서 다른 개발자들이 볼 수 있다.
    

```bash
//submodule Repo 저장소에서 커밋 후 푸쉬

git commit -am "기록용 메시지"

git push
```

- 정상적으로 push 가 성공했다면 아래와 같이 submodule2 @ d3d2408 이라는 디렉토리가 보이게 된다.

![image](https://user-images.githubusercontent.com/79154652/179464609-74b2b275-0e29-4e3a-aafb-53b35defbbdd.png)
### 서브 모듈 코드 수정 후 변경된 코드를 반영하는 방법

1. 먼저 서브 모듈 Repository 에 변경된 코드를 commit, push 한다.
2. 부모 프로젝트인 submodule 저장소로 터미널에서 해당 submodule2 디렉토리로 접근한다.(주의 할 점, 부모 프로젝트에서 해당 서브모듈 디렉토리로 접근해야 한다)
3. 서브모듈 디렉토리에 접근하여 git pull 명령어를 실행한다.(아래 변경된 코드를 업데이트 하였다)

![image](https://user-images.githubusercontent.com/79154652/179464683-40c3137a-68a8-40e8-b88d-f5d15f23757f.png)
1. 다시 부모 프로젝트로 돌아가서 git commit -am “메시지" 명령어를 수행한다.

![image](https://user-images.githubusercontent.com/79154652/179464717-9eb30de0-eeec-4ef7-b520-1c84ce961b27.png)
1. 마지막으로 git push 를 하면 원격 저장소에 변경된 코드가 반영된다.

![image](https://user-images.githubusercontent.com/79154652/179464757-97ee9d2b-5f1d-4a10-814d-507d240fe786.png)
```bash
git submodule update
-> 만약 submodule 이 1개 이상일 경우 submodule의 

git submoudle update --remote
-> 원격 저장소에 커밋된 submodule 버전으로 update 한다.
```

### 반드시 명심해야 할 점

→ **서브모듈을 포함하고 있는 메인프로젝트는 서브모듈 그 자체를 갖고 있는 것이 아니라 서브모듈 원격 Repo 의 commit 내역을 참조 하고 있는 것 이므로, 서브모듈에 수정사항이 생겼다면 꼭 서브모듈 먼저 PUSH 후 메인프로젝트를 PUSH 해야 한다.**

## Git Cherry-Pick

→

동일한 프로젝트에 있는 다른 브랜치에 대한 Commit 이력은 Git Cherry-pick 기능으로 커밋포인트를 적용 할 수 있다.

하지만 다른 프로젝트의 커밋포인트를 땡겨와서 적용하고 싶은 경우 아래와 같이 진행하면된다.
