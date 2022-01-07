
# Scheduled

  __기본사항__
  
  - 메소드는 void 로 리턴값이 없어야 한다.
  - 어떠한 파라미터도 받을 수 없다.
  
    - 특정 상황에서 단독으로 동작하는 기능이기 때문에 위의 두 조건을 만족해야 한다.

 ## 스케줄링 어노테이션 활성화
    
   ~~~java
   @Configuration
   @EnableScheduling
   public class AppConfig{
   
   }
   ~~~
   위 코드와 같이 Java Config 를 사용한다면 `@EnableScheduling` 어노테이션만 붙여 주면 된다.
   
   만약 XML 을 사용 한다면 아래의 `<task:annotation-driven>` 속성을 추가해주면 된다.
   ~~~
   <task:annotation-driven executor="myExecutor" scheduler="myScheduler"/>
   <task:executor id="myExecutor" pool-size="5"/>
   <task:scheduler id="myScheduler" pool-size="10"/>
   ~~~
   
   
## 스케줄러 테스트

  ~~~java
  @Component
public class Scheduler {


    @Scheduled(fixedDelay = 10000)
    public void test(){
        long l = System.currentTimeMillis();
        System.out.println("스케줄러 테스트 = " + l);


    }
  ~~~
  
  위 코드는 간단한 `@Scheduled` 어노테이션을 적용한 메소드를 실행하였고 아래와 같이 10초 마다 메소드가 실행되는 것을 확인 할 수 있다.
  
  ![image](https://user-images.githubusercontent.com/79154652/148498462-3666244c-42b0-4d43-a6f1-ee37a4054c80.png)

 - 추가로 `cron`표현식을 통해서 원하는 시간, 날짜, 요일등 세부적인 스케줄을 등록하는 것이 가능하다.
    
   ### cron 표현식
   
   - cron 표현식은 스페이스(" ")로 구분되는 7개의 단위로 표현되는 문자열 이다.
   - 각 필드는 앞에서부터 `초`, `분`, `시`, `일`, `월`, `요일`, `년` 으로 구성된다.
      
      `,`
      - 목록의 항목을 여러 개 사용할 경우, 항목 값을 구분하는데 사용
      - 예) MON, WED, FRI -> 월, 수, 금요일 에 실행
      
      `-`
      - 범위 지정
      - 예) 1970-2099 
      
      `%`
      - 이스케이스 문자(\) 와 함께 사용하지 않는 한 개행 문자로 샤옹됨
      
      `*`
      - 와일드 카드로 모든 값 의미

      `?`
      - 설정값 없음을 의미
      - 몇몇 표현식에서는 와일드카드 대신 사용하여 월이나 요일 필드를 비워둔다.

      `/`
      - 시작시간/ 단위로 입력하여 단계 값을 지정할 수 있다.
      - 예) 0 0/1 * * * ? -> 1분 마다

      `#`
      - 요일 필드에서 사용
      - 1-5 사이의 숫자가 와야한다.
      - 예) 3#1 -> 매월 첫째 주 수요일
      
      `L`
      - last를 의미
      - 요일 필드에서는 해당 월의 마지막 해당 요일을 표시
      - 월 필드에서는 해당 월의 마지막 날을 표시
      - 예) 5L -> 해당 월 마지막 금요일
      
      `W`
      - 가장 가까운 평일을 표시
      - 예) 1W -> 해당 월 1일과 가장 가까운 평일
   
   ![image](https://user-images.githubusercontent.com/79154652/148499035-10f02088-eb57-4a85-8b47-4679acb4a55f.png)

   
[참고]()
---
[https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#scheduling](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#scheduling)
