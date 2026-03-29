- platform thread(carried thread) : OS thread와 1:1 매핑. 즉 os 스케줄링의 대상. 여러개의 virtual thread를 돌아가면서 실행시킬 수 있음.
- virtual thread : 1개의 carried thread에 할당, JVM에서 스케줄링.
    - blocking I/O시 carried thread에서 언마운트 되어 다른 virtual thread가 해당 자리를 차지함. 


 # https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html#GUID-BEC799E0-00E9-4386-B220-8839EA6B4F5C

Why Use Virtual Threads?
가상 스레드를 사용하는 이유는 무엇입니까?
Use virtual threads in high-throughput concurrent applications, especially those that consist of a great number of concurrent tasks that spend much of their time waiting. Server applications are examples of high-throughput applications because they typically handle many client requests that perform blocking I/O operations such as fetching resources.
처리량이 많은 동시 애플리케이션, 특히 **기다리는 데 많은 시간을 소비하는 다수의 동시 작업으로 구성된 애플리케이션에서 가상 스레드를 사용**하십시오. 서버 애플리케이션은 일반적으로 리소스 가져오기와 같은 I/O 작업 차단을 수행하는 많은 클라이언트 요청을 처리하므로 처리량이 높은 애플리케이션의 예입니다.

Virtual threads are not faster threads; they do not run code any faster than platform threads. They exist to provide scale (higher throughput), not speed (lower latency).
가상 스레드는 더 빠른 스레드가 아닙니다. 플랫폼 스레드보다 더 빠르게 코드를 실행하지 않습니다. **속도(낮은 대기 시간)가 아닌 규모(더 높은 처리량)를 제공하기 위해 존재**합니다.

