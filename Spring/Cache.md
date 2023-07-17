# Spring Cache

  스프링의 캐시 추상화는 캐시 기술에 종속되지 않으며 AOP를 통해 적용되어 애플리케이션의 코드를 수정하지 않고 적용이 가능하다.

  이를 위해 스프링은 사용하고자 하는 캐시 기술을 캐시 매니저 빈으로 등록해야 한다.

  캐시 데이터를 ConcurrentHashMap에 저장하는 `ConcurrentMapCacheManager`, EhCache를 지원하는 `EhCacheManager`, `RedisCacheManger`등 다양한 캐시 매니저가 존재하며 적절하게 CacheManager로 등록하여 사용할 수 있다.

  ## Cache
  대표적으로 Cache는 CPU와 HDD/SSD입니다. `CPU의 레지스터는 우리가 사용하는 하드디스크 보다 훨씬 빠르다. 그래서 이 사이에 Cache를 두어 CPU가 빠르게 처리할 수 있도록 하는 것`

  - 스프링에서 Cache 기능을 추상화 하여 지원한다. 즉 다른 EhCache, Redis등의 캐시 저장소와 빠르게 연동하여 사용할 수 있도록 하였다.
  - `Spring Cache는 트랜잭션과 마찬가지로 AOP를 이용하여 캐싱 어노테이션이 붙어있는 메소드를 캐시 처리한다.`
    - 만약 캐시에 조회하고자 하는 데이터가 있다면 캐시해둔 결과를 `Proxy`에서 반환하기 때문에 해당 타겟 메소드의 로직은 실행되지 않는다.
  - @Cacheable(value= "", key ="") 으로 캐싱할 메소드위에 붙여주면 동작한다.

  ## @Cacheable 동작 원리

  AOP 기반으로 동작하며 스프링 빈으로 등록된 모든 빈들을 빈 후처리기로 보내 해당 빈에서 동작하는 메소드에 @Cacheable 어노테이션이 있는지 확인한다.
  만약 @Cacheable 이 적용된 메소드라면 Proxy에 의해 캐시 메모리에서 데이터가 있으면 반환한다.

  @Cacheable AOP를 적용시키기 위해 `Aspect`, `CacheAnnotionParser`, `CacheProxyFactoryBean`등을 추상화 시켰다.


  스프링에서는 인터셉터를 통해 적당한 시점에 요청을 가로채서 필요한 처리를 실행하게 되는데 Proxy를 통해 들어오는 외부 메소드 호출만 인터셉트 하는 AOP 특성상 동일 클래스내 에서 접근하는 경우 동작하지 않는  다.

  ## Spring Cache 어노테이션

  
![image](https://github.com/russell-seo/TIL/assets/79154652/d56b39fa-f710-4d98-88b7-6bca01ffb898)


스프링에서 `@EnableCaching` 어노테이션을 통해 캐시 기능을 활성화 시킬 수 있다.

실제로 이 기능을 사용해보면서 익혀보자.



  ## Configuration

  ~~~java
  @Configuration
public class RedisConfig {
    @Bean
    public RedisConnectionFactory redisConnectionFactory(){
        LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(new RedisStandaloneConfiguration("localhost", 6379));
        return lettuceConnectionFactory;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;
    }

    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory){
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .entryTtl(Duration.ofMinutes(3L));



        return RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
    }
  }
  ~~~

  
  
