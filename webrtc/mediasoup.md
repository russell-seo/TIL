# Mediasoup 미디어 서버(WebRTC)

필자는 회사에서 게임에 들어가는 음성채팅, 화상채팅을 구현해야 했고 원래는 익숙한 Java 기반의 Kurento 미디어 서버를 사용할려고 하였으나

Process가 일정한 수준이 넘어가면 프로세스가 내려가는 현상이 생겨서 `janus`와 `mediasoup`를 스트레스 테스트를 한 결과 mediasoup 의 CPU 사용량이 훨 낮고 성능상 좋았기에 `mediasoup`을 사용하기로 하였다.



# What is Mediasoup

mediasoup and its client side libraries are designed to accomplish with the following goals:

- Be a SFU (Selective Forwarding Unit).
- Support both WebRTC and plain RTP input and output.
- Be a Node.js module in server side.
- Be a tiny JavaScript and C++ libraries in client side.
- Be minimalist: just handle the media layer.
- Be signaling agnostic: do not mandate any signaling protocol.
- Be super low level API.
- Support all existing WebRTC endpoints.
- Enable integration with well known multimedia libraries/tools.


공식 홈페이지에서는 위와 같이 설명하고 있다.

아래는 위의 번역이 아니라 기본적인 Mediasoup에 대한 설명이다.

- `mediasoup`은 기본적으로 소비자(Consumer)와 생산자(Producer)에 기반한다
- 생산자는 `mediasoup` 라우터를 통해 미디어를 보내고 소비자도 역시 `mediasoup` 라우터를 통해 미디어를 받는다.
- 생산자와 소비자를 생성하기 위해 `Transport`를 생성하고 이는 라우터에서 생성된다.
- 라우터는 Room 을 나타내는데, 방은 여러 전송을 보낼 수 있으며 각 방에는 `ProducerTransport`와 `ConsumerTransport`만 가질 수 있다.
- 라우터는 Worker로 부터 생성되며 많은 라우터를 가질 수 있다.
  - Worker는 단일 CPU 코어 내에서 실행되므로 CPU 하나당 하나의 Worker만 가질 수 있다.
 
## 아키텍처

<img width="996" alt="스크린샷 2023-09-13 오후 11 37 57" src="https://github.com/russell-seo/TIL/assets/79154652/3e65c1fa-873c-40f4-9a47-1dbc8d349c69">




필자는 이 `mediasoup`의 흐름이 처음에 너무 이해하기 어려웠고 특히나 기존에 알던 WebRTC 와 미디어 교환 및 연결하는 과정이 다르기 때문에 더 어려움을 겪었다.

일단 먼저 기본적으로 사용되는 용어에 대해 알아보자

- `RtpCapabilities` : 미디어 수신 관련 정보
- `Transport` : Peer Endpoint 와 Mediasoup Router를 연결
- `Produce` : (Instructs the transport to send an audio or video track to the mediasoup router) 공식 문서상에는 Transport 가 mediasoup router 에 오디오와 비디오 데이터를 전송하는 것을 의미한다.
- `Consume` : (Instructs the transport to receive an audio or video track from the mediasoup router) Transport 가 mediasoup router로 부터 오디오와 비디오 데이터를 받는 것
- `Connect` : Server Side Transport와 연결을 위한 정보 교환을 수행




## Process flow


1. Client -> 서버의 Mediasoup Router 의 RtpCapabilites 를 요청한다.
2. Client -> Local의 Device 를 생성하고 서버에서 받은 Router 의 RtpCapabilites 를 통해 `load()`한다.
3. Client -> 서버에 WebRtcTransport 를 생성하는 요청을 한다. 서버에서는 `router.createWebrtcTransport()`메소드를 통해서 Transport 를 생성한다.
4. Server -> Transport 의 id, iceParameters, iceCandidates, dtlsParameters 를 Client 에 전달한다.
5. Client -> 서버에서 받은 Transport 의 데이터를 가지고 Client 측의 SendTransport를 생성한다.
6. Client -> Client 사이드의 SendTransport 가 생성되면 SendTransport 의 `produce()` 를 호출한다.

          -> SendTransport의 produce()는 connect()이벤트와 produce()이벤트를 발생시킨다.
7. Client -> `Connect()` 를 호출하고 Server 측으로 dtlsParameters를 보낸다.
8. Server -> dtlsParameters 를 받아 해당 Transport의 `connect()` 를 호출한다.

>> Producer Client <-> Server 연결 완료
  
10. Client -> `produce()`를 호출하고 서버측으로 Parameter 및 Callback 메소드를 보낸다.
11. Server -> Transport의 `produce()`를 호출해서 Producer를 만든다.
12. Server -> producer Id를 Client 측에 전달한다.

>> Producer is now Sending media to the Server


13. Client -> Client 에서 미디어 수신을 받을 `Recieve Transport`를 생성한다. Server 에서 이와 connect 할 `Transport`를 생성한다.
14. Client -> Client 에서 Device의 RtpCapabilities 와 Producer의 Id를 보내서 어떤 말하는 Producer의 수신을 받을지 서버에 전달한다.
15. Server -> 서버는 rtpCapabilities 와 Producer id 로 해당 라우터의 `canConsume()` 메소드를 호출해서 미디어 수신이 가능한지 확인하고 Consumer를 생성한다.
16. Server -> Client 측으로 Consumer와 return 받은 데이터를 전달한다.
17. Client -> Client의 Recieve Transport의 `Consume()`을 호출한다. 서버측으로 dtlsParameters를 전달
18. Server -> dtlsParameters 를 받아 `Transport.connect()`를 호출한다.
>> Consumer Client -> Server 연결



## Reference

아래 공식 문서를 처음부터 따라가다 보니 쉽게 구현할 수 있었다.

[Mediasoup Documentation](https://mediasoup.org/documentation/v3/communication-between-client-and-server/)
