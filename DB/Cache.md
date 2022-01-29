
# Cache?

  - 캐시는 접근 비용이 비싼 데이터의 `사본`을 만들어 저장하고 동일한 요청이 있을 때 원본 데이터에 접근하지 않고 사본데이터를 제공할 수 있게 하는
    중간 장치의 개념
    
  - 서비스의 데이터를 저장하기 위해 MySQL 데이터베이스를 사용하고 있는데, MySQL은 `하드디스크 기반`의 데이터 스토리지 이기 때문에 서버의 메모리보다
    약 1000배 정도의 속도 차이가 데이터 검색하는데 차이가 난다.
  
  
  <p align = "center">
  <img src = "https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbAFA4c%2FbtrevGAt1vJ%2FWb4XKy6dXWC1PYFvdXqcRK%2Fimg.png"
       width = "700" height = "400" >
  </p>
  
  
 ## Spring Cache Abstraction
 
 Spring Cache Manager의 문서에서는, 스프링이 `AOP`를 사용하여 비침투적으로 캐싱을 적용한다고 설명한다.
 
 @Transactional 어노테이션을 통해 트랜잭션 서비스를 제공하는 것과 비슷한 패턴으로 캐싱을 적용한다는 것을 알수 있다.
 
 스프링은 특정 기술을 추상화 계층으로 가져가고 개발자에게는 기술을 사용하는 인터페이스를 동일한 형태로 제공함으로써 기술의 벤더가 변경된다고 할 지 라도
 코드에는 전혀 영향을 미치지 않게 한다.
 
 이를테면 스프링 캐시를 관리하는 `CacheManager`에는 ConcurrentMapCacheManager, SimpleCacheManager, EhCacheManager, RedisCacheManager 등 다양한 기술 벤더가
 존재하는데, 이들 모두 `CacheManager` 인터페이스를 구현하고 있기 때문에 우리는 동일한 시그니쳐로 캐시 기능을 사용할 수 있다.
 
 이와 같은 방법으로 프로개름이 특정 기술에 종속되지 않게 하는 방식을 `PSA(Portable Service Abstraction)` 라고 하며, 스프링은 이를 스프링의 3대 요소중 하나로 꼽습니다.
 
 
 ## Local Cahce VS Global Cache
 
 스프링의 PSA 덕분에 우리는 서비스 환경에 따라 어떤 Cache 전략이던 CacheManager를 구현하고 있기만 하다면 유연하게 우리 코드에 적용 할 수 있다.
 
 그럼 어떠한 Cache 기술을 적용할지 고려해보아야 된다. Cache 관리 전략을 선택할 때 가장 먼저 고려해야할 요소는 `캐시데이터를 저장한 스토리지`를 서버가
 자체적으로 소유하고 있을지, 외부 서버에 캐시 저장소를 따로 둘 지에 대한 부분입니다. 캐시 저장소를 서버에 두는 방식을 `Local Cache`, 외부 캐시 저장소를
 두는 방식을 `Global Cache` 라고 합니다.
 
  <p align = "center">
  <img src = "https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FblIDZb%2FbtreB7YOZZr%2Fai81I7etvTlANkQUk7Xov1%2Fimg.png"
       width = "300" height = "350" >
  </p>


## Local Cache

- 데이터 조회 오버헤드가 없다
    - 캐시 데이터를 서버 메모리상에 두는 것이 가장 큰 장점은, 무엇보다도 속도가 빠르다는 점이다. 캐시를 외부 저장소에 저장하면 네트워크 통신을 통해
      캐시 저장소에 접근하고 데이터를 가져오는 과정등의 오버헤드가 없기 때문에 Local Cache 의 데이터 읽기 속도는 현저히 빠르다.
      
- 서버 간 데이터 일관성이 깨질 수 있다.
    - Local Cache는 단일 서버 인스턴스에 캐시 데이터를 저장하기 때문에, 서버의 인스턴스가 여러 개 일 경우 서버 간 캐시 데이터가 일치하지 않아
      신뢰성을 떨어 뜨릴 수 있다.

