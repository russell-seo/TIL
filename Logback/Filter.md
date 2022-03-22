







# Logback 

  기존 log를 관리하기 위한 구현체로서 log4j가 사용되었다. 그러나 보다 안정성이 높고 편리하게 log를 관리하기 위해 Logging framework인 Slf4j(Simple Logging Facade for java)와 그 구현체로써 `Logback`이 고안되었다.
  
  Slf4j는 일명 Facade 패턴으로 이를 사용하면 구현체의 종류와 상관없이 일관된 로깅 코드를 작성할 수 있으며 구현체를 변경할 경우에도 최소한의 수정으로 교체가 가능하다. 이에 맞춘 Logback은 log4j의 후속 버전으로 만든 Logging 라이브러리이다.
  
  __Logback__ 구조
  
  Logback을 사용하기 위해서는 다음과 같은 tag들로 구성되어 있다.
    
    - Logger : 로그의 주체, 로그의 메시지 전달, 특정 패키지 안의 특정 레벨이상인 것에 대해 출력
    - Appender : 어디에 출력할지에 대해 기술 -> console / file / DB appender
    - Encoder : 어떻게 출력할지에 대해 기술

  ## AsyncAppender
  
  - Logback 에서 지원하는 Appender 중 하나이며 기존에 사용하고 있는 Appender를 reference 하여 사용할 수 있도록 제공하고 있다.
  - 왜? 사용하는 것 일까
      - 기존에 sync 방식의 RollingFileAppender 를 이용해도 서비스를 동작하는데는 아무 문제가 없다. 그렇지만 AsyncAppender 를 이용하게 되면 장점이 존재한다.
      - 기존 sync 방식보다 로그를 남기는데 있어서 성능상 빨라진다.
      - 단점도 존재한다.
          - queueSize를 너무 작게 하는 경우 WARN, ERROR를 제외하고 로그의 손실을 가져올 수 있다.
          - 버퍼를 이용하니 메모리의 사용량이 증가하고 CPU 사용량 또한 증가한다.
          - 중간에 서버가 shutdown되는 경우에 버퍼에 로그가 남아 있으면 버퍼의 로그를 다 쓰기 전에 종료되어 손실이 발생한다.

~~~java

<configuration debug="true">
  <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />

  <property name="LOG_PATH" value="/tmp"/>
  <property name="LOG_FILE" value="${LOG_PATH}/logback-test.log"/>

  <appender name="RollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_FILE}</file>
    <encoder>
      <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS,Asia/Tokyo} %-5level [%thread,%X{X-B3-TraceId:-},%X{X-B3-SpanId:-},%X{X-Span-Export:-}] %logger{36} [%file:%line] - %msg ##%n</Pattern>
    </encoder>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}</fileNamePattern>
      <maxHistory>5</maxHistory>
    </rollingPolicy>
  </appender>

 <!-- AsyncAppender 추가 -->
  <appender name="file-async" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="RollingFile" />
    <queueSize>1</queueSize>
    <discardingThreshold>0</discardingThreshold>
    <includeCallerData>false</includeCallerData>
    <neverBlock>false</neverBlock>
  </appender>

  <root level="TRACE">
    <!-- AsyncAppender로 변경 -->
    <appender-ref ref="file-async" />
  </root>

</configuration>

~~~

# Filter


  ## LevelFilter
  
  - 정확한 레벨 일치를 기반으로 이벤트를 필터링 한다. 이벤트 레벨이 구성된 레벨이 같으면 `onMatch` `onMistMatch` 특성의 구성에 따라 필터가
    이벤트를 승인하거나 거부 하게 할 수 있다.
    
    ~~~java
    <configuration>
      <appender name = "CONSOLE" class = "ch.qos.logback.core.ConsoleAppender">
        <filter class = "ch.qos.logback.classic.filter.LevelFilter">
          <level>INFO</level>
          <onMatch>ACCEPT</onMatch>
          <onMismatch>DENY</onMismatch>
        </filter>
        <encoder>
          <pattern>
            %-4relative [%thread] %-5level %logger{30} - %msg%n
          </patter>
        </encoder>
      </appender>
   
    <root level="DEBUG">
      <appender-ref ref = "CONSOLE" />
    </root>
    </configuration>

    
    ~~~
    
    `onMatch`, `onMismatch`를 통해 로그에 남길 것인지 선택 ACCEPT(승인), DENY(거절) 할 수 있다.
    
    
  ## ThresholdFilter
  
   - 지정된 입계 값 아래 이벤트를 필터링 한다. 임계값보다 낮은 레벨의 경우 이벤트는 거부된다.

 ~~~java
 <configuration> 
  <appender name="CONSOLE" 
    class="ch.qos.logback.core.ConsoleAppender"> 
 <!-- deny all events with a level below INFO, that is TRACE and DEBUG --> 
  <filter class="ch.qos.logback.classic.filter.ThresholdFilter"> 
    <level>INFO</level> 
  </filter> 
    <encoder> 
      <pattern> 
        %-4relative [%thread] %-5level %logger{30} - %msg%n 
      </pattern> 
    </encoder> 
  </appender> 
  <root level="DEBUG"> 
    <appender-ref ref="CONSOLE" /> 
  </root> 
 </configuration>

 ~~~
 
 Loglevel 이 info 이하인 것에 대해서는 해당 이벤트를 실행하지 않는다. 즉 INFO, ERROR만 이벤트를 실행한다.
 
 
 ## EvaluatorFilter
 
 EventEvaluator 를 캡슐화하는 일반 필터 입니다.
 
 ~~~java
 <configuration> 
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
    <filter class="ch.qos.logback.core.filter.EvaluatorFilter"> 
      <evaluator class="ch.qos.logback.classic.boolex.GEventEvaluator"> 
        <expression> 
          e.level.toInt() >= WARN.toInt() &amp;&amp; <!-- Stands for && in XML --> 
          !(e.mdc?.get("req.userAgent") =~ /Googlebot|msnbot|Yahoo/ ) 
        </expression> 
      </evaluator> 
      <OnMismatch>DENY</OnMismatch> 
      <OnMatch>NEUTRAL</OnMatch> 
    </filter> 
    <encoder> 
      <pattern> 
        %-4relative [%thread] %-5level %logger - %msg%n 
      </pattern> 
    </encoder> 
   </appender> 
   
   <root level="DEBUG"> 
      <appender-ref ref="STDOUT" /> 
   </root> 
 </configuration>

 ~~~
 
