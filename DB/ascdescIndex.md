# Mysql Ascending Index vs Descending Index

먼저 Asc Index와 Desc Index에 대해서 찾아보게 된 이유가 있다.

필자는 현재 게임 채팅서버 라이브 운영중에 Scouter로 패킷들에 대해 모니터링하고 있는데 패킷중에서 간간히 `200ms ~ 300ms` 정도 수행되는 특정한 쿼리가 있어 사실 개선이 필요한 정도의 SLOW 쿼리는 아니었지만 실행계획을 찾아보니 `Backward Index Scan` 이 보여서 이에 대해 정리해볼려고 한다.

![image](https://github.com/russell-seo/TIL/assets/79154652/dad7a0d3-e00a-41e8-8add-2ac4a581f046)

- `Ascending Index`
  - 작은 값의 인덱스 키가 B-Tree 의 왼쪽으로 정렬된 인덱스
- `Descending Index`
  - 큰 값의 인덱스 키가 B-Tree 의 왼쪽으로 정렬된 인덱스

- `Forward Index Scan`
  - 인덱스 키의 크고 작음에 관계없이 인덱스 리프 노드의 왼쪽 페이지 부터 오른쪽으로 스캔 `왼 -> 오`

- `Backward Index Scan`
  - 인덱스 키의 크고 작음에 관계없이 인덱스 리프 노드의 오른쪽 페이지 부터 왼쪽으로 스캔 `오 -> 왼`
