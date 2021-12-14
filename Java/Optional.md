
# Optional
  
  ### 1. Null 의 늪
  
   - null 개념은 1965년 Tony Hoare라는 영국의 컴퓨터 과학자에 의해서 처음 고안되었다.
   - `존재하지 않는 값` 을 표현할 수 있는 가장 편리한 방법이 null 참조라고 생각했다고 한다.
   - 하지만 나중에 그는 그당시 자신의 생각이 "10억불 짜리 큰 실수" 였고, null 참조를 만든 것을 후회한다고 함.


  ### 2. NPE(NullPointerException)
  
   - null 참조로 인해 자바 개발자들이 가장 골치아프게 겪는 문제는 널 포인터 예외(NPE) 이다.
   - NPE는 코드베이스 곳곳에 깔려있는 지뢰같은 녀석이다. 컴파일 타임에서는 조용히 잠복했다가 런타임 때 터지기 때문이다.
 ~~~java
   /* 주문을 한 회원이 살고 있는 도시를 반환한다 */
      public String getCityOfMemberFromOrder(Order order) {
          return order.getMember().getAddress().getCity();
          }
 ~~~
   
   위코드는
   
   `order` 파라미터에 null값이 넘어옴
   
   `order.getMemeber()` 의 결과가 null 임
   
   `order.getMember().getAddress()` 의 결과가 null 임
   
   `order.getMember().getAddress().getCity()` 의 결과가 null 임
   
   위 네 가지 경우 모두 NPE의 위험을 전파시키는 포인트들 이다. 호출부에서 null을 처리해주지 않으면 NPE를 발생시킬 수 있다.
   
   
   ## JAVA8 이전의 NPE 방어 패턴
   
   JAVA8 이전에는 아래 코드 처럼 NPE를 막기위해 null 체크를 항상 해주어 코드 스타일이 매우 난잡하다.
   
   중첩 null 체크하기
 
  ~~~java
    public String getCityOfMemberFromOrder(Order order) {
            if (order != null) {
              Member member = order.getMember();
              if (member != null) {
                Address address = member.getAddress();
                if (address != null) {
                  String city = address.getCity();
                  if (city != null) {
                    return city;
                  }
                }
              }
            }
            return "Seoul"; // default
           }     
   ~~~
   
  모든 단계마다 null이 반환되지 않을지 의심하면서 null 체크를 합니다. 들여쓰기 때문에 코드를 읽기가 매우 어려우며 핵심 비즈니스 파악이 쉽지 않습니다.


