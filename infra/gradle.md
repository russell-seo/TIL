# Gradle(그레이들)

- What is Gadle?
  - JVM 에서 동작하는 스크립트 언어 `groovy`기반의 DSL을 사용한다.   
  - Gradle is an open-source build automation tool flexible enough to build almost any type of software
  - Gradle 은 한마디로 `빌드 자동화 툴` 이다.


- build.gradle
  - build.gradle 은 파일 자체가 Project 객체로 이는 Project Interface를 구현한 구현체이다. 
  - Project 단위에서 필요한 작업을 수행하기 위해 메소드, 프로퍼티 등을 모아놓은 슈퍼객체이다.

![image](https://github.com/russell-seo/TIL/assets/79154652/9a169ea9-5627-478b-aaff-5ff943868a8d)



- 우리가 build.gradle 에 작성하는 수만은 코드는 모두 Project 오브젝트의 프로퍼티와 메소드가 된다.

- Project 오브젝트는 내부에 많은 메소드 와 프로퍼티를 가지고 있다. 그중 대표적인 것은 Java Application 용 Build.gradle이 가진 plugins, repositories, dependencies, application 메소드이다.
- 우리가 Gradle Task를 이용해 빌드하게 되면 bulid Task 는 이 메소드를 수행시킨다.

![image](https://github.com/russell-seo/TIL/assets/79154652/2f73a215-119f-40cd-869a-1718775067b9)

- `plugins` -> Project Object의 plugins 메소드
- `repositories` -> Project Object의 repositories 메소드
- `dependencies` -> Project Object 의 dependencies 메소드
- `application` -> Project Object 의 application 메소드
