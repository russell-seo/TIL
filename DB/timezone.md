# 글로벌 서비스 헷갈리는 타임존


 - 필자는 글로벌 서비스를 하는 게임회사에서 게임 개발을 제외한 많은 일을 하고있다. 이 글은 글로벌 서비스를 하는 회사에서 개발자들이 항상 헷갈려하는 `타임존`에 대해 이야기 해 볼까 한다.

- 필자도 이 타임존에 대해 글로벌 서비스를 하는 조직에서 처음으로 알게되었고 기록해 놓기로 하였다.


## 현재 직면한 문제

- 현재 필자가 직면한 문제는 Mysql DB에 날짜 타입이 datetime 으로 잡혀있기 때문이다. 이게 문제가 되는 이유는 DateTime은 타임존을 포함하고 있지 않고 DB에 저장되기 때문이다.
- 만약 서버에서 TimeZone을 Asia/Seoul로 `2022-11-25 11:00:00`을 DB에 저장하면 Asia/Seoul 기준으로 DB에 저장되고 불러올때도 `2022-11-25 11:00:00`으로 데이터를 가져올 수 있다.

> 이게 무슨 문제가 되냐는 의문이 들 수도 있다. 하지만 만약 TimeZone을 `Asia/Seoul`에서 `America/New_York`으로 불가피하게 변경해야 한다고 가정해보자. 그러면 현재 DB에 저장되어있는 `2022-11-25 11:00:00`은 `Asia/Seoul`기준으로 데이터가 저장되었고 TimeZone을 `America/New_York`으로 변경하더라도 동일한 `2022-11-25 11:00:00` 데이터가 조회될 것이다. 

> 이러한 데이터 오염은 마이그레이션도 불가능한 상태로 만들 수 있기 때문이다.  그러면 어떻게 하는게 적합한가? 바로 TimeStamp 타입을 지향하는 것이다. 해당 DateTime에 대해 이야기 할려면 Native DateTime과 Aware DateTime에 대해 알아 보고 가야한다.



## Native DateTime vs Aware DateTime

- DateTime 에는 두 가지의 DateTime 이 있다. 
  - `Native Datetime`은 간단히 말해 타임존이 없는 DateTime을 말한다. `2022-11-25 12:00:00` 같은 것을 말한다. 
  - `Aware DateTime`은 타임존이 있는 DateTime을 말한다. `2022-11-25 12:00:00 +09:00` 이와 같은 타임존이 있는 시간을 말한다.

- DateTime은 글로벌 서비스를 하기에는 항상 헷갈리게 하는 것이다.

> ![image](https://user-images.githubusercontent.com/79154652/203882741-1b1491f6-343f-4ca2-9571-c59ed789b48b.png)


- 위와 같이 언급한 내용도 있다. 
   -  `TimeStamp 타입을 사용하면 Mysql 에서 현재 시간을 기준으로 UTC로 변경하여 저장하며 UTC 에서 현재 시간 기준으로 변경해서 가져온다.`
   -  `DateTime은 timeZone 정보를 잃어버릴 수 있다.`


## 그래서 TimeStamp를 어떻게 사용해야 하나?

- 날짜를 저장할때는 Unix Long 타입으로 저장하는 것이 좋은 방법중 하나라고 생각한다.
- Unix 시간이란?
  - POSIX 시간이나 Epoch 시가닝라고 부르기도 하며 32비트로 표현된 유닉스 시간은 1970년 1월 1일 00:00(UTC)을 기준으로 산정하여 계산한다.

- 필자는 Java 를 기반으로 서버를 개발하며 `Instant` 를 사용하여 Long 타입으로 변환하여 DB에 저장한다.

> Instant 는 1970년 부터 현재 시간까지 계산한 nano 초 동안의 시간이며 하나의 순간을 나타낸다. `대부분의 로직과 데이터들은 UTC와 같은 정확한 시간으로 계싼 되어야 하므로 자주 사용하는 클래스이다.`

- DB의 데이터타입이 TimeStamp 이면 UTC 기준으로 저장되며 DB에서 데이터를 불러올때 해당 타임존 기준으로 변환해서 가져오게 된다. 그렇기 떄문에 날짜에 대해 더이상 혼동하고 헷갈릴 필요 없이 글로벌 서비스를 할수 있을 것 이다.


-- 아래의 링크를 통해 DateTime 과 TimeStamp 관련해서 실습해보았고 직접 해보니 조금더 이해하기 수월하였다.
[https://medium.com/@kenny_7143/time-zone-in-mysql-e7b73c70fd4e](https://medium.com/@kenny_7143/time-zone-in-mysql-e7b73c70fd4e)
