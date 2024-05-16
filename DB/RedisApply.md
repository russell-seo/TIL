# Redis 의 자료구조

Redis 를 사용하기 전에 Redis 가 데이터를 어떻게 저장하는지 부터 살펴볼려고 한다.

- Redis의 가장 기초적인 자료구조는 `Key/Value` 형태를 저장하는 것이다.(String타입) 이라고도 한다. 이를 위해 Redis 는 `Bucket을 활용한 Chained Linked List 구조` 를 사용한다.
- 최초에는 4개의 Bucket 에서 사용하며, 같은 Bucket에 들어가는 Key는 `Linked List` 형태로 저장하게 된다.
  
![image](https://github.com/russell-seo/TIL/assets/79154652/ebdb5598-034f-43a8-aaa0-6251fbebe412)

이 Chanined Linked List 에는 약점이 있다. 한 `Bucket`안에 데이터가 많아지면 결국 탐색 속도가 느려지게 된다. 이를 위해서 Redis 는 특정 사이즈가 넘을때 마다 Bucket을 두배로 확장하고

Key들을 `rehash`하게 된다.

아래의 코드는 hash 값이 들어가야 할 hash table 내의 index 를 결정하는 방법은 아래와 같다.

~~~
/* Returns the index of a free slot that can be populated with
 * a hash entry for the given 'key'.
 * If the key already exists, -1 is returned.
 *
 * Note that if we are in the process of rehashing the hash table, the
 * index is always returned in the context of the second (new) hash table. */
static int _dictKeyIndex(dict *d, const void *key)
{
    ......
    h = dictHashKey(d, key);
    for (table = 0; table <= 1; table++) {
        idx = h & d->ht[table “” not found /]
.sizemask;
        ......
    }
    return idx;
}
~~~

table 에는 key 를 찾기 위해 비트 연산을 하기 위한 sizemask 가 들어가 있다. 초기에는 table의 bucket이 4개 이므로 sizemask는 이진수로 11즉 3의 값을 셋팅하게 된다.

즉 해시된 결과 & 11의 연산 결과로 들어가야 하는 Bucket이 결정되게 된다.

`여기서 Key 가 많아지면 Redis는 테이블의 사이즈를 2배로 늘리게 된다. 그러면 당연히 sizemask 도 커지게 된다. table size가 8이면 sizemask 는 7이 된다.`

먼저 간단하게 말하자면 Redis 에서 사용하는 Keys 대신에 사용해아는 Scan 의 원리는 이 Bucket을 한 턴에 하나씩 순회하는 것이다. 그래서 아래 그림과 같이 처음에는 Bucket index 0 을 읽고 데이터를 던져주는 것이다.

![image](https://github.com/russell-seo/TIL/assets/79154652/fdf1e668-c921-41bb-bfc0-ae07e280845b)


# Redis Spring 프로젝트에 적용하기

  회사 프로젝트에 Redis 실제로 적용해보았다.
  
  
  ## 프로젝트에 캐싱 구현하기
  
  
   __Redis 설정__
   
   ~~~java
@Configuration
@PropertySource("classpath:db.properties")
public class RedisConfig {

        @Value("${spring.redis.host}")
        private String redisHost;

        @Value("${spring.redis.port}")
        private int redisPort;


        @Bean
        public RedisConnectionFactory redisConnectionFactory(){
            LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(redisHost, redisPort);
            return lettuceConnectionFactory;
        }

        @Bean
        public RedisTemplate<String,Object> redisTemplate(){
            RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
            redisTemplate.setConnectionFactory(redisConnectionFactory());
            redisTemplate.setKeySerializer(new StringRedisSerializer());
            redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
            return redisTemplate;
        }

        /*
          Redis Cache 적용을 위한 RedisCacheManager 설정
        */
        @Bean
        public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory){
            RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                    .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new             GenericJackson2JsonRedisSerializer()))
                    .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer())).entryTtl(Duration.ofMinutes(30));

            return RedisCacheManager
                    .RedisCacheManagerBuilder
                    .fromConnectionFactory(redisConnectionFactory)
                    .cacheDefaults(redisCacheConfiguration)
                    .build();

        }

}
   ~~~
   
   - `RedisConnectionFactory` : 캐시 데이터 저장소로 Redis를 결정하였으므로, Redis 서버에 접속하기 위한 ConnectionFactory가 필요하다.
   - `CacheManager` : 어떤 Cache 저장소에 데이터를 저장할지 설정해주는 객체.
   
   > Redis는 보통 외부에 저장소를 따로 위치해 사용하며, 네트워크 통신을 위해 Byte Array(바이트 배열)로 변환해야 한다.(OutStream으로 내보내기 위함)
   또한, WAS 관점에서 Redis에 저장된 데이터도 단지 byte일 뿐이다. 그러기 때문에 불러올때도 바이트 배열을 객체로 변환해줘야 한다.
   쉽게말해, 객체와 JSON 간의 직렬화/역직렬화 설정을 해주어야 한다.
   
   
   캐싱을 적용하는 것은 간단하다. 캐싱을 원하는 메소드에 어노테이션을 붙여주기만 하면 된다.
    
   ![image](https://user-images.githubusercontent.com/79154652/152315898-d8e99cc4-b152-4454-b95b-a6f14ef7d9c1.png)

   - `@Cacheable` : 조회 캐시 설정 어노테이션
      - key : 저장소에 저장될 때의 key(프로젝트 에는 파라미터 필드의 update_date 를 사용했다.)
      - value : 저장소에 저장될때의 그룹명
      - condition : SpEL을 사용하여 캐싱조건 설정
      - unless : DB로부터 받아온 값에 따라 캐싱조건 설정(프로젝트에는 DB에는 받은 값이 없으면 캐시되지 않도록 설정)
    
