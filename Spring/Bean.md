
# Bean
   
   ## Spring Bean
    
   * Spring bean 이란?
       * 스프링 컨테이너에 의해서 자바 객체가 만들어 지게 되면 이 객체를 스프링 빈 이라고 한다.
       * 스프링 빈과 자바 일반 객체와는 차이점이 없다. 다만 스프링 컨테이너에서 만들어 질 뿐이다.
       * Beans 는 우리가 컨테이너에 공급하는 설정 메타 데이터(XML 파일)에 의해 생성된다.
   * Reference
      * 스프링 Ioc컨테이너에 의해서 관리되고 애플리케이션의 핵심을 이루는 객체들을 스프링에서는 빈즈(beans)라고 부른다.
      * 빈은 Ioc컨테이너에 의해서 인스턴스화되어 조립되거나 관리되는 객체를 말한다.
      * 빈과 빈 사이의 의존성은 컨테이너가 사용하는 메타데이터 환경설정에 반영된다.
   
   ![이미지](https://gmlwjd9405.github.io/images/spring-framework/spring-bean.png)
   
   
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


### XML Based Configuration
~~~java
<!-- A simple bean definition -->
<bean id="..." class="..."></bean>

<!-- A bean definition with scope-->
<bean id="..." class="..." scope="singleton"></bean>

<!-- A bean definition with property -->
<bean id="..." class="..."><property name="message" value="Hello World!"/></bean>

<!-- A bean definition with initialization method -->
<bean id="..." class="..." init-method="..."></bean>
~~~

### Component Scan Configuration
- 스프링은 리플렉션과 어노테이션을 사용해서 Component Scan 을 지원한다.
~~~java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.springtest.demo"/>

</beans>
~~~

 - XML 파일을 통해서 Component Scan을 활성화 하고 @Autowired, @Service, @Repository 를 사용해서 빈을 설정 가능하다.
 - XML 설정에서 `context : component-scan`, 자바 설정에서 `@ComponentScan`
 - 스프링은 특정 패키지의 이하의 모든 클래스 중에 @Component 어노테이션이 붙은 클래스를 빈으로 자동 등록해준다.

### Spring Bean Scope

- 스프링은 기본적으로 모든 bean을 singleton 으로 생성하여 관리한다.
   
   - 구체적으로는 애플리케이션 구동 시 JVM 안에서 스프링이 bean 마다 하나의 객체를 생성하는 것을 의미한다.
   - 그래서 우리는 스프링을 통해서 bean을 제공받으면 언제나 주입받은 bean은 동일한 객체라는 가정하에서 개발 한다.
- request, session, global session의 SCOPE는 일반 Spring 애플리케이션이 아닌 Spring MVC Web Application에서만 사용
