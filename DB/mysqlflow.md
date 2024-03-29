





# Mysql 구조 및 동작 흐름

MySQL 구조 및 동작 흐름을 알기 전에 먼저 DBMS에 대해 이해 해보자

## DBMS

저장소를 분류하는 방법에는 다양한 기준이 있지만 휘발성 기준으로 나눠보자. 여기서 휘발성이란 전원을 껐을 때 담겨있던 정보가 사라지는 것을 의미한다.

휘발성이 있는 저장소의 대표는 RAM이다. 보통 메모리 라고 부른다. 그리고 비휘발성의 저장소중 제일 대표적인 것은 HDD일 것이다.

DMBS = Disk-Oriented. HDD,SSD = Disk


### Random Access vs Sequential Access

Random Access 란 원하는 주소에 바로 접근하는 것을 말한다. 우리가 주소 10에 위치한 20 바이트의 정보를 사용하고 싶다면 Random Access는 바로 주소 10에 접근해서 20바이트를 가지고 오는 것을 말한다.

메모리에선 어떤 주소든 같은 속도로 Random Access를 지원한다.

Sequential Access는 어떤 주소에 접근하는데 있어서 어떤 순서가 존재해서 해당 순서로만 접근 하는 것. 디스크에는 이러한 Sequential Access만 지원한다.

따라서 Random Access 를 하려고 하면 현재 상황에 따라서 어느 주소에 접근하려냐에 따라서 접근 속도가 크게 달라지게 된다. 그리고 이는 시간에 있어서 큰 차이를 가져온다.

DB가 디스크에 파일을 저장할 때는 주소 상에서 연속적으로 저장하는 것이 중요해진다.

ex -> 크기가 1000인 파일이 크기 50씩 20군데로 나뉘어 저장된 것은 Random Access가 20번 발생할 것이고, 연속적으로 한 군데 저장되어 있다면 Random Access가 한번 발생한다고 생각

### Byte-Addressable vs Block/Page-Addressable

메모리에 접근할 때는 어떤 주소에 있는 10 바이트를 가져오라고 명령하면 정확히 그 주소에 가는 것이 가능하다. 이런식으로 바이트 단위로 주소 지정이 가능한 것을 `Byte Addressable`

디스크는 어떤 주소로 가라고 했을 때 그 주소가 있는 블록 혹은 페이지로 가서 해당 주소가 위치한 블록에 접근한다. 이런 것을 `Block/Page Addressable`

### Disk-Oriented DBMS Overview

데이터베이스 파일이 디스크에 존재한다고 해도 쿼리를 처리할려면 결국 메모리 안으로 들고와야 한다.

데이터베이스 파일은 머신이 가지고 있는 물리적 메모리보다 클 수 도 있다. 그래서 DBMS는 디스크와 메모리 사이에서 파일을 어떻게 이동하고 저장할지를 관리한다.

DBMS가 메모리상에서 Buffer pool 이라는 것을 만들고 처리가 필요한 파일 부분을 디스크에서 가지고 와서 메모리에 올리고 또 필요 없어진 부분은 디스크로 보내며 메모리를 관리한다.

파일을 전부 메모리에 올리지 못하기 때문에 DBMS는 파일을 구성할 때 애초에 여러 조각으로 나누어서 저장하는데 그 단위를 보통 `page` 라고 부른다. 

데이터베이스의 특정 영역에 접근하고 싶다고 Buffer pool Manager에게 명령이 들어오면 Buffer pool Manager는 해당 페이지가 현재 자신이 관리 중인 메모리 상에 존재하는지 체크 한다.

만약 있다면 해당 내용을 반환하고, 없다면 페이지 디렉토리라고 페이지를 관리하는 자료구조에 접근해서 해당 페이지의 위치를 찾는다. 이때 페이지 디렉토리 또한 디스크에 존재하므로 만약에 메모리 상에 존재하지 않는다면 이를 먼저 디스크에서 들고온다.

그렇게 요구한 페이지를 찾은 뒤에 디스크에서 메모리 상에 올린다.

