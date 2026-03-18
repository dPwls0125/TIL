## gradle 관련 디렉터리 차이 정리 

Gradle 문서를 읽다가 gradle/, build/, .gradle/ 디렉터리의 차이를 정리했다. 

“While ```gradle``` is usually checked into **source control**, ```build``` and ```.gradle``` directories contain the output of your builds, caches, and other transient files...”  
라는 문장이 굉장히 난해하게 느껴졌다.

**핵심은 Git에 올려야 하는 파일과 올리면 안 되는 파일을 구분하는 것이다.**

우선 gradle/ 디렉터리와 gradlew, gradlew.bat 같은 파일은 프로젝트를 실행하는 데 필요한 설정 파일이기 때문에 보통 소스 코드와 함께 버전 관리한다.   
특히 Gradle Wrapper와 관련된 파일들은 팀원들이 같은 Gradle 버전으로 프로젝트를 실행할 수 있게 해준다.

반면 build/와 .gradle/은 성격이 다르다.
build/는 컴파일 결과물, 테스트 결과, jar 파일 등 빌드를 수행한 뒤 생성되는 산출물이 들어가는 곳이다.
.gradle/은 캐시나 임시 파일처럼 Gradle이 빌드 속도를 높이기 위해 사용하는 보조 데이터가 저장되는 곳이다.

즉, gradle/은 프로젝트 실행에 필요한 파일이고,
build/와 .gradle/은 실행 과정에서 생기는 결과물이다.

또한 여기서 **incremental build**라는 개념도 함께 이해했다.
이는 매번 전체를 새로 빌드하는 것이 아니라, 변경된 부분만 다시 처리해서 빌드 시간을 줄이는 방식이다. Gradle은 이를 위해 .gradle/ 같은 디렉터리에 캐시와 기록을 남긴다.

정리하면,
```
gradle/과 Wrapper 파일은 Git에 올린다.
build/, .gradle/은 빌드 결과물과 캐시이므로 보통 Git에 올리지 않는다.
```
