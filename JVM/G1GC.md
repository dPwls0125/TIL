# Garbage Collector란?

### GC를 간략하게 설명한다면? 
heap 영역에 동적으로 할당 된 객체들 중, 더이상 참조되지 않는(unreachable) 객체들을 제거하는 것. GC 주기가 돌아갈떄,   **GC Root로 부터 Reachable한 객체들을 하나의 다른 안전한 공간에 옮기고**, 기존의 공간을 Freelist에 넣어버린다. 

=> 핵심은 GC는 쓰지 않는, 참조하지 않는 즉 unreachable 객체를 대상으로 움직이지 않는다는 것. 목적은 더이상 참조되지 않는 객체를 정리하는 것이지만, 실제 GC 시스템은 lived Object를 중심으로 움직인다. 즉, GC의 본질은 살아있는 객체를 안전한 공간으로 옮기고, 원래 있던 공간을 통째로 반환하는 것이다.


### Mark And Sweep _ 기본적인 GC 알고리즘

- Mark - 객체 그래프의 GC Root에서 시작하여, GC가 Reachable 객체를 마킹함. 
- Sweep - Mark되지 않은 객체들을 Heap에서 제거한다. (marking한 객체들을 비어있는 영역으로 이동한 후에, sweep!)
- Compact - Sweep 이후에 분산된 객체들을 힙의 시작 주소로 모아 메모리에 할당된 부분과 그렇지 않는 부분으로 압축한다. 

### GC Root란?

GC 루트는 아무도 참하지 않아도 반드시 살아있어야 하는 객체로, 마킹의 시작점이 된다. 
ex) 스택 프레임의 지역변수, 메서드 영역의 static 변수 등. 


**SATB (Snapshot-At-The-Beginning)**

1.1 SATB란?
"마킹 시작 시점의 힙 상태를 기준으로 살아 있는 객체 집합을 정의하는" 마킹 알고리즘. 
- Concurrent Marking이 시작되는 순간을 기준으로 
- 그 시점에 Reachable 했던 객체는 마킹이 끝날 때 까지 무조건 살아있다고 간주합니다. 

즉, 마킹 도중에 죽은 객체라도 마킹 시작 시 살아 있었으면 live로 처리합니다. 

 1.2 왜 SATB가 필요한가요? 

 문제 상황 (Concurrent Marking의 본직적인 문제)
 G1의 마킹은 애플리케이션과 동시에 진행된다. 

 이때 문제가 발생. 

1. 마킹 스레드가 아직 객체 A를 방문하지 않음. 
2. 애플리케이션 스레드가 애플리케이션 스레드가 A로 가는 마지막 참조를 제거 
3. 마킹 스레드는 A를 "죽었다"고 판단. 
4. But,마킹 시작 시점에는 A가 살아 있었음. 

-> 이 경우 객체를 잘못 회수(메모리 오류) 할 수 있습니다. 

1.3 SATB Write Barrier(Pre-Write Barrier)

SATB를 구현하기 위해 G1은 Pre-Write Barrier를 사용합니다.

동작 방식

- 어떤 참조 필드가 변경되기 직전
- 기존에 참조하던 객체(old value)를
- SATB 버퍼에 기록

```
obj.field = newValue
↑
이전에 가리키던 객체를 SATB 큐에 기록
```

이렇게 하면:

“마킹 시작 시 살아 있었던 객체”를 놓치지 않게 됩니다.

📌 중요.  
SATB는 참조 삭제에만 반응
참조 추가에는 관심 없음


1.4 SATB의 결과적 특징 

장점 
- Remark 단계 (STW)가 매우 짧아짐
- Concurrent Marking 안정성 확부 
- Low-Latency GC에 유리 

단점 
- 보수적 마킹
- 이미 죽은 객체도 다음 사이클까지 살아 있을 수 있음. 
- Old Generation 회수 가 한 사이클 늦어질 수 있음. 

