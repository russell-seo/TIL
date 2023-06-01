# Gradle(그레이들)

- `What is Gadle?`
  - JVM 에서 동작하는 스크립트 언어 `groovy`기반의 DSL을 사용한다.   
  - Gradle is an open-source build automation tool flexible enough to build almost any type of software
  - Gradle 은 한마디로 `빌드 자동화 툴` 이다.


- `build.gradle`
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



## Gradle 동작 순서

`Initialization` -> `Configuration` -> `Execution`
위 순으로 Gradle 은 동작한다.

### Initialization

- Initialization step 에는 보통 setting.gradle.kts를 읽어 `프로젝트 이름이 무엇인지. 프로젝트에 어떤 모듈이 있는지 확인한다`.
- 멀티 모듈일 때는 각 모듈 별로 build.gradle.kts 파일이 있는지 확인한다.
~~~
api project(':pms-common')
api project(':pms-domain')
~~~

### Configuration

- Gradle 작업을 하기 위해 필요한 Gradle Task를 순차적으로 수행한다.
  - 보통 라이브러리를 가져오는 작업

![image](https://github.com/russell-seo/TIL/assets/79154652/6ffdfb47-4fa6-4467-9567-213dc3eff02d)

- 우리가 흔히 dependencies를 추가하고 나서 누르는 요놈이 Configuration 단계이다.

### Execution

- Compile 하거나 Test를 하거나 패키징 하는 작업(jar. war. apk)을 Execution 단계에서 한다.
  - Kotlin 이나 Java 파일을 바이트 코드로 변환하는 작업, 변환된 바이트 코드를 사용해 테스트하거나 패키징 하는 작업

~~~
> Task :compileKotlin UP-TO-DATE
> Task :compileJava NO-SOURCE
> Task :processResources NO-SOURCE
> Task :classes UP-TO-DATE
> Task :inspectClassesForKotlinIC UP-TO-DATE
> Task :jar UP-TO-DATE
> Task :startScripts UP-TO-DATE
> Task :distTar UP-TO-DATE
> Task :distZip UP-TO-DATE
> Task :assemble UP-TO-DATE
> Task :compileTestKotlin NO-SOURCE
> Task :compileTestJava NO-SOURCE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test NO-SOURCE
> Task :check UP-TO-DATE
~~~

- Task 가 붙어서 나오는 task들이 Execution 작업이다.


> Gradle Task 기본 블록은 Configuration 단계에서 수행, 추가로 doFirst{}, doList{} 메소드 블록은 Execution 단계에서 실행된다.

~~~
tasks.register("myJob"){
    println("is this first?????")
    
    doFirst{
        println("this is first tasks ")
    }

}


tasks.named("build"){
    dependsOn("myJob")
}
~~~

위처럼 Build 전에 tasks 를 실행하기 위해서 tasks를 작성해 놓았으며, doFirst 를 통해서 해당 Tasks를 먼저 실행할 수 있다.

![image](https://github.com/russell-seo/TIL/assets/79154652/266594cc-5a7e-4a0d-85a7-80b0e37a675e)




- Configuration 단계에서 `is this first?????`가 먼저 출력되고, 다음으로 Execution 단계의 `this is first tasks` 가 출력된다


## Wrapper
- 빌드도구 `Wrapper` 가 제공되기 전에는 각 개발자가 자신의 환경에 빌드 도구를 설치하여 실행환경을 설정하고 관리해야한다.
- 각 개발자마다 빌드도구 버전이 다른 경우 실행되지 않는다.

> 빌드도구를 실행할 수 있는 jar 파일과 이를 실행할 수 있는 스크립트를 함께 등록하여 관리하는 방식. Jar를 포함해야 해서 약간 무거워질 수 있지만 그걸 감수하고 편의를 누린다.

~~~
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar // 그레이들 래퍼 jar
│       └── gradle-wrapper.properties // 그레이들 래퍼 버전 및 실행환경 기록
├── gradlew // Unix 계열에서 실행가능한 스크립트
└── gradlew.bat // 윈도우에서 실행가능한 스크립트
~~~

## Groovy VS Kotlin

- `Groovy Gradle` 과 `Kotlin Gradle`은 `확장자명`으로 구분한다.
  - Groovy -> `build.gradle`, Kotlin -> `build.gradle.kts`
- `Groovy DSL`은 비교적 자유로운 언어 답게 같은 빌드스크립트 지만 작성자에 따라 다른 형태를 취한다.

`build.gradle`
~~~groovy
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter'
  implementation("org.springframework.boot:spring-boot-starter-data-jpa")
  
}
~~~
필요한 라이브러리 의존성을 선언하는 방식인데 큰따옴표, 작은따옴표, 괄호를 이용해서 혼합해서 쓸 수 있다.
즉 하나로 통일되지 않는다.

`build.gradle.kts`
~~~kotlin
dependencies {
implementation("org.springframework.boot:spring-boot-starter-data-jpa")
implementation("org.springframework.boot:spring-boot-starter")
}
~~~
코틀린 DSL은 모든 문자열을 큰따옴표로 작성하도록 한다. 추가로 IDE에서 잘못된 문법 오류코드를 잡아준다.


## 참조
[https://docs.gradle.org/current/userguide/migrating_from_groovy_to_kotlin_dsl.html]()
