# Netty 이해하기

회사에서 Netty 기반의 채팅서버를 운영하고 있는데, 처음부터 개발한 것이 아니었기에 Netty에 대해 자세하게 알지 못하고 지금까지 사용해 왔는데

이번기회에 정리해볼려고 한다.

Netty 에 대해서 공부하기 전에 NIO 에 대해서 먼저 알고 가는게 좋을 것 같다.

## NIO

NIO 는 Java non-blocking Input/Output의 약자로 이전에 사용하던 BIO 의 한계를 보완하기 위해 나온 기능이다.

BIO는 Blocking Input/Output 이다

- BIO 는 기존 I/O의 가상머신의 한계로 OS의 커널 버퍼를 직접 핸들링 할 수 없었다.
  - 이유는 소켓이나 파일에서 Stream 이 들엉면 커널 버퍼에 데이터를 써야하는데, 당시에는 이를 코드 레벨에서 접근할 수 있는 방법이 없었다.
  - 그 대안으로 BIO의 경우, JVM 이 커널에서 `시스템 콜`을 사용하게 하여 문제를 해결했다. 하지만 이 과정에서 JVM 은

    JVM -> 커널 -> 시스템 콜 -> 디스크 컨트롤러 -> DMA가 커널버퍼로 복사 -> JVM 버퍼에 복사 의 긴과정을 거치게 된다.
  - 위 과정에서 발생할 수 있는 문제점
    1. JVM 으로 내부 버퍼 복사 시 CPU 가 관여 -> `CPU 오버헤드`
    2. 복사된 Buffer 는 활용 후 GC의 대상이 됨 -> `Stop-the-World 로 인한 성능 저하`
    3. 복사중인 I/O 요청 쓰레드는 블로킹 상태 -> `처리속도 저하`

Stream 은 단방향으로 읽기 때문에 읽고 쓸때 InputStream, OutputStream 으로 구분하여 사용하였고 읽고 쓰는 작업이 다 끝날때 까지 아무것도 하지못한다.

BIO 의 문제점에 대해서 간략히 위에서 설명하였고 이제 NIO에 대해서 설명해보자

Java 1.3 이후 부터 JVM 에 통일된 인터페이스가 도입되어, 각 OS별 커널 버퍼에 접근할 수 있게 되었다.

I/O와 NIO 모두 Blocking 을 지원하지만 Blocking 을 빠져 나오기 위한 방법에는 차이가 있다. IO는 오직 Stream을 닫는 것 만으로 Stream을 빠져 나올 수 있지만, NIO 는 `Selector`를 통해 해결 할 수 있다.

<img width="819" alt="selector-image" src="https://github.com/user-attachments/assets/1ba1ee07-9e01-4962-af4d-321499efb481">


`Selector`는 Java 상에서 Non Blocking I/O 의 핵심으로 Multiplex IO Select 와 같다. `Selector`는 시스템 이벤트 통지 API를 사용하여 하나의 쓰레드로 동시에 많은 I/O를 처리할 수 있다.

Netty는 리눅스 위에서 동작할 경우 자동적으로 Selector 가 아닌 Epoll 을 사용한다.

`Selector`, `Epoll` 은 모두 시스템콜 이다. 간단히 설명하자면 소켓을 열면 파일 디스크럽트 라는 unsigned int 형식의 소켓 ID를 부여한다.

Selector 는 루프를 돌면서 파일 디스크럽트의 변화를 감지, Epoll 의 경우 콜백 형식으로 관리한다. 즉 Epoll 이 더 빠르다.

## Netty

Netty 는 위와 같은 Low level 의 API 를 직접 사용하면 코드 복잡성이 심화 되기에 Netty 가 이를 인터페이스화 하여 쉽게 구성할 수 있게 탄생하게 된 것 이다.

- Netty 의 주요요소
  - Channel
    - Channel 은 Java NIO 의 Channel 이다. Netty 에서는 데이터를 위한 운송수단으로 활용된다. Netty 가 자동으로 Channel 으로 열고 닫아주기 때문에 개발자가 직접 구현할 필요가 없다.
   
  - 


[https://sightstudio.tistory.com/15](https://sightstudio.tistory.com/15)
