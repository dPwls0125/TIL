**Spring Boot 설정 파일 상속(Override) 시스템**

Spring Boot에서 profile을 prod로 설정하면, application-prod.yml만 단독으로 읽지 않는다. 
1. applicatoin.yml을 먼저 읽고, 
2. 그 위에 application-prod.yml의 내용을 덮어씌우는 방식으로 동작한다.


