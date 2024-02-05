# Select for update, Insert DeadLock 이슈

필자는 게임회사에 재직중인데 MMORPG의 채팅서버를 개발하고 있으며 곧 출시를 앞에두고 성능 테스트를 하던중 DeadLock 이 걸리는 기능이 발견되었고 이를 해결하다 보니 Mysql 에서 `select for update` 와 `insert`가 같은 트랜잭션으로 묶여서 실행되면 `데드락`이 종종 발생할 수 있다는 사실을 알고 이를 정리할려고 한다.


## InnoDB

MYSQL 에서 InnoDB 를 사용하면 기본 isolation level 은 `REPEATTABLE READ` 입니다.

InnoDB에서 Isolation Level이 `REPEATTABLE READ` 라면 select, delete, update 쿼리를 실행할때 GAP LOCK을 사용한다.

GAP LOCK과 NEXT-KEY LOCK도 사용한다. `NEXT-KEY-LOCK`은 `supremum` 이라 불리는 임시의 레코드 다음에 있는 GAP 을 잠금하며

NETX-KEY LOCK은 REPEATABLE READ 에서 `Phantom Read` 를 막기 위해 사용한다.

`supremum` 레코드는 인덱스에서 실제 존재하는 값들 보다 더 큰 값을 가진다. 진짜 인덱스 레코드가 아니다. 그래서 NEXT-KEY-LOCK은 가장 큰 인덱스 다음에 오는 값을 잠근다.

> 레코드를 잠글때, 인덱스를 잠근다는 사실을 기억하자. 따라서 테이블의 레코드에 잠금을 획득한다는 것은 인덱스 테이블에 잠금을 획득하는 것이다.

## SELECT FOR UPDATE

MYSQL 에서 SELECT FOR UPDATE 는 하나 또는 특정 범위의 ROW에 대해 여러 세션에서 접근하여 발생할 수 있는 동시성 문제를 해결하기 위해 이용할 수 있다.

기본적으로 `Exclusive Lock(row-level)` X락이 걸리게 된다.

이렇게 해당 ROW에 X락이 걸리기 때문에 다른 트랜잭션에서 해당 ROW에 접근하지 못하고 대기하게 된다. X락은 어떠한 LOCK이랑도 호환되지 않는다.

물론 DB에 데이터가 존재한다면 위와 같이 X락이 걸리면서 Record Lock 이 되는게 맞지만 만약에 DB에 찾고자 하는 Row 가 존재하지 않으면 `NETX-KEY-LOCK` 이 걸리게 된다.\

> GAP LOCK은 레코드 그 자체가 아니라 레코드와 바로 인접한 레코드 사이의 간격을 잠그는 것을 의미한다. GAP LOCK의 역할은 레코드와 레코드 사이의 간격에 새로운 레코드가 생성되는 것을 제어하는 것이다. GAP LOCK은 독자적으로 사용되지 않고 `NEXT-KEY-LOCK`의 일부로 사용된다.

## 내가 만났던 이슈

이슈 내용은 코드상에서 유저를 SELECT FOR UPDATE 로 조회하고 없으면 유저를 생성하는 코드가 하나의 트랜잭션에서 동작하고 있는 상황이다.

여기서 SELECT FOR UPDATE 쿼리를 날릴 때 존재하지 않는 열을 조회하게 되면 원래는 해당 레코드에만 락이 걸려야 하는데 레코드가 존재하지 않아 NEXT-KEY-LOCK 즉 GAP LOCK이 발생하게 된다.

QA 성능 테스트를 하면서 만났던 상황과 동일한 상황을 예시로 들고 데드락을 재현해 보자.

User 라는 아래와 같은 테이블이 존재한다. 

<img width="249" alt="스크린샷 2024-01-31 오전 2 07 19" src="https://github.com/russell-seo/TIL/assets/79154652/0896bdbb-7f2b-436a-9471-cac157beef92">

유저 테이블을 만들고 pk값인 id로 조회하면 데이터의 존재여부에 따라 데드락이 발생할 수 있고 발생하지 않을 수도 있었다.
  - 락을 획득할 때 데이터가 존재했다면 경합이 발생하지 않았다.
  - 반면 데이터가 존재하지 않으면 데드락이 터졌다.

1. 아래와 같이 user 테이블에 데이터가 존재하지 않는다.

<img width="425" alt="스크린샷 2024-02-04 오후 9 36 32" src="https://github.com/russell-seo/TIL/assets/79154652/f229fb81-a4d5-4762-b346-2539ff427432">


~~~mysql

//Transaction 1
begin;
select * from user where uid = 2 for update;
insert into user (name, created_at) values ('서상원', '20240129');


//Transaction 2
begin;
select * from user where uid = 3 for update;
insert into user (name, created_at) VALUES ('서상원2', '20240129');

~~~

2. pk 값으로 아래와 같이 데이터가 없는 테이블에 조회를 하면 데드락이 발생한다.

<img width="812" alt="스크린샷 2024-02-04 오후 9 22 01" src="https://github.com/russell-seo/TIL/assets/79154652/33eae712-09f2-4223-914b-d970d9e5c649">

아래와 같이 데이터가 없는 ROW를 `select for update`로 조회하면 아래와 같은 LOCK 이 잡힌다.

<img width="947" alt="스크린샷 2024-02-04 오후 10 40 59" src="https://github.com/russell-seo/TIL/assets/79154652/868f8df9-730d-4186-b694-4e0ee8d9d3bc">

- 데이터가 없으면 `테이블의 IX락, LOCK_DATA = null`
- `RECORD 단위의 X락, LOCK_DATA = supremum pseudo-record`
  - 데이터가 없을때 발생하는 X락의 LOCK_DATA인 `supremum pseudo-record` 의 경우 마지막 지점을 의미한다. 그래서 `해당 값 기준으로 마지막 PK(INDEX) 까지 GAP-LOCK` 이 걸리게된다

> MYSQL 공식문서에 supremum pseudo-record에 대한 설명을 가져와보았다.
>
> For the last interval, the next-key lock locks the gap above the largest value
in the index and the “supremum” pseudo-record having a value higher than any v
alueactually in the index. The supremum is not a real index record, so, in eff
ect, this next-key lock locks only the gap following the largest index value.

요악하면 인덱스의 가장 큰 값 위쪽의 갭을 모두 잠그게 된다는 말이다. 실제로 데이터를 삽입하면서 확인해보니 잠근 데코드의 PK 컬럼값 보다 대략적으로 더 높은 값들로 INSERT 시 데드락이 발생한다.

GAP LOCK 은 어떤 락과도 호환되지 않는다. 즉 INSERT 시 X락을 걸게되고 서로 락이 해제될때 까지 대기하다가 데드락이 발생하게 되는 이유 인 것 같다.

결국 `select for update`의 비관적 락을 사용할때는 INSERT 문과 같은 트랜잭션에서 사용하면 데드락 발생 확률이 높기 때문에 최대한 쓰는 것을 고려해야 할 것 이다.

참고
---
[https://medium.com/daangn/mysql-gap-lock-%EB%91%90%EB%B2%88%EC%A7%B8-%EC%9D%B4%EC%95%BC%EA%B8%B0-49727c005084](https://medium.com/daangn/mysql-gap-lock-%EB%91%90%EB%B2%88%EC%A7%B8-%EC%9D%B4%EC%95%BC%EA%B8%B0-49727c005084)