## Optional

 `Optional`는 "존재할 수도 있지만 안 할 수도 있는 객체" 즉 "null이 될 수도 있는 객체" 를 감싸고 있는 일종의 래퍼 클래스 입니다.
 
 원소가 없거나 최대 하나 밖에 없는 Collection 이나 Stream으로 생각해도 된다. 직접 다루기에 위험하고 까다로운 null을 담을 수 있는 특수한
 
 그릇으로 생각하면 이해하기 쉽다.
 
 
 ### Optional 의 효과
 
 - NPE를 유발할 수 있는 null을 직접 다루지 않아도 된다.
 - 수고롭게 null 체크를 직접 하지 않아도 된다.
 - 명시적으로 해당 변수가 null 일 수도 있다는 가능성을 표현 할 수 있습니다.(따라서 불 필요한 방어 로직을 줄일 수 있습니다.)


 ## Optional 사용하기
 
  제네릭을 제공하기 때문에 변수를 선언할 때 명기한 타입 파라미터에 따라서 감쌀 수 있는 객체의 타입이 결정된다.
  
  ![image](https://user-images.githubusercontent.com/79154652/145735291-7e279ab1-3752-40cf-b8a9-5ece222fa955.png)
  
  변수명은 그냥 클래스의 이름을 사용하기도 하지만 "maybe"나 "opt"와 같은 접두어를 붙여서 Optional 타입의 변수라는 것을 명확히 나타냄
  
  Optional 객체 내에 값이 있는지 확인하기 위해 `isPresent()` 메소드를 사용할 수 있다.
  
 ## Optional 변수 선언하기

  Optional 클래스는 간편하게 객체를 생성 할 수 있도록 3가지 정적 팩토리 메소드를 제공합니다.
  
 `Optional.empty()`
 
 null을 담고 있는, 한 마디로 비어있는 Optional 객체를 얻어옵니다. 이 비어있는 객체는 Optional 내부적으로 미리 생성해놓은 싱글턴 인스턴스입니다.
 
~~~java
Optional<FileMove> optFileMove = Optional.empty();
~~~

 `Optional.of(value)`
 
 null이 아닌 객체를 담고 있는 Optional 객체를 생성합니다. null이 넘어올 경우, NPE를 던지기 때문에 주의해서 사용해야 한다.
 
 ~~~java
 Optional<FileMove> optFileMove = Optional.of(file);
 ~~~   
 
 
`Optional.ofNullable(value)`

 null인지 아닌지 확신할 수 없는 객체를 담고 있는 Optional 객체를 생성합니다. `Optional.empty()`와 `Optional.ofNullable(value)`를 합쳐놓은
 
 메소드라고 생각하시면 됩니다. null이 넘어올 경우, NPE를 던지지 않고 `Optional.empty()` 와 동일하게 비어있는 Optional 객체를 얻어옵니다.
 
 `해당 객체가 null인지 아닌지 자신이 없는 상황에서는 이 메소드를 사용해야 한다.`
 
 
~~~java
Optional<FileMove> optFileMove = Optional.ofNullable(file);
Optional<FileMove> optNotFileMove = Optional.ofNullable(null);
~~~
 
 ### Optional 이 담고 있는 객체 접근
 
 Optional 클래스는 담고 있는 객체를 꺼내오기 위해서 다양한 인스턴스 메소드를 제공한다. 아래 메소드들은 모두 Optional 이 담고 있는 객체가
 
 존재할 경우 동일하게 해당 값을 반환합니다. 반면에 Optional이 비어 있는 경우(즉, null을 담고 있는 경우), 다르게 동작합니다. 따라서 비어있는 
 
 Optional에 대해서 다르게 작동하는 부분만 설명 드리겠습니다.
 
 - `get()`
    
    비어있는 Optinal 객체에 대해서, `NoSuchElementException` 을 던집니다.
    
 - `orElse(T other)`
    
    비어있는 Optional 객체에 대해서, 넘어온 인자를 반환합니다.
    
 - `orElseGet(Supplier<? extends T> other)`

    비어있는 Optional 객체에 대해서, 넘어온 함수형 인자를 통해 생성된 객체를 반환합니다. `orElse(T other)`의 게으른 버전. 비어있는 경우에만
    
    함수가 호출되기 때문에 orElse(T other) 대비 성능상 이점을 기대할 수 있다.
    
 - `orElseThrow(Supplier<? extends X> exceptionSupplier)`

    비어있는 Optional 객체에 대해서, 넘어온 함수형 인자를 통해 생성된 예외를 던집니다.
    
    
### isPresent(), isEmpty()

메소드에서 반환된 Optional 객체가 있거나 직접 생성한 경우 isPresent() 메소드를 사용하여 값이 있는지 확인.

~~~java
Optional<FileMove> optOf = Optional.of(file);
assertTrue(optFileMove.isPresent());

Optional<FileMove> optOfNullable = Optional.ofNullable(null);
assertFalse(optOfNullable.isPresent());
~~~

### ifPresent()

ifPresent() 메소드를 사용하면 null이 아닌 경우의 코드를 실행할 수 있다.

~~~java
if(file != null) {
    System.out.println(file.length());
 }
~~~

위 코드는 내부 로직을 수행하기 전 file 변수가 null 인지 확인합니다. 이 방법은 오류가 발생하기 쉬운데 변수를 선언한 다음에 변수에 대한 

null 검사를 수행하는 것을 잊어버릴 수 있다. 따라서 아래와 같은 코드는 입력 데이터 체크에 대한 프로그래밍 적인 문제로 NPE를 발생시킬 수 있다.


~~~java
Optional<FileMove> optOf = Optional.of(file);
optOf.ifPresent(name -> System.out.println(name.length());
~~~

위와 같은 코드로 Optional의 좋은 프로그래밍 관행을 유지하면서 nullable 값을 명시적으로 처리할 수 있다.

Optional을 정확히 이해하고 제대로 사용할 수 있는 개발자라면 아래와 같이 한 줄의 코드로 작성 할 수 있어야 한다. 다시말해서 기존의 조건문으로 null을 대하던 생각을 함수형 사고로 완전히 새롭게 바꿔야 한다.

~~~java
int length = Optional.ofNullable(getFile()).map(String::length).orElse(0);
~~~

## 참고문서 
---

![링크](https://www.daleseo.com/java8-optional-after/)
