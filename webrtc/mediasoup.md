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
- `Transport` : Endpoint 와 Mediasoup Router를 연결
- `Produce` : (Instructs the transport to send an audio or video track to the mediasoup router) 공식 문서상에는 Transport 가 mediasoup router 에 오디오와 비디오 데이터를 전송하는 것을 의미한다.
- `Consume` : (Instructs the transport to receive an audio or video track from the mediasoup router) Transport 가 mediasoup router로 부터 오디오와 비디오 데이터를 받는 것
- `Connect` : Server Side Transport와 연결을 위한 정보 교환을 수행




## Process flow



