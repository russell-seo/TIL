
# Bean
   
   ## Bean
    
   * Spring bean 이란?
       * 스프링 컨테이너에 의해서 자바 객체가 만들어 지게 되면 이 객체를 스프링 빈 이라고 한다.
       * 스프링 빈과 자바 일반 객체와는 차이점이 없다. 다만 스프링 컨테이너에서 만들어 질 뿐이다.
   * Reference
      * 스프링 Ioc컨테이너에 의해서 관리되고 애플리케이션의 핵심을 이루는 객체들을 스프링에서는 빈즈(beans)라고 부른다.
      * 빈은 Ioc컨테이너에 의해서 인스턴스화되어 조립되거나 관리되는 객체를 말한다.
      * 빈과 빈 사이의 의존성은 컨테이너가 사용하는 메타데이터 환경설정에 반영된다.
   
   
   Bean의 주요 속성
   
   * class(필수) : 정규화된 자바 클래스 이름
   * id : bean의 고유 식별자
   * scope : 객체의 범위
   * constructor-arg : 생성 시 생성자에 전달할 인수
   * property : 생성 시 bean setter 에 전달할 인수
   * init method와 destory method
   
  
  
## Bean 설정 방법

  * Bean 설정 방법은 총 3가지 이다.
    * XML을 이용한 설정
    * Component Scan을 이용한 설정
    * 자바 설정파일을 이용한 설정


   
