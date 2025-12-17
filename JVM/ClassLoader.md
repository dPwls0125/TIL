### JVM 클래스 로더 (class loader)

Class Loader는, 컴파일된 ".class"파일을 읽어, JVM 메모리에 로드하는 시스템이다. 
메모리에 적재하는 과정은, Loading -> Linking (verifiy/prepare/resolve)-> initialization 과정을 거친다. 

1) 클래스는 언제 로딩되나?
JVM은 기본적으로 interpretor 형식이다. 따라서 클래스는 보통 필요해지는 시점에 지연 로딩 된다. 

2) 전체 단계 
    1. Loading 
    2. Linking
    - verifiying 
    - preparing
    - resolvings
    3. initialization 