# Flyway(DB Migration)




## What is Flyway?

- 간단히 말해서 DB 마이그레이션 오픈 소스 툴 입니다.
- Database 의 DDL 이력을 쌓아서 관리하는 툴 입니다. 이를 통해 DB 형상관리 및 마이그레이션을 할 수 있다.

- svn 에서는 위험하지만 git에서는 시도 해 볼 만 하다(개인적인 생각)
![image](https://user-images.githubusercontent.com/79154652/209100130-742649e7-55a3-4caa-b485-67697f77b83e.png)




### 장점
- 우리는 현재 dev fqa live 형상 등 모두 DB에 접근하여 DDL을 수행하여야 한다. 즉 실수가 발생할 수 도 있다.
- 소스코드 상에서 관리가 가능하다.
- 도입하기 쉽다.

### 단점
- 롤백이 불가능하다.
- 성공한 sql 파일에 대해서 변경을 권하지않는다(hash 값이 변하여 이후 마이그레이션에 문제가 생길 수 있다)












## Flyway 동작 방식

- 마이그레이션은 버전 숫자 기준으로 순서대로 동작한다.
- flyway 적용은 application이 구동될 때 application 이 반영한다. 
- jpa entity 클래스 와 마이그레이션 파일은 같이 작성해야한다(하나라도 누락되면 어플리케이션이 구동되지 않는다)

![image](https://user-images.githubusercontent.com/79154652/209100599-e5aa736c-eb43-48cf-a6e5-8b00cccc58e1.png)


## Flyway 적용해보기
### flyway 설정

- dependency 추가하기(mysql 사용시)
~~~java
  dependencies {
    implementation("org.flywaydb:flyway-core")
    compileOnly ("org.flywaydb:flyway-mysql")
  }
  //위 처럼 두개를 추가해도 가능하며
~~~

~~~java
  //필자는 Flyway를 코틀린에 적용(멀티모듈에서는 위에 디펜던시가 안먹음) 왜그런지는 모르겠음
  
  plugins {
        id 'org.flywaydb.flyway' version '8.5.13'
  }
  
  dependencies {
      implementation ("org.flywaydb:flyway-core")
  }
~~~

- 두 설정 중 하나를 택하면 된다.


- application.yml 파일에 flyway 설정 추가

![image](https://user-images.githubusercontent.com/79154652/209101314-0df31e1f-f182-4da3-9c7f-95cdb808d453.png)





- enable -> flyway를 사용하겠다는 flag 값(default true)
- baseline-on-migrate -> flyway_history_schema 기준으로 migration 여부
- url -> DB url
- locations -> flyway default location은 resource/db/migration으로 설정되어 있다. 즉 db/migration 아래에 flyway가 정해놓은 규칙으로 sql 파일을 생성하면 버전을 보고 자동으로 db에 DDL을 수행한다.
  
  형상별로 dev/fqa/live 등 폴더를 생성하고 각 yml 파일에 db/migration/dev, /fqa, /live 로 관리를 하면 형상 별로 버전이 꼬일 경우가 없다.



## SQL 파일 작성

- flyway 에서는 version 을 파일명으로 나타내며 flyway에서 정한 규칙대로 작성하여야 한다.(커스텀도 가능하긴 하다)
- 기본 형태는 V{버전 번호}__{name}.sql
- spring.flyway.locations 위치에 V1에 해당하는 .sql 파일이 있어야 한다. 빈 파일이어도 파일자체는 존재해야 flyway_schema_history에 이력이 생기며 이를 baseline으로 하여 이력을 남긴다


- Prefix
	-  V, U ,R 중 선택
	- V(Version) : 버전 마이그레이션
	- U(Undo) : 실행취소
	- R(Repeatable) : 반복 가능한 마이그레이션
	- 버전 상관없이 매번 실행되는 스크립트로 버전 명시가 필요없다.
	- 예시로 테스트를 위해 더미데이터를 넣을 때 사용

		- Separator : 구분자로 _ 가 2개 인것에 주의하자 1개로 작성하면 flyway에서 해당 파일을 스킵해버린다.
		- Description : 실질적 파일명으로 밑줄이나 공백으로 단어를 구분한다.
		- Suffix : 접미사로 보통 .sql을 사용한다


## 아래 해당 Entity 에 email 컬럼을 추가해보자.

1. Entity에 email을 추가해준다.

![image](https://user-images.githubusercontent.com/79154652/209487800-226fbff4-5cfd-45b5-b619-da26859b9a10.png)

	
2. V2__addEmail.sql 파일을 아래와 같이 작성한다.

![image](https://user-images.githubusercontent.com/79154652/209487852-af156a62-de0c-4d59-9292-37fb42e2c837.png)


![image](https://user-images.githubusercontent.com/79154652/209487835-cb734d59-da50-421f-ac4c-56de8fba6983.png)

3. Spring boot 를 구동시킨다.

4. 아래와 같이 로그가 찍히는 걸 확인할 수 있으며 db의 flyway_schema_history 에서 이력을 확인 할 수 있다.

![image](https://user-images.githubusercontent.com/79154652/209487907-d5e61006-c9a4-47a8-ac81-c1942fdbd123.png)









## 기존 .mwb 파일과 다른점

- 필자의 회사에서는 아직 SVN을 다루고 있다... 그래서 Mysql Workbench의 .mwb파일로 DB 테이블 변경을 한다. 하지만 .mwb파일은 이러한 DDL에 대한 이력이 남지않았고 필자는 Flyway 도입을 제안하였고 내부 서비스에 적용하여 테스트 해 보기로 하였다.

- AS IS
~~~
1. 기존 svn 에서는 erd.mwb 파일에 lock을 걸어서 다른 유저가 수정하고 있을 시 수정하지 못하게 한다.
2. 그 이유는 svn은 커밋을 하면 모두 trunk 로 머지되기 때문에 동시성 문제가 생긴다.
~~~
- TO BE
~~~	
1. git에서는 각자 feature/** 브랜치를 따서 로컬에서 작업을 한다. 
2. 즉  서로다른 개발자가 동시에 개발을 해서 develop merge를 하여도 버전과 파일 이름만 동일 하다면 conflict 가 난다. 
3. 만약파일 이름이 달라도 버전이 같다면 같은 버전이 있다고 에러를 뱉어낸다.
4. DDL은 실수 및 위험한 작업이므로 git 에서 merge 할 시 한번 더 확인이 가능하다.
5. 즉 override 되거나 누락되는 일이 없어지게 된다.
~~~

- AS IS
~~~
현재는 로컬에서 dev db를 같이 바라보고 작업을 하기 때문에 동시성 문제를 걱정한다.
~~~
- TO BE
~~~
각자 local 에 docker 로 dev db image 컨테이너를 띄워서 각자 격리된 곳에서 작업하고 local에서 문제 없으면 서로 dev에 머지하는 방향도 괜찮은 것 같다.
~~~
### 참고 문서
[https://flywaydb.org/documentation/getstarted/how](https://flywaydb.org/documentation/getstarted/how)
