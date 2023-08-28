# Kotlin Inline Class


Kotlin 의 Inline Function 과 Inline Class 에 대해 알아보자.

> 참고로 1.4.x 버전까지는 `inline class` 로 사용되어 왔는데 1.5.x 버전이 되면서 `inline -> value` 와 `@JvmInline을 붙여서 사용하도록 권장` 되게 변경되었다.
> 그 이유는 Kotlin 에 inline function 이 별도로 존재하기 때문에 이것과 혼란이 야기될 수 있어 변경되었다고 한다.

## Value Class(inline Class)

  ~~~
  @JvmInline
  value class Password(val p : String)
  ~~~

  - Inline Class 는 다른 변수와 혼동되지 않기 위해 값을 래핑할 수 있는 클래스
  - value 로 선언함으로써 컴파일러가 객체를 생성하지 않고 값을 매핑 해줌
