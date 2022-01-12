
# Redis

  인메모리 데이터 저장소라고 부르는 Redis 에 대해 간략하게 공부한 내용을 기록해볼려고 한다.
  
  ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbrCdyD%2FbtqWBAZA5rE%2FPgs4thI67EvRWDRis29Jgk%2Fimg.png)
  
  Redis는 외부에 key-value 를 저장하는 서버를 말한다. `인메모리` 데이터 구조 저장소로 데이터베이스, `캐시`, 메시지 브로커로 사용한다고 말한다.


 여기서 인메모리에 대해 잠시 공부해보고 가자.
 
 ## 인메모리(In-memory)?
 
 ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FBcwuU%2FbtqWKYSRi5Q%2FjRblmXQlGtIdD4nURFkHWk%2Fimg.png)
 
 인메모리란 컴퓨터의 메인 메모리 `RAM`에 데이터를 올려서 사용하는 방법을 말한다. 왜 메모리에 데이터를 올릴까? 이유는 명확하게도 속도 때문이다.
 
 SSD, HDD 같은 저장공간에서 데이터를 가져오는 것보다 `RAM`에 올려진 데이터를 가져오는데 걸리는 속도가 수백배(HDD 기준) 이상 빠르다. 때문에 Redis 는 `빠른 속도`가 큰 장점이다.
 
 ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcJNBig%2FbtqWzsOg5he%2FX3tHPGyI7obqDSlc9LDokk%2Fimg.png)
 
 Redis는 빠른 속도를 자랑하는 대신 치명적인 단점이 있다. 바로 `용량` 이다. 내 노트북만 봐도 RAM 용량은 8GB 이다. 그래서 메인 데이터베이스로 사용하기에는 무리가 있다.
 
 그렇다고 RAM을 10TB씩 구매하자니 비용이라는 걸림돌이 있다.
 
 또 다른 특징으로는 Key-Value 형티의 NoSql 이라는 점이다. Redis가 다양한 형태의 데이터 구조를 지원하기는 하지만 복잡한 데이터를 저장하는 DB로 사용하기에는 어려움이 있다.
 
 즉, Redis는 보조적인 수단으로 사용되는 경우가 많고 가장 적합한 역할은 `캐시 데이터베이스 서버`이다.
 
 ex)게임의 랭킹 상위 100위를 보여주는 기능이 있다고 해보자. 랭킹 정보를 사용자에게 제공하기 위해 오라클같은 관계형 데이터베이스에 랭킹 정보를 저장을 하고 order by 로 불러올 수 있다. 
 그런데 사용자 수가 폭발적으로 증가해 수백만명으로 늘어나게 되면 어떻게 될까? 보여주는건 100명의 데이터뿐이지만 시간이 점점 오래 걸리게 될 것이다.
 이러한 문제를 해결하기 위해 캐시에 상위 100명의 랭킹정보를 담고 있다면 좋은 해결책이 될 것이다
