# Sync, Async, Blocking, Non-Blocking in Java Thread
Sync와 Async 그리고 Blocking vs Non-blocking은 중심이 되는 이해의 축(?)이 다릅니다. 

먼저 **Sync vs Async**를 구분하기 위해서는 “호출한 메서드의 결과를 누가 처리하는가?”를  판단하는 것이 중요합니다.

- Sync는 특정 작업을 요청한 스레드가 결과를 직접적으로 처리합니다.
- Async는 작업을 요청하고, 해당 작업의 결과에 대한 처리를 다른 스레드 / 콜백 / 이벤트 등으로 위임합니다.

예를 들어 아래 코드가 실행될 때, CompletableFuture은 다른 워커쓰레드에서 작업이 수행될 수 있도록 해줍니다. 처음에는 thenApply, thenAccept와 같은 callback메서드가 람다를 다른 쓰레드에서 비동기적으로 처리하고 돌아와서 메인쓰레드에서 실행되는 것이기 때문에 동기 아닌가? 라고 착각했지만 thenApply와 ThenAccept 또한 호출 쓰레드가 아닌 백그라운드 쓰레드에서 처리됨을 알 수 있었습니다.  따라서 아래의 코드는 비동기적으로 처리된다고 말할 수 있을 것 같습니다.

```
CompletableFuture cf =
    CompletableFuture
        .supplyAsync(() -> 10)
        .thenApply(...)
        .thenAccept(...);
```

반면 아래의 코드는 단지 부분적으로 비동기를 사용했지만 비동기적으로 수행한 람다 함수의 결과를 future.get()을 통해 결과가 들어올 때 까지 기다리고, 그 결과를 받아 호출 쓰레드에서 사용하고 있기 떄문에 blocking + sync라고 볼 수 있을 것 같습니다.
 ```

CompletableFuture<Integer> future =
    CompletableFuture.supplyAsync(() -> 10);

Integer result = future.get(); 
System.out.println(result * 2);
```

Non-blocking , blocking을 구분하기 위해선, **쓰레드가 멈추는가?**를 생각해보아야합니다.
- blocking의 경우, 호출 쓰레드가 특정 작업을 호출한후 waiting 상태가 됩니다. 즉, Thread를 점유한 상태에서 ready/running을 하지 않기 떄문에 즉 Thread 자원을 점유한 상태에서 멈춰있기 떄문에 서버의 측면에서는 처리율이 낮아질 수 밖에 없습니다.
- 반면 non-blocking의 경우, 호출 쓰레드가 멈추지 않고 남은 작업을 이어서 진행하며 계속해서 Runnable 상태가 됩니다. 따라서 호출한 작업이 끝나기를 기다리지 않기 때문에 빠른 응답(?)을 줄 수 있습니다.

다른 영상들에서는 쓰레드가 멈추는가에 대한 표현보다는 “제어권”을 기준으로 설명하는 것 같기도 합니다. 



# *Blocking은 무조건 서비입장에서 비효율적인가? 

요청수 > 가용 Thread인 상황에서는 처리량을 높이기 위해 Non-blocking이 유리하다. 
반면에 Thread 하나의 측면에서 본다면, Blocking이 된순간 Thread는 cpu 자원을 할당받지 않는다. 즉, 다른 Thread에 cpu를 점유할 수 있는 기회가  
더 많아진다는 것이다. 따라서, 요청수가 많지 않은 상황에서는 Blocking이 오히려 더 유리할 수 있다. (이 경우 webFlux를 써서 non-blocking으로 구현한다고 해도 어차피 병목이 되기 때문에 의미가 없음)


그러한 측면에서 blocking은 해야할 일이 있을 때(ex DB에 네트워크로 쿼리한 후 해당 데이터를 이용해야할때)리소스를 사용하게 되기 떄문에 불필요한 리소스 점유를 줄일 수 있다.