- 서버간 동기화가 어렵고, 동기화 비용이 발생한다.
    - 캐시 일관성을 유지하기 위해 동기화를 한다고 하더라도, 추가적인 비용이 발생한다. 더군다나 서버의 개수가 늘어날수록, 자신을 제외한 모든 인스턴스와
      동기화 작업을 해야하기 때문에 비용의 크기는 서버 개수의 제곱에 비례하여 증가한다.
      

## Global Cache

- 네트워크 I/O 비용 발생
  - Global Cache는 외부 캐시 저장소에 접근하여 데이터를 가져오기 때문에, 이 과정에서 네트워크 I/O가 비용이 발생한다. 하지만 서버 인스턴스가 추가될때
    에도 동일한 비용만을 요구하기 때문에, 서버가 고도화될수록 더 높은 효율을 발휘한다.
    
- 데이터 일관성을 유지할 수 있다.
  - Global Cache는 모든 서버의 인스턴스가 동일한 캐시 저장소에 접근하기 때문에, 데이터의 일관성을 보장할 수 있습니다. 데이터의 일관성이 깨지기 쉬운
    분산 서버 환경에 적합한 구조입니다.
    
    
   ### Local Cache vs Global Cache, 어떤 기준으로 선택?
   
   - Local Cache 와 Global Cache 의 특성을 고려했을 때, 어떤 기술을 선택해야 할지에 대한 기준은 `데이터의 일관성이 깨져도 비지니스에 영향을 주지 않는가?`
     라고 생각한다.
     -  이를테면, 사용자 정보가 변경되어 프로필에 반영되어야 하는 상황을 가정할 때, 서버 간 동기화가 맞지 않아서 프로필에 반영되는데 시간이 조금 걸린다 하더라도, 전체적인 서비스 운영에 큰 타격을 주지는 않습니다. 금전이 오고가는 문제도 아니고, 프로필의 정보가 조금 늦게 반영된다고 해서 큰 문제가 발생하지 않으니까요. 이러한 경우에는 서버간 동기화 없이 서버 자체적으로 로컬 캐싱을 하는 것도 괜찮은 선택지라고 생각합니다.
     
     -  하지만, 상품 데이터를 캐싱한다고 했을 때는 상황이 달라집니다. 사용자가 가격을 변경했는데, 그것이 반영되지 않으면 서비스 신뢰를 심각하게 손상하고, 운이 나쁘면 법적 문제로 이어지기도 합니다. 따라서, 이러한 경우에는 동기화가 속도보다 더 중요하며, 그렇기에 동기화가 확실하게 보장되는 Global Cache를 사용하는 것이 좋습니다.
     
 
 # Look Aside Cache
 
  캐시에도 다양한 전략이 존재한다.
  
  대표적으로 Look Aside Cache 와 Write Back이 존재한다.
  
  - Look Aside Cache : 캐시 저장소에 데이터가 있으면, 캐시에서 가져오고 없으면 메인 DB에서 값을 가져오는 대표적인 캐시 전략.
  - Write Back : 캐시 저장소에 특정 시간동안 데이터를 모아서, 배치 처리를 통해 DB에 저장하는 전략
  
  ![image](https://github.com/binghe819/TIL/raw/master/Spring/Cache/spring%20boot%20cache%20with%20redis/image/webserver_cache.png)

- Web Server는 데이터가 존재하는지 Cache를 먼저 확인한다.
  - Cache에 데이터가 있으면 Cache에서 값을 가져온다.
  - Cache에 데이터가 없으면 DB에서 데이터를 가져와서 Cache에 저장하고 값을 가져온다.

- Write 하는 방식
  - 방법1 : 어플리케이션이 새로운 데이터 쓰기 혹은 업데이트 할 때 캐시와 DB 모두에 같은 작업을 실행하는 방법.
  - 방법2 : 어플리케이션의 모든 쓰기 작업은 DB에만 적용되고, 기존의 캐시 데이터를 무효화 시키는 방법.
