Discovery-Server
1.	Service Registry : 마이크로서비스 아키텍처에서 각 개별 서비스 (AccountService, CartService, CatalogSerice, OrderService, GatewayService)가 자신의 위치 (IP 주소, 포트 번호)를 등록하고 관리하는 중앙 저장소 역할을 합니다. 
2.	서비스 발견 (Service Discovery) : 서비스 간 통신이 필요할 때, 상대방 서비스의 물리적인 주소를 직접 알 필요 없이 서비스 이름(ID)만으로 대상을 찾을 수 있게 해줍니다. 
3.	동적 라우팅 및 부하 분산 (Load Balancing) : GatewayService의 설정을 보면 인스턴스 정보를 제공하면, 게이트웨이가 이를 바탕으로 가용한 인스턴스에 요청을 분산(Client-side Load Balancing ) 처리합니다. 
API Gateway (Spring Cloud Gateway)
1.	Client의 요청을 받아 적절한 Server(Server API)로 Routing 하는 개념이다. 
wifi 공유기가 Network Gateway 역할을 하는것 처럼, API Gateway는 API Reqeust의 Gateway 역할만 수행한다. 
2.	SGC는 WebFlux(Netty) 비동기 방식으로 동작한다.
3.	현재 각 서버 모듈로 분리되어있는 서비스 모듈의 진입점을 하나로 모아, EndPoint를 제공한다.
4.	Eureka 서버에서 인스턴스를 매핑해주는 정보를 확인하여 연결한다.


