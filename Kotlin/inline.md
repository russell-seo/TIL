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

위 처럼 alarm 이라는 메소드에 Duration 이라는 Class 를 파라미터로 받으면서 한 번에 2초라는 사실을 알 수 있다.

하지만 `data class` 를 사용하면 매번 alarm 을 호출 할 때 마다 Duration 이라는 객체를 생성해서 할당한다.

그렇기에 추가적으로 Heap 메모리에 할당함으로 런타임 오버헤드가 발생하며 이를 `Value Class` 로 변환해서 처리하는 것을 권장한다.




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
