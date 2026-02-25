### 2026.02.17
DB에서 네트워킹을 많이 타는 서버 로직과 DB에서 쿼리를 최적화하여 서버에의 데이터 연산을 최소화 할때의 차이를 비교하고자 하였다.
```"후자의 경우가 평균 처리 시간이 짧을 것이다." 라고 가설```을 세운것이다.
그런데 성능 테스트 결과 반대의 결과가 나왔다.  

이유는 바보 같게도 로컬에 띄운 DB에 연결하고 있었기 때문이다. 그러니 당연히 네트워크를 안탄다.  
테스트를 위해 AWS EC2에 Docker로 Mysql를 띄우고 로컬에서 EC2 내부의 컨테이너에 연결하여 테스트를 재진행했고, 초기 가설대로 결과가 나왔다. (10ms의 차이가 발생했다.)

**localhost에서 EC2내부 컨테이너의 특정 포트에 연결한 과정 정리**

1. 최소 사양의 인스턴스를 개설하고, 보안 그룹에 모든 포트 허용(테스트만 진행하고 마칠 거라서 IP를 특정하지는 않았다.)
2. docker로 컨테이너의 3306 포트로 Mysql 띄움
3. ```
   [로컬 PC]
    Spring Boot
    │
    │ JDBC 연결
    ▼
    localhost:3307
    │
    │ SSH 터널링
    ▼
    [EC2 서버]
    localhost:3306
    │
    ▼
    Docker MySQL (3306)
    ```
4. ssh 터널링을 경유하는 해당 방식은 터미널에서 터널 연결을 계속해서 유지해야했다.  
따라서 백그라운드에서 돌아갈 수 있도록 설정해주었다.
```
   Host my-ec2
    HostName 43.200.4.238
    User ubuntu
    IdentityFile ~/delivery.pem
    LocalForward 3307 localhost:3306
```

**LocalForward의 의미는 로컬 3307 포트로 연결할 시, Ec2의 3306 포트로 연결된다는 의미이다.** 


### 2026.02.17
**SSH 터널링 vs SSH 접속은 서로 다른 개념**

**문제 상황**
DB 부하 테스트를 진행하던 중, EC2 CPU 과부하로 테스트 실행에 실패했고 재부트를 진행하여 터널링을 진행했다.
터널링은 성공했음에도 테스트시 EC2연결에 실패하는 문제가 있었다. 

**핵심 정리**

- `ssh -N -f` 로 실행한 **SSH 터널링**은 "원격 서버 로그인"이 아니라  
  **로컬 ↔ 원격 포트 연결만 만든 상태**이다.

- 따라서 터널링이 떠 있다고 해서 SSH 접속이 완료된 것이 아니다.
---

### 왜 헷갈렸나

- 터널 프로세스는 이미 백그라운드에서 실행 중이었고
- EC2는 부팅 중이라 SSH daemon 이 아직 안 떠있는 상태였다.
- 그래서 터널은 살아있는데 SSH 접속은 계속 실패했다.
---

### 실제 원인

EC2 부팅 완료 전에 접속 시도해서 발생한 문제였다.  
(약 1~2분 기다리면 정상 접속 가능)

---

> SSH 터널은 네트워크 연결일 뿐, 서버 로그인 상태를 의미하지 않는다.


### 2026.02.22

**SSE**

서버에서 배차 완료 처리를 하고, 실시간으로 서버에서 클라이언트에게 RIDER가 배차 완료되었음을 알리고자한다.   
그래서 HTTP통신으로는 해당 기능을 구현할 수 없었다. 
HTTP는 기본적으로 요청-응답 후 연결이 종료되는 단발성 통신이고, 클라이언트가 요청해야만 서버가 응답할수 있어, 서버가 먼저 데이터를 보낼 수 없다.  
따라서 HTTP를 통해 실시간 알림을 구현하려며느 Polling이라는 방식을 통해 지속적으로 요청을 보내는 수 밖에 없다.  

SSE는 HTTP 기반이지만, **연결을 유지하는 스트리밍 방식**이다.  
한 번 연결되면 서버가 이벤트 발생시 실시간으로 데이터를 push할 수 있다. 따라서 polling 없이 이벤트를 서버 -> 클라이언트에게 발행할수 있는 것이다.  
정리하면, **HTTP는 “요청 중심 통신”, SSE는 “이벤트 중심 스트리밍 통신”**이다.  


