
# Logback Filter

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
 
