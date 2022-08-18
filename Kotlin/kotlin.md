 # Kotlin 


  ## Lamda
  
  - 람다식은 우리가 마치 value 처럼 다룰 수 있는 익명 함수이다.
  - 메소드의 파라미터로 넘겨줄 수 있다. fun maxBy(a : Int)
  - return 값으로 사용할 수 있다.

  ~~~kotlin
  
  val square : (Int) -> (Int) = {number -> number * number}
  
  val square = {number : Int -> number * number}
  
  // 둘다 사용가능 타입을 정해줘야 한다.
  
  
  ~~~

  ~~~kotlin
  
  val nameAge = {age : Int , name : String ->
    "my name is ${name} I'm ${age}"
  }
  
  //마지막에 있는 줄이 리턴 값이다.
  ~~~
  
  ## 확장 함수
  
  - 기존의 함수를 Extention 해서 사용 가능하다.
  
  아래 String Class를 예시로 들어 보겠다.
  
  ~~~kotlin
  
  val pizzaIsGreat : String.() -> String = {
     this + "pizza is the best!"
  
  //함수를 정의한다.
  
  val a = "russell said"
  
  println(a.pizzaIsGreat())
  
  // russell said pizza is the best! 라는 값이 출력된다.
  }
  ~~~
  
  ~~~kotlin
  
  fun extendString(name : String, age : Int) : String {
  
      val introduceMyself : String.(Int) -> String = { "I am ${this} and ${it} years old"}
      return name.introduceMyself(age)
  }
  
  println(extendString("russell", 29))
  
  ~~~
