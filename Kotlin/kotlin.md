 # Kotlin 

 Kotlin 에서 JPA 를 사용하여 ManyToOne 속성의 지연로딩으로 걸어놨을 때 ManyToOne 속성의 데이터를 가져올 때 SQL이 실행되어야 한다.
 
 하지만 Kotlin 같은 경우에는 지연로딩이 제대로 작동하지 않는다.
 
 그 이유는
 
 Kotlin 클래스는 final(상속 불가)이 기본 프록시 클래스를 생성하려면 클래스가 상속이 가능해야 한다.

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
  
  - 람다의 리턴
    
    - 마지막 파라미터가 람다일 때 `funtionName { it > 3.22}` 와 같이 `()`를 생략할 수 있다. 코틀린에서 자주 마주치는 녀석이다.
  ~~~kotlin
  
  val calculateGrade : (Int) -> String {
  
      when(it) {
          in 0..40 -> "fail"
          in 41..80 -> "success"
          in 80..100 -> "great"
          else -> "Error"
      }
  }
  
  ~~~
  
  ~~~kotlin
  
  fun invokeLamda(lamda : (Double) -> Boolean) : Boolean {
    return lamda(5.323)
  } 
  
  
  val lamda = {number : Double ->
   number == 4.3213 
   }
  
  ~~~
  
  ## companion object 
  
  - Java 의 Static 과 같은 것이라고 생각하면 된다.
  ~~~java
  
  public class Person{
    public static final int MAX_AGE = 100;
  
  }
  
  // ->> kotlin
  
  class Person{
    companion Object {
       const val MAX_AGE : Int = 100;
    }
  
  }
  ~~~
  
  ~~~java
  public class Perons{
    public static void hello(){
       System.out.println("1");
    }
  
  }
  
  // -> to kotlin
  
  
  class Person{
    companion Object{
      fun hello (){
         println("1");
      }
    }
  }
  ~~~
  

  ## object class
  
  - object 클래스는 SingleTon 패턴이라고 생각하면 된다.
  - 무조건 프로젝트가 실행될때 1번만 생성한다.
  