**Polling 대신 SSE를 선택한 이유**  

Polling은 클라이언트가 주기적으로 서버에 요청해야 하므로 불필요한 요청이 많이 발생한다.  
이로 인해 서버 부하와 네트워크 트래픽이 크게 증가한다. 동일한 데이터를 주기적으로 반복해서 보내기 때문.   
또한 polling 주기만큼 실시간성이 떨어지고 지연이 발생한다.  
SSE는 한 번 연결하면 서버가 이벤트 발생 시 즉시 데이터를 push할 수 있다.  
따라서 불필요한 반복 요청 없이 효율적인 실시간 알림 구현이 가능하다.  
특히 알림, 상태 변경, 로그 스트리밍처럼 서버 → 클라이언트 단방향 이벤트 전달에 적합하다.

### 2026.02.25
**TCP 연결이 해재되었을때 SSE는 어떻게 재연결하는가**  

라이더는 이동하면서 IP가 변경될 수 있다.
TCP는 IP + Port 기반 연결이므로, 연결 중인 IP가 변경되면 기존 TCP 세션은 유지될 수 없다.

실제로 네트워크를 변경해보면 기존 SSE 연결은 즉시 끊기며, 서버 측에서는 IOException이 발생하고 onCompletion 콜백이 실행된다.

클라이언트 측에서는 new EventSource()(브라우저 엔진 내부 코드)로 생성된 객체가 내부적으로 자동 재연결을 수행한다.
MDN 문서 및 Chromium 구현 코드를 확인해보면, 연결이 끊기면 일정 시간(retry, 기본 약 3초) 후 동일한 URL로 재요청을 보내도록 구현되어 있다.

따라서 Delivery 서버는 IP 변경을 감지하거나 emitter의 IP를 업데이트할 필요가 없으며,
기존 emitter를 정리하고 재요청 시 새로운 emitter를 생성하는 구조로 충분하다.

1. 네트워크 종료 시 동작

Chromium Blink 엔진의 EventSource 구현을 보면, 네트워크 요청이 종료되면 NetworkRequestEnded()가 호출된다.
```
void EventSource::NetworkRequestEnded() {
loader_ = nullptr;

if (state_ != kClosed)
ScheduleReconnect();
}
```
TCP 연결 종료, 서버 연결 종료, 네트워크 에러 등은 모두 동일하게 "네트워크 요청 종료"로 처리된다.  

2. 재연결은 타이머 기반으로 수행됨
재연결 로직은 ScheduleReconnect()에서 구현되어 있다.
```
void EventSource::ScheduleReconnect() {
state_ = kConnecting;
connect_timer_.StartOneShot(
base::Milliseconds(reconnect_delay_), FROM_HERE);
DispatchEvent(*Event::Create(event_type_names::kError));
}
```
동작 과정은 다음과 같다.
상태를 CONNECTING으로 변경, 재연결 타이머 설정 (기본 3000ms), error 이벤트 발생

3. 실제 재연결 수행 시점
타이머가 만료되면 Connect()가 호출된다.
```
void EventSource::ConnectTimerFired(TimerBase*) {
Connect();
}
```

Connect()에서는 새로운 HTTP 요청을 생성한다.
```
request.SetHttpMethod("GET");
request.SetHttpHeaderField("Accept", "text/event-stream");
loader_->Start(request);
```

즉, 기존 연결을 복구하는 것이 아니라 동일 URL로 새로운 HTTP 요청을 보내는 방식이다.

4. 재연결 시 Last-Event-ID 자동 포함
재연결 요청에는 마지막 이벤트 ID가 자동으로 포함된다.
```
if (parser_ && !parser_->LastEventId().empty()) {
request.SetHttpHeaderField("Last-Event-ID", ...);
}
```
이를 통해 서버는 누락된 이벤트를 재전송할 수 있다.

5. IP 변경 시 동작 정리

IP 변경은 브라우저 입장에서 별도의 특별한 처리가 아니라 단순히 네트워크 연결 종료로 처리된다. 따라서 다음과 같은 흐름으로 동작한다.

1. TCP 연결 종료 감지
2. NetworkRequestEnded() 호출
3. ScheduleReconnect() 실행
4. 타이머 만료 후 Connect() 호출
5. 동일 URL로 새로운 HTTP 요청 전송
