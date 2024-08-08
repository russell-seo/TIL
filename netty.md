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
   
  - Callback
    - 콜백 또한 기존에 우리가 알고 있는 콜백함수와 같다. Netty 가 이벤트를 처리할 때 내부적으로 Callback 을 트리거 하는데, 콜백이 발생하면 내부에서는 ChannelHandler 인터페이스를 구현함으로써 이벤트를 처리할 수 있다.

  - Future
    - Future 는 작업이 완료되면 Application 에 알리는 방법 중 하나다.
      - 이 객체는 비동기 작업의 결과를 담는 역할을 하며, 이를 PlaceHolder 라고 부른다.
    - JDK 자체에서 이와 같은 역할을 하는 java.util.concurrent.Future 인터페이스를 제공하지만, 수동으로 작업 여부를 확인하거나, 완료 전 까지 블로킹을 하는 기능만 있었다.
    - 그래서 Netty 는 자체적으로 CompletableFuture를 사용하여 비동기 작업을 완료되도록 구현하였다.
      - ChannelFuture에는 ChannelFutureListener 인스턴스를 하나 이상 등록할 수 있으며, 작업이 완료되면 이 Listener 들이 호출되며 처리 성공 유무를 확인할 수 있다.
      - 이런 메커니즘으로 인해 수동, 블로킹 하지 않아도 된다.
     
  - `Event`와 `Handler`
    - Netty 는 작업의 상태 변화를 알리기 위해 고유한 이벤트루프를 사용하며, 발생한 이벤트를 기준으로 적절한 동작을 트리거 할 수 있다.

    ![image](https://github.com/user-attachments/assets/e9c4c609-b0ab-4dce-9ebd-4984e3a8b044)

    - 다음과 같은 설계에서 한 Channel 의 입출력이 동일한 스레드에서 처리되기 때문에 동기화가 필요 없다. 비동기 논블로킹 시에는 하나의 이벤트 루프가 여러개의 Channel 과 연결될 수 있다.
