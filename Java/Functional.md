
# Functional Interface(함수형 인터페이스)

  함수형 인터페이스는 1개의 추상메소드를 갖고 있는 인터페이스를 말합니다. Single Abstract Method 라고 부르기도 한다.
  
  Java8 부터 인터페이스는 기본 구현체를 포함한 디폴트 메소드(default method)를 포함할 수 있습니다.
  
  여러개의 디폴트 메소드가 있더라고 `추상 메소드가 오직 하나`이면 함수형 인터페이스 입니다.
  
  자바의 람다 표현식은 함수형 인터페이스로만 사용 가능 합니다.
  
  - 즉, 추상 메소드가 하나라는 뜻은 `default method` 또는 `static method`는 여러개 존재해도 상관 없다는 뜻
  - `@FucntionalInterface` 어노테이션을 사용하는데, 이 어노테이션은 해당 인터페이스가 함수형 인터페이스 조건에 맞는지 검사해준다.

~~~java
@FunctionalInterface
interface CustomInterface<T> {

    // abstract method 오직 하나
    T myCall();

    // default method 는 존재해도 상관없음
    default void printDefault() {
        System.out.println("Hello Default");
    }

    // static method 는 존재해도 상관없음
    static void printStatic() {
        System.out.println("Hello Static");
    }
}

~~~

## Java 에서 기본적으로 제공하는 함수형 인터페이스

  __Predicate__ : `T -> boolean`<br>
  __Consumer__ : `T -> void`<br>
  __Supplier__ : `() -> T`<br>
  __Function<T,R>__ : `T -> R`<br>
  __Comparator__ : `(T, T) -> int`<br>
  __Runnable__ : `() -> void`<br>
  __Callable__ : `() -> T`<br>
  
  ### 1. Predicate
  
  `Predicate`는 인자 하나를 받아서 boolean 타입으로 리턴, 람다식은 `T -> boolean`
  
  ~~~java
  @FunctionalInterface
  public interface Predicate<T> {
    boolean test(T t);
  }
  ~~~
  
  ### 2. Consumer
  
  `Consumer`은 인자 하나를 받고 아무것도 리턴하지 않는다. 람다식 `T -> void`로 표현<br>
  소비자라는 이름에 걸맞게 무언가(인자)를 받아서 소비만 하고 끝낸다고 생각하면 된다.
  
  ~~~java
  @FunctionalInterface
  public interface Consumer<T> {
    void accept(T t);
  }
  ~~~
  
  ### 3. Supplier
  
  `Supplier` 는 아무런 인자도 받지 않고 T 타입의 객체를 리턴한다. 람다식 `() -> T` 로 표현<br>
  공급자라는 이름처럼 아무것도 받지 않고 특정 객체를 리턴한다.

  ~~~java
  @FunctionalInterface
  public interface Supplier<T> {
    T get();
  }
  ~~~
  
  ### 4. Function
  
  `Function` 은 T 타입 인자를 받아서 R 타입을 리턴합니다. 람다식은 `T -> R`로 표현<br>
  수학식에서의 함수처럼 특정 값을 받아서 다른 값을 반환해준다. T,R은 같은 타입도 가능
  
  ~~~java
  @FunctionalInterface
  public interfacee Function<T, R> {
    R apply(T t);
  }
  ~~~
  
  ### 5. Comparator
  
  `Comparator`은 T 타입 인자 두개를 받아서 `int` 타입을 리턴합니다. 람다식 `(T, T) -> int` 로 표현
  ~~~java
  @FunctionalInterface
  public interface Comparator<T>{
    int compare(T o1, T o2);
  }
  ~~~
  
  ### 6. Runnable
  
  `Runnable` 은 아무런 객체를 받지 않고 리턴도 하지 않는다. 람다식 `() -> void`로 표현
  
  ~~~java
  @FunctionalInterface
  public interface Runnable{
    public abstract void run();
  ~~~
  
  
  ### 7. Callable
  
  `Callable`은 아무런 인자를 받지 않고 T 타입 객체를 리턴, 람다식 `() -> T` 로 표현
  
  ~~~java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
  ~~~
  
  
