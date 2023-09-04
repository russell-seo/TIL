# Kotlin Inline Class


Kotlin 의 Inline Function 과 Inline Class 에 대해 알아보자.

> 참고로 1.4.x 버전까지는 `inline class` 로 사용되어 왔는데 1.5.x 버전이 되면서
>
> `inline -> value` 와 `@JvmInline`을 붙여서 사용하도록 권장 되게 변경되었다.
>
> 그 이유는 Kotlin 에 inline function 이 별도로 존재하기 때문에 이것과 혼란이 야기될 수 있어 변경되었다고 한다.

## Value Class(inline Class)

  ~~~kotlin
  @JvmInline
  value class Password(val p : String)
  ~~~

  - Inline Class 는 다른 변수와 혼동되지 않기 위해 값을 래핑할 수 있는 클래스
  - Value Class 는 단 1개의 내부 필드만을 가질 수 있다.
  - value 로 선언함으로써 컴파일러가 객체를 생성하지 않고 값을 매핑 해줌
  - 왜 Value(inline) Class 를 사용할까? data Class 를 사용해도 되는데
    - 비즈니스 로직을 위해 특정 type을 Wrapper로 감싸줘야 할 때가 있는데, 이 때 `객체를 생성하기 위해 Heap 에 할당을 하게됨으로써 런타임 오버헤드가 발생`
    - 즉 성능 저하가 일어나기 때문에 inline Class 를 사용하라고 공식문서 에서 설명해주고 있다.




## Code

`data class`
~~~kotlin
fun alarm(duration : Durations) {
  prtinln(duration.seconds)
}

fun test(){
  alarm(Duration.seconds(2)
}

data class Durations(
  val millis : Long
){

  companion object {
    fun seconds(sec : Long) = Duration(sec * 1000)
  }
}
~~~

위 코드는 Value(inline) Class 대신 data class 를 사용해서 작성한 코드이다.

위 처럼 alarm 이라는 메소드에 Duration 이라는 Class 를 파라미터로 받으면서 한 번에 2초라는 사실을 명확하게 알 수 있다.

하지만 `data class` 를 사용하면 매번 alarm 을 호출 할 때 마다 Duration 이라는 객체를 생성해서 할당한다.

그렇기에 추가적으로 Heap 메모리에 할당함으로 런타임 오버헤드가 발생하며 이를 `Value Class` 로 변환해서 처리하는 것을 권장한다.

### data class to decompile

~~~kotlin
public static final void alarm(@NotNull Duration duration) {
      Intrinsics.checkNotNullParameter(duration, "duration");
      String var1 = duration.getMillis();
}
~~~

`Java Code`로 디컴파일 했을 시 alarm 클래스의 파라미터로 Duration 객체가 들어온다.




`Java Code`로 디컴파일 했을 시 alarm 클래스의 파라미터로 `long` 값이 들어오는 것을 볼 수있으며, 이 long 값은 `Duration 내의 프로퍼티` 타입을 그대로 가져온다.

`value(inline) class`
~~~kotlin
fun alarm(duration : Durations) {
  prtinln(duration.seconds)
}

fun test(){
  alarm(Duration.seconds(2)
}

@JvmInline
value class Durations(
  val millis : Long
){

  companion object {
    fun seconds(sec : Long) = Duration(sec * 1000)
  }
}
~~~

`value class` 로 선언된 Duration 은 파라미터로 들어올 때 객체의 Property로 대체된다.
`value class` 는 클래스 내에 프로퍼티와 함수를 정의할 수 도 있다.

### value class to decompile
~~~kotlin
public static final void alarm_ZlhV6Ys(long duration) {
      String var2 = duration;
   }
~~~


### JPA Entity AI Column with value class

JPA 에서 AI 컬럼을 value class로 Long -> value class 로 사용

> JapRepository<User, UserId> 를 상속받아 구현체를 등록할 때 findById() 메소드로 Entity를 가져올 때 TypeMisMatch Error가 생긴다.
>
> StackOverflow 를 찾아봐도 해당 에러에 대한 해결법은 찾을 수 없었다.
>
> 그래서 QueryDsl로 id 값을 가져오는 메소드를 작성하거나, JapRespository<User, Long> 을 상속받는 Repository 를 하나 더 구현하는 방법이 있다.

~~~kotlin
@Entity
class User(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id : UserId,

    val name : String
) {


}

@JvmInline
value class UserId(
    val id : Long? = null
) : Serializable
~~~

~~~kotlin
@Repository
interface UserReadRepository : JpaRepository<User, Long>{
}

@Repository
interface UserCmdRepository : JpaRepository<User, UserId>{
}
~~~

- Entity를 findById()같이 조회할려고 하면 첫번째 ReadRepository 를 사용
- Entity를 Insert, Update 할 때는 두번째 CmdRepository 를 사용

QueryDsl
~~~kotlin

@Autowired
    lateinit var entityManager: EntityManager

    @Bean
    fun query() : JPAQueryFactory {
        return JPAQueryFactory(entityManager)
    }

    fun where() : BooleanBuilder {
        return BooleanBuilder()
    }

    fun findById(): User? {
        val user =  query().selectFrom(user)
            .where(user.id.eq(1L)).fetchOne()
        return user
    }
~~~

- JpaRepository 를 사용하지 않는다면, 위와 같이 QueryDsl로 작성하여 findById()를 대체해서 사용 가능하다.



## 마무리

이처럼 단일 프로퍼티에 대해서는 일반 타입과 거의 같게 사용될 수 있지만,

내부적으로 유효성 검사를 할 수 있으며 또한 코틀린 코드에서는 명백하게 타입을 갖기 때문에 효용성에 있어서 코드를 작성하는데 큰 도움이 된다고 생각한다.

하나 변수를 VO로 감싸는데 부담을 느낀다면 어느정도 적절한 트레이드 오프를 가진 `value class`를 사용해보는건 어떨까싶다


