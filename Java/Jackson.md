
 ## Jackson 라이브러리를 이용하여 JSON을 Objcet로 변환
 
  1. JSON 문자열 -> Java Object
    ObjectMapper.readValue(JSON, 매핑 객체.class) 메소드 사용
    
    아래는 Object로 매핑할려는 클래스를 만들었다.
    
   ![image](https://user-images.githubusercontent.com/79154652/140070569-26e7e324-5214-4d5a-9f2a-aa6edbdc5af2.png)

JSON 문자열을 KaKaoDto 객체로 변환 할 것이다.

아래 코드는 카카오 서버에서 token값을 JSON 형식으로 받아와 Object로 변환한 코드이다.
  ![image](https://user-images.githubusercontent.com/79154652/140070924-f7e28932-27e7-4431-be61-18da3b452c05.png)
  
  확실히 JSON으로 데이터를 받아 편리하게 원하는 타입으로 매핑할 수 있어서 유용할 것 같다.
