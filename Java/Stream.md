
# Stream

1. Java 8 추가한 스트림은 람다를 활용할 수 있는 기술 중 하나이다.
    - 스트림은 데이터 소스와 상관없이 같은 방식으로 데이터를 다룰 수 있다.(코드의 재사용성)
        - 스트림은 데이터 소스를 추상화하고, 자주 사용되는 메서드들을 정의해놓았다.
    - 스트림은 '데이터의 흐름' 이다.
        - 배열 또는 컬렉션에 함수 여러 개를 조합해서 원하는 결과를 필터링하고 가공된 결과를 얻을 수 있다.
    - 스트림은 '데이터의 흐름' 이다.
        - 배열 또는 컬렉션에 함수 여러 개를 조합해서 원하는 결과를 필터링 하고 가공된 결과를 얻는다.
    - 람다를 이용해서 코드의 양을 줄이고 간결하게 표현한다.
    - DB의 쿼리(SELECT)와 같이 정형화된 처리 패턴을 적용시킨 것.
        - 배열 혹은 컬렉션의 데이터를 쿼리(Stream와 동일하다고 봐도 무방)하는 것.
        

## Stream API 연산 종류

1. 생성하기
    - Stream 객체를 생성
    - Stream 은 재사용불가 , 닫히면 다시 생성해야 함
        - Stream 연산하기 위해 Stream 객체를 생성. 배열, 컬렉션, 임의의 수, 파일 등 거의 모든것을 가지고 스트림 생성.
2. 가공하기
    - 원본의 데이터를 별도의 데이터로 가공하기 위한 중간 연산
    - 연산 결과를 Stream으로 다시 반환하기 때문에 연속해서 중간 연산을 이어갈 수 있다.
3. 결과 만들기
    - 가공된 데이터로부터 원하는 결과를 만들기 위한 최종 연산
    - Stream의 요소들을 소모하면서 연산이 수행되기 때문에 1번만 처리가능하다.

## Collection의 Stream 생성

Collection 인터페이스에는 stream()이 정의되어 있기 때문에, Collection 인터페이스 구현한 객체들 List, Set 등 모두 이 메소드를 이용해 Stream 생성 가능.

~~~java
List<String> list = Arrays.asList("a","b")
Stream<String> listStream = list.stream();
~~~

## 가공하기

1. map
    - map은 요소들을 특정 조건에 해당하는 값으로 변환
        - 요소들을 대,소문자 변형등 작업을 하고싶을때 사용 가능
        
        ~~~java
        list.stream().map(s-> s.toUpperCase())
        							.collect(Collectors.toList());
        ~~~
   flatMap은 기존 2차원 이상의 배열을 조작시 쓰인다는 점에서 map과 차이가 있다.
   flatMap은 중첩구조를 한 단계 제거하고 단일 컬렉션으로 만들어주는 기능을 수행한다.
   
   String[][] nameArray = new String[][]{
                {"kim", "hyoz"}, {"lee", "sooyoung"},
                {"joy", "taehui"}, {"woo", "park"}
   };
   
   Arrays.stream(nameArray)
          .flatMap(innerArray -> Arrays.stream(innerArray))
          .filter(name-> name.length() <= 3)
          .collect(Collectors.toList());
        
2. filter는 요소들을 조건에 따라 걸러내는 작업을 해준다.
    - 길이의 제한, 특정문자포함 등 작업 가능
        
        ~~~java
        list.stream().filter(s -> s.length()>5)
        ~~~
        
3. sorted는 요소들을 정렬해주는 작업
    - 요소들 가공이 끝났다면 리턴 해줄 결과를 collect를 통해 만든다
    
4. Distinct - 중복제거
    - Stream의 요소들에 중복된 데이터가 존재하는 경우, 중복을 제거하기위해 Distinct를 사용 할 수 있다.
    - distinct는 중복된 데이터를 검사하기 위해 Object의 equals()메소드를 사용한다.
      ~~~java
      List<String> list = Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift", "Java"); 
      Stream<String> stream = list.stream() .distinct()
      ~~~
    - 만약 생성한 클래스를 Stream으로 사용 하면 equals 와 hashCode를 오버라이드 해야만 distinct()를 제대로 적용할 수 있다.
 
 
 5. Peek - 특정 연산 수행
    - Stream의 요소들을 대상으로 Stream에 영향을 주지 않고 특정 연산을 수행하기 위한 peek함수가 존재한다. peek 함수는 Stream의 각각의
      요소들에 대해 특정 작업을 수행할 뿐 결과에 영향을 주지 않는다.
      또한 peek 함수는 파라미터로 함수형 인터페이스 Consumer를 인자로 받는다. 예를 들어 stream 요소들을 중간에 출력하기 원할때 쓴다.
      ~~~java
      int sum = IntStream.of(1,3,5,7,9)
                .peek(System.out::println)
                .sum();
      ~~~
  
  6. mapToint, mapTolong, mapToDouble
    ~~~java
    List<Integer> integerList = Arrays.asList(20,30,50,88,100);
    int[] mapToInt = integerList.stream().mapToint(x->x).toArray();
    
    Stream.of("a1", "a2", "a3")
          .map( s-> s.substring(1))
          .mapToint(Integer::parseInt)
          .max()
          .ifPresent(System.out::println); // String -> Int
   ~~~
