# CPU도 당신처럼 예측하고 미리 움직인다.

CPU : 연산장치 -> 속도가 중요하다

연산은 Core가 한다. 연산할 데이터들은 RAM에 존재한다.

CPU와 RAM 사이에 완충 역할은 하는 것이 `Cache` 이다.

- Cache 3개 종류가 있다.

L1, L2 Cache 는 CPU와 붙어있다.(제조사 마다 다름)

L3는 코어가 같이 쓴다.

소위 예측은 Cache 에서 일어난다. CPU가 미리 예측해서 RAM에서 `Cache` 메모리로 미리 가져온다.


CPU -> 연산
RAM -> 데이터(자료를 다루어줌)


GPU -> 다수의 코어, 용도가 다르다, AI 연산에 자주 사용됨
PIM -> Process In Memory 메모리에서 연산을 한다.

# CPU가 예측해서 발생한 심각한 문제(보안사고)

가상화된 VM에서 CPU연산은 진짜 CPU에서 처리한다. 즉 2개의 가상머신중 하나가 읽어온 Cache 데이터를 다른 VM에서도 읽을 수 있다.

[참조](https://parksb.github.io/article/31.html)

# 프로세스와 스레드

컴퓨터에서 자원은 크게 `CPU`, `RAM` 나뉜다.

`RAM`, `HDD`를 합쳐서 Virtual Memory 라고 한다. 프로세스 단위로 `Virtual Memory` 가 주어진다. 

기본적으로 자원을 Process 에게 준다. 윈도우 같은 경우에는 스레드 기준으로 CPU를 준다.

OS 입장에서 프로세스를 관리하기 위해 필요한 장치를 `PCB`가 있다. 쓰레드는 `TCB`

- CPU는 프로세스를 줄을 세워서 분할해서 사용한다.(시분할 사용)

- OS가 프로세스를 관리할때 Queue가 쓰인다. 몇천개의 프로세스를 큐에 넣고 첫번째 부터 디스패치 해서 연산. 코어 개수만큼씩 꺼낸다.

# 프로세스 상태(휴식, 보류)와 문맥 교환

![image](https://user-images.githubusercontent.com/79154652/233117316-19767dd3-c541-4587-ab9a-b2fb8e7faa44.png)


- Sleep, Suspend 두 상태가 존재한다.

- Suspend 는 외부 요인(OS, 다른 프로세스)
  - 의도되지 않은 것이다.
 
- Sleep은 자발적이다.

- Ready-Queue 에서 어떤 스케줄링 되고 있는 프로세스가 Sleep or Suspend를 하면 대기열에서 이탈한다.
- Sleep(10ms)를 하면 10ms + & 만큼 쉬는데 이 & 가 대기열에서 이탈했다가 다시 대기열로 재진입한다. 그래서 대기열 앞에있는 프로세스가 다 처리되어야 다시 진입한 프로세스가 연산한다. 이것이 & 이다.
- Suspend 도 Sleep과 동일하다.

![스크린샷 2023-04-20 오전 12 10 43](https://user-images.githubusercontent.com/79154652/233120092-4fdee5ed-71ba-470d-a938-ed771126002e.png)

- CPU는 연산하면서 상태가 계속 변화 한다. 이러한 상태 데이터는 모두 Register에 저장되어 있다.

# Process의 생성과 복사

- Process 에 가상메모리가 할당된다. 가상메모리는 독립적인 공간이며 Process에 속한 Threads들이 접근 할 수 있다.
- 새로운 프로세스가 생성될 때 가상메모리 공간이 있어야 한다.


