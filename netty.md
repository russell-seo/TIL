# Netty 이해하기

회사에서 Netty 기반의 채팅서버를 운영하고 있는데, 처음 개발한 것이 아니었기에 Netty에 대해 자세하게 알지 못하고 지금까지 사용해 왔는데

이번기회에 정리해볼려고 한다.

Netty 에 대해서 공부하기 전에 NIO 에 대해서 먼저 알고 가는게 좋을 것 같다.

## NIO

NIO 는 Java non-blocking Input/Output의 약자로 이전에 사용하던 BIO 의 한계를 보완하기 위해 나온 기능이다.

BIO는 Blocking Input/Output 이다

- BIO 는 기존 I/O의 가상머신의 한계로 OS의 커널 버퍼를 직접 핸들링 할 수 없었다.
  - 이유는 소켓이나 파일에서 Stream 이 들엉면 커널 버퍼에 데이터를 써야하는데, 당시에는 이를 코드 레벨에서 접근할 수 있는 방법이 없었다.
  - 그 대안으로 BIO의 경우, JVM 이 커널에서 `시스템 콜`을 사용하게 하여 문제를 해결했다. 하지만 이 과정에서 JVM 은 JVM -> 커널 -> 시스템 콜 -> 디스크 컨트롤러 -> DMA가 커널버퍼로 복사 -> JVM 버퍼에 복사 의 긴과정을 거치게 된다.



[https://sightstudio.tistory.com/15](https://sightstudio.tistory.com/15)