파일을 페이지로 구성하고 페이지 단위로 메모리와 디스크 사이에서 파일을 왔다갔다 하는 이유는 메모리가 데이터 베이스 파일에 비해 매우 작기 때문이다. 전체를 메모리에 담을 수 없으니 작은 단위로 필요한 부분을 올려놓고 필요 없어지면 내려놓는다.

![image](https://github.com/russell-seo/TIL/assets/79154652/1266334e-a17c-423e-9469-60215e9ee5b5)

위 그림처럼 Execution Engine 이라는 DBMS 일부가 Buffer Pool Manager에게 2번 페이지를 요구한다. 이때 페이지 디렉토리가 메모리 상에 없다면 이것부터 디스크에서 불러오고 그 뒤에 2번 페이지 디렉토리에서 찾은 뒤에 들고 온다.

그 뒤에 메모리 상의 2번 페이지 주소를 Execution Engine에게 돌려준다.

### Page

파일은 단순히 페이지의 모음이다. OS에서 쓰이는 페이지도 그렇듯이 페이지는 고정길이 이다. 

페이지는 데이터베이스가 가져야할 내용의 어떤 테이블에 대한 메타 데이터, 실제 레코드, 인덱스, 로그 등 어떤 정보도 가질 수 있다. 그렇다고 두 종류의 정보를 한 페이지 안에 담지 않고 하나의 페이지엔 한 종류의 정보만 담는 것이 일반적이다.

DBMS 중 일부는 전체 페이지만 있어도 해당 페이지를 이해할 수 있도록 한다. 이것은 페이지 안에 해당 페이지 내부에 이 페이지에 대한 어떤 내용이 담겨 있고 어떻게 해석하면 되는지에 대한 정보가 있어야 한다는 것이다.

이와 같은 메타데이터가 다른 페이지에 담겨 있다고 할때 그런 페이지가 손상되었다면 다른 페이지를 읽을 수 없기 때문에 아래와 같은 제한을 걸기도 한다.

각 페이지는 유니크한 Id를 가져서 지정할 수 있다.

- DBMS page
  - 고정길이를 가지며 그 길이는 시스템마다 다르게 정의되기도 한다.
  - 페이지 안에 담길 수 있는 정보는 아무거나 다 가능하다. 메타데이터, 레코드, 로그, 인덱스등.
  - 각 페이지는 유니크한 identifier를 가지고 있어서 어떤 페이지를 찾으면 그 페이지가 어떤 것인지 바로 알 수 있도록 한다.
 
### How to store pages

DMBS가 페이지를 디스크에 저장하는 여러방법이 있지만 먼저 heap file organization이라는 방식이다.

heap file 이란 튜플들이 임의의 순서대로 저장되어 있는 페이지들의 unordered collection 이다.

이 Heap file을 저장하기 위해서 Linked list나 page directory를 사용한다. `Linked list`는 굉장히 비효율적인 방식으로 free page와 data page의 Page id 들을 각각 리스트 하나에 저장하고 필요할 때 마다 해당 리스트를 순차검색한다.

`page directory` 란 각 페이지 id와 그 페이지의 주소를 기록해두는 페이지 테이블과 같은 역할을 한다. 접근하고 싶은 페이지 id를 가지고 있으면 그 페이지가 디스크의 어디에 위치했는지 알 수 있다.

그리고 해당 페이지에 공간이 얼마나 남아있는지도 같이 저장해둔다. 이러한 정보들을 담은 page directory 또한 페이지에 담겨서 디스크에 저장될테니 이에 대한 정보도 관리해 주어야 한다.

![image](https://github.com/russell-seo/TIL/assets/79154652/29f072a7-5d0d-40f4-8068-aa9f1132aa4c)


## MySQL

![image](https://github.com/russell-seo/TIL/assets/79154652/af154f4b-5661-4d8f-aeb2-2df3bd5dd823)



Mysql 서버는 크게 Mysql 엔진과 스토리지 엔진으로 나눌 수 있다.

Mysql 엔진의 경우 클라이언트로 부터 접속하고 쿼리 요청을 처리하는 커넥션 핸들러와 SQL 파서, 옵티마이저로 이루어져 있다.

쿼리 관련 처리는 이곳에서 하기 때문에 거의 `두뇌`와 같은 역할을 하면 된다고 생각하자.

스토리지 엔진의 경우 데이터를 `실제 디스크에 저장하거나, 혹은 디스크에 저장된 데이터를 읽어오는 작업을 하며` 하나의 MySQL 서버는 여러 개의 스토리지 엔진을 사용 할 수 있다.

MySQL 엔진과 스토리지 엔진은 서로 `Handler API`라는 것을 사용하여 데이터를 주고 받는다.



## Mysql Thread

![image](https://github.com/russell-seo/TIL/assets/79154652/8da3b87a-9aef-4a2c-9879-43d8d230df05)

Mysql 서버는 기본적으로 스레드 기반으로 동작하며, ForeGround / BackGround 스레드로 나누어져 있다.

### ForeGround Thread

- 최소 MySQL 에 접속된 클라이언트 수 만큼 존재하며, 클라이언트가 요청하는 쿼리문을 수행한다.
- 데이터를 MySQL의 데이터 버퍼나 캐시로 부터 가져오고, 없으면 직접 디스크에 있는 데이터를 가져오거나 인덱스 파일로 부터 파일을 읽어온다.
- 이때 MysISAM 같은 스토리지 엔진은 디스크 쓰기 작업까지 foreGround 가 하지만, InnoDB의 경우 데이터 버퍼 / 캐시 까지만 관리하고 디스크 쓰기는 BackGround 가 처리한다.

- 실행 후 커넥션 종료 시 해당 스레드는 캐시로 돌아가는데, 만약 스레드 캐시에 이미 일정 개수 이상의 대기 중인 스레드가 있으면 스레드 캐시에 넣지 않고 자체 스레드 캐시에 존재하는 스레드 수를 관리하는 역할 도 한다.

- 보통 스레드 수는 thread_cache_size 라는 시스템 변수로 컨트롤이 가능하다.


### BackGround Thread

InnoDB 스토리지 엔진의 경우 백그라운드 스레드가 정말 여러가지 일을 처리한다.

1. 인서트 버퍼(Insert Buffer) 병합
2. 디스크에 로그 기록(Log Thread)
3. 버퍼 풀(Buffer Pool)의 데이터를 디스크에 기록(Write Thread)
4. 데이터를 버퍼로 읽어오기
5. 잠금, 데드락 모니터링

- log / write Thread의 경우 Innodb_write_io_threads, Innodb_read_to_threads 시스템 변수로 스레드 개수를 제어할 수 있다. 쓰기 쓰레드의 경우 2~4개 별도의 스토리지 사용 시 디스크를 최적으로 사용할 만큼 설정하는 것이 좋다.

- 추가로 데이터 쓰기 작업은 지연이 가능하지만 읽기 작업은 지연 처리가 안 되기 때문에 보통 DBMS에서 쓰기 작업은 버퍼링을 통해 일괄 처리하는 기능이 존재한다.
- 덕분에 CUD 쿼리 작성 시 디스크에 반영될 때 까지 완전히 기다릴 필요가 없다. 하지만 MyISAM 에서는 foreGround Thread가 쓰기 작업 까지 하다 보니 쓰기 버퍼링 기능을 사용할 수 없다.


### MySQL Memory Allocation

![image](https://github.com/russell-seo/TIL/assets/79154652/03cdea2f-f73e-4a5e-a1f5-50f56a85f4c2)

MySQL 에서 사용하는 메모리 공간은 크게 `글로벌 메모리 영역` `세션 메모리 영역` 으로 구분된다.

`글로벌 메모리 영역`의 경우 MySQL 서버가 시작되면서 OS로 부터 할당되며, OS마다 조금씩 정책이 다르다. 글로벌 메모리 영역은 단 하나의 메모리 공간만 할당 받으며, 모든 스레드 에게 공유 된다.

`세션 메모리 영역`의 경우 스레드당 하나씩 생성되며, 공유되지 않고 사용된다.
쿼리의 용도 별로 공간이 할당되고, 필요하지 않으면 아예 메모리 공간 자체가 생성되지 않을 수 있기 때문에(정렬, 조인 버퍼) 주의해야 한다.


### MySQL 쿼리 실행 과정

`SQL Parser -> SQL 옵티마이저 -> SQL 실행기 -> 데이터의 읽기/쓰기 작업 -> 디스크`

MySQL 에서는 MySQL 엔진에 의해 SQL 실행기 까지 실행되고, 마지막으로 디스크에 데이터를 읽거나 쓰는 작업은 스토리지 엔진이 처리한다.

어떤식으로 디스크에 데이터를 읽고 쓰는지가 스토리지 엔진별로 갈린다. 이때 데이터를 읽고 쓰는 작업은 대부분 1건의 레코드 단위로 처리된다.

이때 MySQL 엔진은 스토리지 엔진을 조정하기 위해 `Handler` 라는 것을 사용한다.


### MySQL Query Execution

![image](https://github.com/russell-seo/TIL/assets/79154652/7de039a5-ec62-47a9-bc97-aaea3807ccf0)

- `Query Parser`
  - 사용자 요청으로 들어온 `쿼리 문장을 Token 으로 분리하여 트리 형태의 구조로 만들어` 내는 작업을 의미한다.
  - 쿼리 문장의 기본적인 문법 오류는 이 단계에서 발생하며, 이 경우 오류 메시지가 사용자에게 나가게 된다.
 
- `전처리기`
  - 파서를 통해 나온 `트리를 바탕으로 쿼리 문장에 구조적인 문제` 가 있는지 확인한다.
  - 테이블 이름이나 컬럼 이름, 내장 함수 존재여부, 객체의 접근 권한등을 여기에서 확인
 
- `옵티마이저`
  - 쿼리 문장을 `가장 저렴한 비용으로 처리하기 위한 최적화`를 진행한다. 매우 중요


- `실행 엔진`
  - 핸들러에게 요청해서 `받은 결과를 사용자나 또 다른 핸들러의 요청에 대한 입력으로 연결`하는 역할을 수행한다.
  - 핸들러는 실제로 일을 처리하는 역할을 수행하고, 실행 엔진이 옵티마이저와 실행기 사이에서 연결하는 역할을 한다.
    
  > Group By 처리 과정
  > 실행 엔진이 핸들러에게 임시 테이블 생성 요청
  > 실행 엔진은 where 절에 대한 레코드를 읽으라고 핸들러에게 요청
  > 읽어온 테이블을 임시 테이블에게 저장하라고 핸들러에게 요청
  > 생성된 임시테이블에 대해 필요한 방식으로 데이터를 읽어오라고 핸들러에게 요청
  > 최종 결과를 실행 엔진이 사용자나 다른 핸들러에게 넘긴다.
  
- `핸들러(스토리지 엔진)`
  - 실행 엔진의 요청에 따라서 `데이터를 디스크로 저장하거나 디스크로부터 읽어오는 역할`을 진행한다.
  - 어떤 스토리지 엔진을 가진 테이블을 처리하는지에 따라서 InnoDB 엔진을 사용할지, MyISAM 엔진을 사용할지 채택하게 된다.
 
- `쿼리 캐시`
  - 그림에는 없지만, MySQL 쿼리 캐시는 `SQL 실행 결과를 메모리에 캐시하고` 동일한 쿼리가 실행되면 테이블을 읽는 대신에 메모리에 저장된 결과를 바로 반환하는 역할을 한다.
  - 하지만 테이블의 데이터가 변경되면 캐시도 함께 갱신되어야 하기 때문에 변경된 테이블에 관련된 내용은 전부 삭제 했어야 한다. 8.0부터는 쿼리 캐시 기능 자체가 제거됬다.
 

--
https://seastar105.tistory.com/139
--
