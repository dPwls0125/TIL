# TIL - `bootJar`와 `jar` 설정 차이

멀티모듈 프로젝트에서 `bootJar`와 `jar` 설정 차이  
실행 모듈인 `status-server`, `api-server`는 아래처럼 설정합니다.

```gradle
bootJar { enabled = true }
jar { enabled = false }
```

`bootJar`는 내장 톰캣과 의존성을 포함한 **실행 가능한 fat JAR**를 생성하므로  
`java -jar`로 바로 실행할 수 있습니다.  
반면 일반 `jar`는 라이브러리 배포용이므로, 실행 모듈에서는 비활성화합니다.

반대로 `common`, `common-event` 같은 라이브러리 모듈은 다음처럼 설정합니다.

```gradle
bootJar { enabled = false }
jar { enabled = true }
```

이 모듈들은 직접 실행되는 것이 아니라 다른 모듈에서 의존성으로 사용되기 때문입니다.  
결국 기준은 **이 모듈이 실행용인지, 공통 라이브러리용인지**입니다.  
실행 모듈이면 `bootJar`, 라이브러리 모듈이면 `jar`를 사용한다고 이해했습니다.
