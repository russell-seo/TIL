
# EazyRandom 으로 테스트 객체 랜덤으로 생성하기

 - Easy Random 이란 자바 객체를 랜덤으로 만들어주는 라이브러리 이다.
 - `You can think of it as an ObjectMoter for the JVM`이라고 설명한다.
 - 여기서 [ObjectMother](https://martinfowler.com/bliki/ObjectMother.html) 은 한마디로 테스트 할때 예시 객체를 만들게 도와주는 객체의 하나라고 말할 수 있다.
 - EazyRandom 깃허브 공식 사이트에 가면 더 많은 정보를 제공한다 URL : [깃허브](https://github.com/j-easy/easy-random)



 ## Seed

 - EazyRandom 은 Seed라는 값을 통해서 랜덤 값을 생성한다. 즉 시드값은 랜덤 값 생성의 기준이 된다.
 - 동일한 시드값을 가지고 있으면 항상 동일한 결과가 나오게 된다.

![스크린샷 2023-07-30 오후 11 16 32](https://github.com/russell-seo/TIL/assets/79154652/7cb4b85f-555d-4529-922f-6b27d7b3d407)

 - 위를 보면 Default Seed 값이 잡혀있다.

  자 그러면 이제 EazyRandom을 통해서 어떻게 테스트 객체를 만들어내는지 코드로 바로 알아보자


  ## EazyRandom 사용법

  - build.gradle 에 아래 dependency를 추가하자
  - `testImplementation 'org.jeasy:easy-random-core:5.0.0'`
  - 추가하면 이제 EazyRandom 객체를 생성할 수 있게 된다.
  
  먼저 Entity 객체를 하나 아래와 같이 하나 정의했다고 가정하자

  ~~~java

@Entity
@Getter
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private final Long memberId;

    private final String contents;

    private LocalDateTime createdAt;

    @Builder
    public Post(Long id, Long memberId, String contents, LocalDateTime createdAt) {
        this.id = id;
        this.memberId = memberId;
        this.contents = contents;
        this.createdAt = createdAt;
    }
}
  ~~~


  - 이 Post Entity 의 객체를 저장하는 메소드를 구현하고 테스트에서 EazyRandom으로 임의의 객체를 생성하는 코드를 만들어보자

  PostFixtureFactory
  ~~~java
  public static Post register(){
        var param  = new EasyRandomParameters();
        return new EasyRandom(param).nextObject(Post.class);
    }

  public static Post register(Long seed){
        var param = new EasyRandomParameters().seed(seed);
        return new EasyRandom(param).nextObject(Post.class);
    }
  ~~~

  - 위의 두 메소드중 하나는 seed 값을 받아서 만드는 메소드이다. 시드값을 받지 않는 메소드는 for loop 로 생성하게 되면 동일한 객체만 생성되게 된다.
  - 위 두 메소드 처럼 그냥 `단 건의` 랜덤 객체를 생성하기 위해서는 저렇게 사용해도 된다.
  - 하지만 우리는 테스트를 작성할 때 각 데이터에 제약조건이 걸려있기 마련이다. 그래서 EazyRandomParameter를 임의의 범위값이나 커스텀해서 만들 수 있게 제공한다


  ~~~java
  EasyRandomParameters parameters = new EasyRandomParameters()
   .seed(123L)
   .objectPoolSize(100)
   .randomizationDepth(3)
   .charset(forName("UTF-8"))
   .timeRange(nine, five)
   .dateRange(today, tomorrow)
   .stringLengthRange(5, 50)
   .collectionSizeRange(1, 10)
   .scanClasspathForConcreteTypes(true)
   .overrideDefaultInitialization(false)
   .ignoreRandomizationErrors(true);

EasyRandom easyRandom = new EasyRandom(parameters);
  ~~~

  위 코드는 EazyRandom 깃허브에서 공식으로 알려주는 EazyRandomParameter에서 제공하는 데이터를 커스텀 할 수 있는 코드이다.


 필자는 아래코드와 같이 memberId 와 dateRange를 위해 두개의 날짜 데이터를 받는다. 
 
 ~~~java
 public static EasyRandom get(Long memberId, LocalDate start, LocalDate end){
        var idPredicate = named("id") //Post.class에 있는 id 값의 타입과 클래스를 설정해준다.
                .and(ofType(Long.class))
                .and(inClass(Post.class));

        var memberPredicate = named("memberId")
                .and(ofType(Long.class))
                .and(inClass(Post.class));

        var param = new EasyRandomParameters()
                .excludeField(idPredicate)// 위에서 설정한 id는 제외하고 생성한다는 조건이다.
                .dateRange(start, end) //날짜는 start -> end 사이의 랜덤 데이터를 생성
                .randomize(memberPredicate, () -> memberId); //들어오는 memberId 파라미터로 생성하는 함수 
                .randomize(memberPredicate, new LongRangeRandomizer(1L, 10000L)); // 이렇게 Long 데이터 범위도 정할 수 있다.
        return new EasyRandom(param);

    }
 ~~~

### 테스트 객체 생성하기

~~~java
@Test
void 랜덤_객체_생성(){
        var param=PostFixtureFactory.getRandom(
                LocalDate.of(2023, 1, 1),
                LocalDate.of(2023, 12, 31)
        );

        var post = IntStream.range(0, 10).mapToObj(
                i -> param.nextObject(Post.class)
        ).toList();


        for (Post post1 : post) {
            System.out.println("memberId = " + post1.getMemberId()+"/"+ post1.getCreatedAt());

        }
    }
~~~

![스크린샷 2023-07-30 오후 11 43 53](https://github.com/russell-seo/TIL/assets/79154652/dd9df51b-9942-4bee-9130-a913b932d1bf)

위와 같이 랜덤 객체가 우리가 지정한 데이터 범위에서 생성이 된다.
