# WebRTC

  Web Real-Time Communication 의 약자로 웹/앱에서 별다른 소프트웨어 없이 카메라, 마이크등을 사용하여 실시간 커뮤니케이션을 제공하는 기술이다.

  - 화상통화, 화상 공유 등을 구현할 수 있는 오픈소스
  - 비디오, 음성 및 일반 데이터가 P2P방식으로 피어간의 전송 지원
  - WebRTC는 웹에서 사용할 수 있는 유일한 P2P 기술이며, 각각의 기기가 서버 도움 없이 연결되기 위해 이 연결을 도와주는 `시그널링 서버`가 필요하고 P2P 연결이 불가능한 상왕을 대비해 `TURN/STUN 서버`가 필요하다.
  - UPD 기반의 스트리밍 기술이다. Latency가 가장 짧은 것 중 하나이다.

  P2P 방식을 사용하기 위해서는 각 피어간의 IP를 연결하는 방식을 차용하기 때문에 방화벽이 존재하거나 라우터를 사용하는 NAT 환경에서는 연결이 불가하다. 따라서 각 피어간의 연결을 위해 사설 IP를 공인 IP 로 바꿔주는 STUN 서버 또는 TURN 서버를 이용해야 한다.

  ![image](https://github.com/russell-seo/TIL/assets/79154652/a7da75a8-17f7-47f6-baec-20bcb246362e)


## ICE(Interactive Connectivity Establishment)

ICE 는 두 단말이 서로 통신할 수 있는 최적의 경로를 찾을 수 있도록 도와주는 프레임워크 이다.

- Relayed Address : TURN 서버가 패킷 릴레이를 위해 할당하는 주소
- Server Reflexive Address : NAT 가 매핑한 클라이언트의 공인망(Public IP, Port)
- Local Address : 클라이언트의 사설주소(Private IP, Port)

따라서 STUN 서버는 Server Reflxive Address 만을 응답하지만, TURN 서버는 Relayed Address와 Server Reflexive Address를 모두 응답한다.

ICE Candidate 라는 개념이 추가로 존재한다. 이것은 `IP와 포트의 조합으로 표시된 주소이며 이제 이 확보된 것을 통해서 연결을 해야 한다.`

3가지 연결을 지원한다.

- Direct Connection : Host 간의 직접적인 미디어 송수신
- Server Reflexive Connection : Server Reflexive Candidate를 이용한 미디어 송수신
- TURN Relay Connection : Relay Candidate 를 이용한 미디어 송수신

이렇게 확보된 3개의 주소들의 우선순위를 정하여 `SDP`내에 포함시켜 전송한다. Connection을 체크한 후 Connection 이 완료되면 RTP 및 RTCP 패킷을 전송하여 통화가 가능하게 된다.

> RTP란 실시간 전송 프로토콜(Real-time Transport Protocol, RTP)은 IP 네트워크 상에서 오디오와 비디오를 전달하기 위한 통신 프로토콜 이다.

![image](https://github.com/russell-seo/TIL/assets/79154652/177d1a77-c50f-4034-89be-ea4bd2b1d80d)


`왜 ICE를 사용해야 하는가?`
모든 단말은 각자의 환경이 다양하기 떄문에 P2P로 단순하게 연결되지 않는다. 방화벽이 존재하는 환경에서는 방화벽을 통과해야 하고 단말의 공인 IP가 없다면 유일한 주소값을 할당 해야하고 라우터가 Peer 간의 직접 연결을 허용하지 않을때는 데이터를 릴레이 해야 한다.

ICE 프로세스를 사용하면 NAT 가 통신을 위해 필요한 모든 포트를 열어두고 두 엔드 포인트 모두 다 연결할 수 있는 IP주소 포트에 대한 완전한 정보를 갖게 된다.

결국 요청하는 클라이언트와 미디어 서버 사이의 연결을 통해 미디어를 주고 받을 수 있다. ICE는 혼자 동작하지 않으며 STUN과 TURN서버를 사용해야 한다.

## STUN

STUN(Session Traversal Uilities for NAT) 은 NAT 환경에서는 사설 IP를 별도로 가지고 있기 때문에 Peer to Peer(P2P) 통신이 불가능하다. 따라서 클라이언트는 자신의 Public IP를 확인하기 위해 STUN 서버로 요청을 보내고 서버로 부터 자신의 Public IP를 받는다. 자신이 받은 Public IP를 이용하여 시그널링을 할 때 받은 정보를 이용해서 연결을 한다.

![image](https://github.com/russell-seo/TIL/assets/79154652/92e3ff5e-0a27-4b97-b3b6-dd4b415b4868)


## TURN

STUN 의 확장으로 NAT 환경에서 릴레이하여 통신하게 된다.

NAT 보안 정책이 너무 엄격하거나 NAT 순회를 하기 위해 필요한 NAT 바인딩을 성공적으로 생성할 수 없는 경우 TURN을 사용한다.

TURN 서버는 인터넷망에 위치하고 각 Peer들이 사설망 안에서 통신한다. 각 Peer 들이 직접통신 하는 것이 아닌 Relay(전달) 역할을 하는 TURN 서버를 사용하여 경유한다.

TURN은 이러한 릴레이로 부터 IP 주소와 포트를 클라이언트가 취득할 수 있는 릴레이 주소를 할당한다.

1. 클라이언트는 자신의 Private IP가 포함된 TURN 메시지를 TURN 서버로 보낸다.
2. TURN 서버는 메시지에 포함된 Network Layer IP 주소와 Transport Layer 의 UDP 포트 넘버와의 차이를 확인하고 클라이언트의 Public IP로 응답하게 된다.
3. NAT는 NAT 매핑테이블에 기록되어 있는 정보에 따라서 내부 네트워크에 있는 클라이언트의 Private IP로 메시지를 전송한다.

하지만 TURN 서버는 클라이언트와의 연결을 거의 항상 제공하지만 STUN에 비해 리소스 낭비가 심하다. 그렇기 때문에 ICE Candidate 과정에서 Local IP로 연결할 수 있는지 공인 IP로 연결할 수 있는지를 알아낸 후 사용해야 한다.

![image](https://github.com/russell-seo/TIL/assets/79154652/150f1d0f-84a5-4aa0-ab77-49e258dc79f9)


## SDP(Session Description Protocol)

SDP란 Session Description Protocol 의 약자로 연결하고자 하는 Peer 서로간의 미디어와 네트워크에 관한 정보를 이해하기 위해 사용된다.

### Offer SDP

- 먼저 연결하고자 하는 Peer 가 만든 SDP를 말한다.


### Answer SDP

- 응답하는 Peer 가 만든 SDP를 말한다.

SDP의 간단한 예를 보면서 간략하게 알아보자

~~~
v=0 
o=- 6137031273746274589 2 IN IP4 127.0.0.1 
s=- 
t=0 0 
a=group:BUNDLE audio video data 
m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 126
...
~~~

`v=0` -> 프로토콜 버전

`o=- 6137031273746274589 2 IN IP4 127.0.0.1` -> 유저이름 (생략됨 -로 표기됨), sessionId, sessionVersion, network type, address type, unicast address 

`s=-` -> 세션이름 (여기선 생략)

`t=0 0` -> 세션이 활성화 됐을 시간 (좌측: start time / 우측: end time 세션만료없이 영구적)

`a=group:BUNDLE audio video data` -> 미디어 라인들 간 관계 (오디오, 비디오, 데이터채널 사용)

`m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 126` -> 미디어타입, 포트번호, 프로토콜, 코덱 프로파일 번호 (peer간 협상과정에 번호. 코덱이 상호 간 가능한지 확인하고 실패시 순서대로 적용)

번들 그룹핑은 SDP 내에 있는 미디어 라인들간에 관계를 형성 한다. 예를 들어 audio video 라고 기술되어 있다면, 이는 datachannel없이 audio와 video 에 관한 라인들만 있음을 의미한다. 즉 audio, video만 사용됨을 의미한다.


### SDP Offer <-> Answer SDP 흐름

![image](https://github.com/russell-seo/TIL/assets/79154652/adca7c09-37c3-4c97-b237-f3e9c2a12138)

- Device A
  - getUserMedia()로 미디어 캡처
  - RTCPeerConnection 생성, RTCPeerConnection.addTrack() 호출
  - RTCPeerConnection.createOffer() 로 Offer 생성
  - RTCPeerConnection.setLocalDescription() 으로 Offer를 LocalDescription으로 설정
  - SetLocalDescription() 호출 후 , Ice Candidate를 STUN 서버에 요청
  - 시그널링 서버를 이용해 Device B에 Offer 전송
 
- Device B
  - Device A의 Offer를 수신
  - RTCPeerConnection.setRemoteDescription 호출 해서 remoteDesciption 기록
  - 호출 종료 시 , Local Media 캡처, 각 mediaTrack을 RTCPeerConnection.addTrack()을 통해 Peer Connection 연결
  - RTCPeerConnection.createAnswer() 로 answer 생성
  - RTCPeerConnection.setLocalDescription 으로 생성된 Answer에 LocalDescription 설정ㅈ
  - 시그널링 서버를 사용해 Device A에 Answer 전달
 
- Device A
  - Answer를 받고
  - RTCPeerConnection.setRemoteDescription 으로 Answer를 RemoteDescription 으로 설정
  - 이제 Device A 와 B는 서로의 구성을 모두 알 수 있다.



## 마무리

WebRTC를  사용하기 위한 기본적인 개념에 대해서 알아보았다. 이제는 WebRTC Kurento 프레임워크에 대해서 알아 볼려고 한다.
