
#  Redis 운영 이슈와 효율적인 관리


   Redis 운영 이슈는 4가지가 있다.
   
   - 메모리 관리를 잘하자.
   - O(n) 관련 명령어를 주의하자(keys *등) -> Single Thread라 소요 시간이 큰 명령어가 들어오면 Block 된다.
   - Replication
   - 설정
