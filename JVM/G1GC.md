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



