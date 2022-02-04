
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
    
