
# Optional
  
  #### 1. Null 의 늪
  
   - null 개념은 1965년 Tony Hoare라는 영국의 컴퓨터 과학자에 의해서 처음 고안되었다.
   - `존재하지 않는 값` 을 표현할 수 있는 가장 편리한 방법이 null 참조라고 생각했다고 한다.
   - 하지만 나중에 그는 그당시 자신의 생각이 "10억불 짜리 큰 실수" 였고, null 참조를 만든 것을 후회한다고 함.


  #### 2. NPE(NullPointerException)
  
   - null 참조로 인해 자바 개발자들이 가장 골치아프게 겪는 문제는 널 포인터 예외(NPE) 이다.
   - NPE는 코드베이스 곳곳에 깔려있는 지뢰같은 녀석이다. 컴파일 타임에서는 조용히 잠복했다가 런타임 때 터지기 때문이다.

           /* 주문을 한 회원이 살고 있는 도시를 반환한다 */
          public String getCityOfMemberFromOrder(Order order) {
          return order.getMember().getAddress().getCity();
          }
     
   위코드는
   
   `order` 파라미터에 null값이 넘어옴
   
   `order.getMemeber()` 의 결과가 null 임
   
   `order.getMember().getAddress()` 의 결과가 null 임
   
   `order.getMember().getAddress().getCity()` 의 결과가 null 임
   
   위 네 가지 경우 모두 NPE의 위험을 전파시키는 포인트들 이다. 호출부에서 null을 처리해주지 않으면 NPE를 발생시킬 수 있다.
   
   
   ## JAVA8 이전의 NPE 방어 패턴
   
   JAVA8 이전에는 아래 코드 처럼 NPE를 막기위해 null 체크를 항상 해주어 코드 스타일이 매우 난잡하다.
   
   중첩 null 체크하기
        
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
           
  모든 단계마다 null이 반환되지 않을지 의심하면서 null 체크를 합니다. 들여쓰기 때문에 코드를 읽기가 매우 어려우며 핵심 비즈니스 파악이 쉽지 않습니다.
