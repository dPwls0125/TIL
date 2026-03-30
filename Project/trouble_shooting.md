## 컨테이너가 kafak 호출할때 Kafka Connection Refused 해결
에러 상항 : docker container로 각 모듈을 띄운 상태에서, payment-server가 localhost:9092로 kafka연결을 계속해서 실패함.   

```에러 메시지: Connection to node -1 (localhost/127.0.0.1:9092) could not be established.```


**해결 내용**  

**AS-IS:**
kafka 컨테이너가 외부 접근용 리스너(PLAINTEXT://localhost:9092) 하나만 제공함.
각 Spring Boot 서버들은 Kafka 접속 주소가 아예 명시되지 않아 기본값(localhost:9092)로 시도함.  

**TO-BE:**
Kafka Listener 분리 (docker-compose.yml): 외부 Host용(localhost:9092)과 Docker 내부 통신용(kafka:29092) 리스너를 분리하여 KAFKA_ADVERTISED_LISTENERS에 등록.
Spring Boot 환경변수 주입: docker-compose.yml 내 각 애플리케이션(api-server, payment-server 등)의 environment에 - SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:29092를 추가해, 연결지점이 localhost가 아닌 kafka 컨테이너가 되도록 동적 주입.

**회고**
Docker-Compose로 여러 애플리케이션을 묶을 때는 각 컨테이너가 분리된 독립된 네트워크 안에 있다는 점을 잊지 말아야 한다.  
localhost는 내 PC가 아니라 컨테이너 자기 자신을 의미하므로, 컴포넌트 간 통신 시 반드시 컨테이너 이름(Service Name)을 주소로 사용해야 한다.  



