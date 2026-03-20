# 대규모 트래픽 상황에서, I/O 병목을 어떻게 해결해볼 수 있을까?

```./JVM/Sync_Async_Blocking_Non-Blocking_In_Java_Thread.md```에서 언급한 대로,  
DB에서 데이터를 조회하여 그 값을 사용해야하는 상황이 많이 발생하고, 이때 Thread를 점유한 상테에서 대기하게 된다. OS Thread는 이때 Ready 상태로 들어가게 되며 CPU 사용률이 떨어지게 될 수 있다.

죽, 정리하자면
> IO병목 > OS Thread(platform thread) 대기 > CPU 할거 없음 > CPU 사용률 떨어짐. 

 나는 IO병목을 "데이터를 보내고 받는 동안 Thread가 blocking 되는 상태"라고 정의해 보았다.

1. 그렇다면 무작정 Thread의 갯수를 늘리면 될까?   

- platform thread는 메모리를 많이 잡아먹는다. 
- cpu의 context switching 비용 

2. 서버를 수평 확장하거나 수직 확장해서 자원을 더 확보하면 될까?

- 서버를 확장하는 것은 비용과 직결된다. 

3. 자원 효율을 높인다.

- 경량 스레드 사용 > 메모리를 줄이고 CPU 사용률을 높일 수 있음. 
- 논블로킹 or 비동기 IO 사용. 





 
