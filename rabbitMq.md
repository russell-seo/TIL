# Rabbit MQ 로 Queue 데이터 처리중 Error가 발생하면?


필자는 회사에서 MQ를 사용하여 동시성 문제를 처리하고 있는데, 처음 Rabbit MQ 도입을 하였다.

### As is
- Rabbit MQ로 메시지를 처리하다가 Error 가 발생하면 다음 메시지를 처리하기 위해 이 메시지를 건너뛰어야 하는 경우가 발생한다.
- 하지만 필자는 Consumer 에서 해당 데이터를 소비하면 끝날 줄 알았는데, 에러가 발생하는 메시지는 `다시 해당 Queue에 재진입하였다`
- 이로 인해 해당 메시지를 소비하는 것이 아닌 계속해서 에러를 뱉어내었고 Queue 에 들어오는 데이터가 계속해서 소비되지않고 쌓이게 되는 현상이 발생한다.


### To be

- 에러가 발생한 데이터는 requeue 하는 것이 아니라 예외를 잡아내고 건너뛰어야 한다.
- 필자는 에러가 발생한 메시지는 다시 유저에게 예외처리 메시지를 보낸다.




## 해결방안

1. Dead-Letter Queue 를 두고 처리할 수 있다.
2. 예외를 잡아서 처리할 수 있다.



필자는 굳이 또 하나의 Queue를 만들어서 처리하지않고 `try-catch`로 예외를 잡아서 간단하게 처리하게만 할 예정이다.

> 이때 yml 파일에 requeue = false로 설정해야 한다. requeue = true 로 설정해놓으면 예외나 dead letter Queue 로 가지 않고 원래 queue로 돌아간다



.yml 파일
~~~
spring:
  rabbitmq:
    host: 
    port:
    username: 
    password: 
    listener:
      simple:
        default-requeue-rejected: false
        retry:
          enable: false
~~~

- default-requeue-rejected : false 가 핵심이다.


참고

[참고](https://blog.leocat.kr/notes/2018/06/20/rabbitmq-dead-lettering-with-reject-or-nack)

