**Controller에서 CompletableFuture를 반환하면 무엇이 이득인가**

Controller에서 CompletableFuture를 반환하는 목적은 클라이언트에게 “나중에 응답을 주기 위해서”가 아니라, 서버의 thread 자원을 효율적으로 사용해 동시 처리량을 높이기 위함이다.

HTTP는 기본적으로 요청-응답 1:1 모델이다. 클라이언트는 요청을 보내고, 서버는 같은 연결 안에서 응답을 반환해야 한다. 따라서 Controller에서 CompletableFuture를 반환하더라도 클라이언트는 결국 해당 Future가 완료될 때까지 기다리게 된다. 즉, API가 진짜 비동기 모델로 바뀌는 것은 아니다.

동기 Controller의 경우, 요청을 처리하는 동안 Tomcat worker thread가 계속 점유된다.

```java
@GetMapping
public Response get() {
callExternalApi(); // 2초 소요
return response;
}
```

위 코드에서는 외부 API 호출이 2초 걸리면, Tomcat thread는 그 2초 동안 아무 일도 하지 못하고 점유된 상태가 된다. thread pool이 200개라면 동시에 최대 200개의 요청만 처리할 수 있고, 그 이상은 대기하게 된다.

반면, CompletableFuture를 반환하면 다음과 같이 동작한다.

```java
@GetMapping
public CompletableFuture<Response> get() {
return CompletableFuture.supplyAsync(() -> callExternalApi());
}
```

요청을 받은 Tomcat thread는 곧바로 반환되고, 실제 작업은 별도의 thread에서 수행된다. 작업이 완료되면 그때 응답이 작성된다. 이 방식의 핵심은 “기다리는 동안 Tomcat thread를 점유하지 않는다”는 점이다. 따라서 IO 대기 시간이 긴 작업에서는 서버의 동시 처리량이 크게 증가한다.

이 구조가 특히 효과적인 경우는 IO-bound 작업일 때이다. 예를 들어 외부 API 호출, 네트워크 통신, 느린 DB 조회 등 “대기 시간이 긴 작업”에서는 thread 낭비를 줄여 확장성을 개선할 수 있다.

반대로 CPU-bound 작업이나 트랜잭션 중심의 핵심 도메인 로직에서는 큰 이득이 없다. 예를 들어 배차 생성과 같은 트랜잭션 로직은 즉시 성공/실패가 확정되어야 하고, 상태 일관성이 중요하다. 이런 로직을 비동기로 감싸도 응답이 빨라지지 않으며, 오히려 트랜잭션 경계와 예외 처리가 복잡해질 수 있다.

정리하면, Controller에서 CompletableFuture를 반환하는 것은 “클라이언트 비동기 처리”를 위한 기술이 아니라, “서버 내부 자원 관리 최적화”를 위한 기술이다. 응답 속도를 줄이는 것이 목적이 아니라, 같은 자원으로 더 많은 동시 요청을 처리하기 위한 구조라는 점이 핵심이다.
