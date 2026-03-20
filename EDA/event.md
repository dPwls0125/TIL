---
**이벤트와 명령의 구분**
이벤트 기반 아키텍처를 이해하기 위해서는 이벤트와 명령의 차이를 이해하는 것이 중요하다. 
** 이벤트(Event)**
- "어떤 일이 일어났다"는 사실을 나타내는 정보입니다.  
- 예를 들어, "주문 완료", "사용자 로그인"과 같이 이미 발생한 사건을 알리는 역할을 합니다.  
- 이벤트는 불변(Immutable)하며, 한번 발생하면 변경되거나 삭제되지 않는 영구적인 기록이 됩니다.    
- 이벤트는 정보 자체에 초점을 맞추며, 특정 행동을 지시하기보다는 발생한 사실을 전달합니다.


**What is EDA** 

https://martinfowler.com/articles/201701-event-driven.html?utm_source=chatgpt.com

이해한 내용 간략하게 정리 : 이벤트 알림은 시스템이 도메인 변경을 다른 시스템에게 알리기 위해 사용된다. 그러나 source system은 이벤트가 처리되는 과정이나 그 응답에 관심을 갖지 않는다.  따라서 어떠한 논리적 흐름이 필요한 경우에 이벤트 소싱을 사용하는 경우, 버그 추적이 어려울 수 있다. 

**핵심적이라고 생각하는 문장 : There would be a marked separation between the logic flow that sends the event and any logic flow that responds to some reaction to that event.**

이벤트를 보내는 논리 흐름과 그 이벤트에 반응하는 모든 논리 흐름 사이에는 뚜렷한 분리가 있을 것이다. (낮은 결합) 

**이해가 더 필요한 지점 :** There's no need for event processing to be asynchronous, consider the case of updating a local git repository - that's entirely a synchronous operation, as is updating a centralized version-control system like subversion.

이벤트 소싱의 핵심 아이디어는 시스템 상태에 변화가 있을 때마다
그 상태 변화를 이벤트로 기록하고, 미래에 언제든지 이벤트를 재처리하여 시스템 상태를 자신 있게 재구성할 수 있다는 점입니다

**AI 답변 :** 

**“이벤트 소싱(event sourcing) = 비동기 처리”라고 착각하지 말라.
이벤트를 저장/적용하는 핵심 동작은 동기식으로도 충분히 가능하다.**

조금 풀어보면,

## 1) 사람들이 흔히 하는 오해

- 이벤트 소싱을 이야기하면 자동으로
    - “이벤트 발행 → 다른 서비스가 비동기로 처리”
    - “메시지 브로커/EDA”
        
        같은 그림을 떠올리기 쉬워요.
        

그런데 Fowler는 “그건 필수 조건이 아니다”라고 말하는 겁니다.

## 2) 이벤트 소싱의 핵심은 “비동기”가 아니라 “이력(event log)”

이벤트 소싱은 본질적으로:

- 상태를 바로 저장하는 대신
- **‘무슨 일이 일어났는지’ 이벤트를 순서대로 저장**하고
- 현재 상태는 이벤트를 재생(replay)해서 만든다

이게 핵심이에요.

이 과정이 **반드시 비동기일 이유는 없다**는 거죠

## 4) 결론

Fowler의 포인트는:

- 이벤트 소싱을 **EDA/비동기 메시징과 묶어서** 생각하지 말고
- 먼저 “동기식 커밋 + 이벤트 로그로 상태를 만든다”는 구조로 이해하라는 것.
