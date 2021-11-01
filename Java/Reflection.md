필자는 프로젝트 진행중 로젠 서버에서 데이터를 받아와 VO객체에 값을 할당 하여야 했다. 하지만 데이터 값이 30개 이상되어 vo객체.set을 이용해 매번 값을 매핑하는 것은 막노동이라 생각하였다.

그래서 Java Refletion을 사용하여 자동 Mapping 하는 모듈을 만들었다.

## Reflection

- 리플렉션이란 객체를 통해 클래스의 정보를 분석하는 것이다.
- 실행중인 자바프로그램 내부를 검사하고 내부의 속성을 수정할 수 있도록 한다.

## 메소드

- Class.forName("java.lang.String");
    - 자바 기본형에 대한 클래스 정보를 얻음.
- Class.newInstance()
    - 주어진 클래스의 인스턴스를 생성
- Class.getName()
    - 클래스의 이름을 반환
- Class.getMethods()
    - 클래스에 선언된 모든 public 메소드의 목록을 배열로 반환
- Method.invoke()
    - 해당 메소드를 호출
- Method.getParameterTypes()
    - 메소드의 매개변수 목록을 배열로 반환

```java
Class c = vo.getClass() // VO의 클래스를 얻어옴

Field[] sc = c.getDeclaredFields(); 
//모든 필드값 배열로 받아오기

Method[] method = c.getDeclaredMethods();
//모든 메소드 가져오기, Declared -> private 접근가능
//기본 getMethods는 public만 접근 가능

Method m = c.getMethod("메소드명", "파라미터타입")
//특정 메소드 가져오기

field.setAccessible(true)
//private 인 필드나 메소드를 변경할려면 꼭 호출해야함

m.invoke(VO객체, 파라미터)
//메소드를 실행시키는 함수 -> invoke 
```
![image](https://user-images.githubusercontent.com/79154652/139623064-7548a7c4-66a9-461e-895b-810dd17bbea8.png)
