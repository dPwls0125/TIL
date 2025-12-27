# Garbage Collector란?

### GC를 간략하게 설명한다면? 
heap 영역에 동적으로 할당 된 객체들 중, 더이상 참조되지 않는(unreachable) 객체들을 제거하는 것. GC 주기가 돌아갈떄, **GC Root로 부터 Reachable한 객체들을 하나의 다른 안전한 공간에 옮기고**, 기존의 공간을 Freelist에 넣어버린다. 

=> 핵심은 GC는 쓰지 않는, 참조하지 않는 즉 unreachable 객체를 대상으로 움직이지 않는다는 것. 목적은 더이상 참조되지 않는 객체를 정리하는 것이지만, 실제 GC 시스템은 lived Object를 중심으로 움직인다. 즉, GC의 본질은 살아있는 객체를 안전한 공간으로 옮기고, 원래 있던 공간을 통째로 반환하는 것이다.


### Mark And Sweep _ 기본적인 GC 알고리즘

- Mark - 객체 그래프의 GC Root에서 시작하여, GC가 Reachable 객체를 마킹함. 
- Sweep - Mark되지 않은 객체들을 Heap에서 제거한다. (marking한 객체들을 비어있는 영역으로 이동한 후에, sweep!)
- Compact - Sweep 이후에 분산된 객체들을 힙의 시작 주소로 모아 메모리에 할당된 부분과 그렇지 않는 부분으로 압축한다. 

### GC Root란?

GC 루트는 아무도 참하지 않아도 반드시 살아있어야 하는 객체로, 마킹의 시작점이 된다. 
ex) 스택 프레임의 지역변수, 메서드 영역의 static 변수 등. 

**Concurrent Marking Cycle (동시 마킹 사이클)**

Old Region에 Garbage가 쌓이기 시작하면 G1GC는 Old 영역 전체를 대상으로 한 마킹 사이클을 시작합니다.

이 과정은 Young GC와 병행하여 수행됩니다.

5.1 Initial Mark

STW

Old Region에 직접 연결된 Root 객체를 마킹

Young GC와 함께 수행되므로 추가 비용이 적음

5.2 Concurrent Mark

STW 아님

애플리케이션 스레드와 동시에 실행

Old Region 전체를 순회하며 Live Object 마킹

이 단계에서 각 Region의 Live Ratio / Garbage Ratio 계산

이 단계가 G1GC의 핵심입니다.
“어느 Region에 Garbage가 얼마나 있는지”가 이때 결정됩니다.

5.3 Remark

STW

Concurrent Mark 중 변경된 객체 참조 정리

SATB(Snapshot-At-The-Beginning) 방식 사용

Remark는 짧지만 중요합니다. 이 단계가 정확하지 않으면 잘못된 객체가 수집 대상이 될 수 있습니다.

5.4 Cleanup

STW + Concurrent

Live Object가 거의 없는 Region을 즉시 Free 처리

Old Region 중 일부는 수집 후보로 분류됨

이 시점에서 JVM은 다음을 이미 알고 있습니다.

어떤 Old Region에 Garbage가 많은지

어떤 Region을 먼저 수집하면 Pause Time 대비 효율이 좋은지


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

**=> 하지만 다음 마킹 사이클에서 정상 회수**


**TAMS (Top-At-Mark-Start)**
2.1 TAMS란 무엇인가요? 

TAMS는 "마킹이 시작될 당시, 각 Region에서 객체가 할당된 최상단 위치"를 기록한 포인터! 

- Region 마다 존재 
- 마킹 시작 시점에 고정됨 
- Pre TAMS, Next TAMS 두가지 존재 

2.2 왜 TAMS가 필요한가요? 

문제는:

마킹 도중에 생성된 객체를
마킹 대상으로 포함할지 여부
SATB의 관점에서는 답이 명확합니다.

마킹 시작 이후에 생성된 객체는
스냅샷에 포함되지 않으므로
“자동으로 살아 있는 객체”로 취급

=> 이 구분을 빠르고 정확하게 하기 위해 TAMS가 필요합니다.

2.3 TAMS 핵심 규칙 

Region 내의 객체를 다음과 같이 판단한다. 

- 객체 주소 < TAMS => 마킹 대상 (스냅샷에 포함)  
- 객체 주소 >= TAMS => 마킹 제외 (무조건 live)


| 단계              | SATB         | TAMS       |
| --------------- | ------------ | ---------- |
| Initial Mark    | 활성화          | TAMS 설정    |
| Concurrent Mark | 참조 삭제 감시     | TAMS 기준 판별 |
| Remark          | SATB 큐 처리 완료 | 유지         |
| Cleanup         | 마킹 종료        | TAMS 전환    |



**한문장 정리** 

- SATB : "마킹 시작 시 살아 있던 객체는 끝까지 살아 있다고 보자"
- TAMS : "마킹 시작 이후에 생성된 객체는 마킹하지 말고 그냥 살려두자"

이 두개가 결합되어 G1의 짧은 STW + 안전한 Concurrent Marking이 가능해진다. 


**GC가 반드시 지켜야 하는 불변식**

마킹 시작 시점에 살아 있던 객체는
이번 GC 사이클에서는 절대 회수되면 안 된다.


GC는 힙 전체를 단일 시점으로 보지 못함

A가:

- 처음부터 죽어 있었는지
- 아니면 중간에 죽었는지
- 구분할 방법이 없음

따라서 STW or Concurrent Marking의 경우에는 STAB가 필요함!!