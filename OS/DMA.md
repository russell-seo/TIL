# DMA를 알면 고성능 소켓이 보인다.

## DMA(Direct Memory Access)

![스크린샷 2023-04-19 오전 12 10 35](https://user-images.githubusercontent.com/79154652/232821558-f9d2ee5e-9016-4d0b-b90f-e3618164d80e.png)

### DMA를 지원 안할 시

1. 네트워크에 송신할 데이터를 위에 보이는 빨간색 1번 버퍼에 적재한다.
2. File 에다가 Write 한다. Socket 이면 Send -> I/O Buffer에 복사가 된다.
3. 커널 계층으로 내려가면서 Data 가 Segment화 된다.
4. 커널에 존재하는 버퍼에 복사한다.
5. NIC에 보낸다.
6. 네트워크로 나간다.

- 1,2,3 번 모두 RAM 메모리 이다.

### DMA를 지원할 시

1. 프로세스가 메모리를 확보해놓고 recv를 대기한다.
2. 위 그림의 왼쪽에 있는 2,3을 스킵하고 바로 NIC에 들어온 데이터를 Process의 메모리에 적재한다.
