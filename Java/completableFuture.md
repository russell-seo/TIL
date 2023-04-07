# 비동기 프로그래밍 과 CompletableFuture


동기,비동기 프로그래밍을 설명해야한다면 항상 같이 따라오는게 논블로킹과 블로킹일 것이라고 생각한다. 이에대해 내가 알고있는 부분에 대해 기술하고 

JAVA 1.8 부터 지원하는 CompletableFuture을 사용해 `비동기, 논블로킹`에 대해 기록할 예정이다.


## 블로킹, 논 블로킹

`블로킹` 이란 누군가의 행위로 인해 다른 누군가가 제한되거나 대기해야 하는 상태

`논 블로킹` 이란 누군가의 행위로 인해 다른 누군가가 제한되지 않거나 대기하지 않는 상태

> 동기 - 하나의 함수가 정해진 코드를 실행하는 과정에서, 다른 함수를 호출함으로써 제한되거나 대기하는 상태
> 비동기 - 하나의 함수가 정해진 코드를 실행하는 과정에서, 다른 함수를 호출하고도 제한되거나 대기하지 않는 상태

__이는 제어권의 관점에서 이해하면 이해하기 쉽다__


![image](https://user-images.githubusercontent.com/79154652/229471020-30aa0077-e12e-404f-8b6c-c4d1c13aa3fd.png)


- __Blocking__
  - 호출된 함수가 요청한 작업을 모두 완료 할 때 까지 호출한 함수에게 제어권을 넘겨주지 않고 대기하게 한다.
  - 즉 호출한 함수는 호출된 함수가 작업을 완료할 때 까지 다음 작업을 하지못하고 기다리게 된다.

- __Non-Blocking__
  - 호출한 함수가 요청한 작업을 시작하기 전에 바로 리턴해서 제어권을 호출한 함수에게 넘겨주고 다음로직을 실행한다.
  - 즉 호출한 함수는 호출된 함수가 무엇을 하던 기다리지 않게 된다.


- __Synchronous(동기)__
  - 하나의 작업을 수행하는데 이 작업이 모두 끝나야 다음 작업을 수행하는 것이다. 즉 A 작업이 모두 진행될 때 까지 B 작업은 대기한다.

- __Asynchronous(비동기)__
  - 비동기 방식은 반대로 하나의 작업을 수행하는데 응답 상태와 상관없이 다음 동작을 수행한다. 즉 A 작업이 시작하면 동시에 B 작업이 실행된다.
  - A 작업은 결과값이 나오는 대로 출력된다.
  - 대표적으로 callback 함수를 넘겨 호출된 함수가 작업완료 후 callback을 실행시키는 것

![image](https://user-images.githubusercontent.com/79154652/229472790-4cbb13a3-a2c1-4b25-a255-a370b34749f9.png)


위에서 언급한 `Blocking`, `Non-Blocking`, `Synchronous`, `Asynchronous`를 조합하면 총 4개의 조합이 나올수 있다.

기본적으로 흔히 우리가 흔히 볼 수 있는 `Blocking Synchronous` 조합은 서버에서 기본적으로 가장 많이 쓰는 조합이다.

이번글에서는 비동기 프로그래밍에 관한 내용 이므로 

`Async Non-Blocking`에 대해서만 간단하게 설명할 예정이다.

### Async Non-Blocking

`Async` 는 호출하는 함수가 호출되는 함수의 완료 여부를 상관하지 않고 다음 로직을 수행, `Non-Blocking`은 호출하는 함수가 다른 함수를 호출해도 제어권을 바로 리턴받아서 가지고 있다.

이를 조합하면 Non-Blocking 방식으로 함수를 호출하고 바로 반환 받아 다른 로직을 수행하며 함수 호출에 대한 완료 여부는 신경쓰지 않는다.

호출한 함수에 대한 완료 혹은 리턴값은 `Callback`으로 받은 것이 대표적이다.

![image](https://github.com/binghe819/TIL/blob/master/OS/blocking_nonblocking_%26_synchronous_asynchronous/image/asynchronous_non_blociking.png)


Asyn Non-Blocking 에 대한 코드를 CompletableFuture을 사용해서 봐보자.

~~~kotlin

@GetMapping("/{id}")
    fun fluxTest(@PathVariable id : Long)   {
        val executors = Executors.newFixedThreadPool(1)

        val completableFuture = 
        CompletableFuture.supplyAsync({
                        personRepository.findById(id).get()
                        }, executors)
                         .thenApply {
                         personService.getById(it.id!!)
                         }
                         .exceptionally {
                        println("it.stackTrace.toString() = ${it.stackTrace.toString()}").toString()
                        }


        println("제어권 반환 다음로직 수행")
    }

~~~

~~~Kotlin
@Service
class PersonService(
    private val personRepository: PersonRepository
) {

    fun getById(id : Long): String {
        println("callback start")
        Thread.sleep(10000)
        println("callback end ")

        return "Future"
    }
}
~~~

![image](https://user-images.githubusercontent.com/79154652/230521475-134f8926-3730-467d-8050-e3d00ad00c45.png)


### supplyAsync()
- CompletableFuture 의 `supplyAsync` 는 대표적인 비동기 처리를 요청하는 메소드 이다.
- `supplyAsync`는 비동기적으로 동작하길 원하는 작업중 리턴값이 있는 경우에 사용된다
- 

> `Future`와 동일하게 `get()` 호출시 Blocking 이 된다. 그리고 `get()`을 호출하지 않아도 `supplyAsync()`로 넘겨준 `Task가 비동기로 실행된다`


### thenApply()
