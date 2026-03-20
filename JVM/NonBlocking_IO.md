# TIL - 논블로킹 I/O와

블로킹 I/O의 한계와, 논블로킹 I/O를 소켓 통신 코드를 기반으로 이해해보았다. 

## 소켓(Socket)이란?

네트워크에서 두 프로그램이 통신하기 위한 연결 끝점(endpoint)이다. IP 주소 + 포트 번호로 식별된다.

- **ServerSocket**: 서버 쪽에서 특정 포트를 열고 클라이언트의 연결 요청을 기다리는 역할. 직접 데이터를 주고받지 않고, 연결 수락(accept)만 담당한다.
- **Socket**: 실제로 데이터를 읽고 쓰는 통로. ServerSocket이 연결을 accept하면 새로운 Socket이 생성되고, 이를 통해 서버 ↔ 클라이언트 간 데이터를 송수신한다.

---

## 블로킹 I/O의 구조와 문제점

### 기본 흐름

```java
while (true) {
    Socket client = serverSocket.accept();  // 클라이언트 올 때까지 멈춤
    new Thread(() -> {
        InputStream in = client.getInputStream();
        in.read(data);  // 데이터 올 때까지 또 멈춤
    }).start();
}
```

- `accept()`에서 클라이언트가 올 때까지 스레드가 멈춘다.
- `read()`에서 데이터가 올 때까지 또 멈춘다.
- 클라이언트마다 스레드를 하나씩 생성해야 하므로, 클라이언트 1000개 → 스레드 1000개가 필요하다.

### 스레드풀로 제한하면?

스레드를 4개로 제한하면 동시에 **최대 4개 클라이언트만** 처리 가능하다. 한 스레드가 특정 클라이언트의 데이터를 기다리며 멈춰 있는 동안, 나머지 클라이언트들은 큐에서 대기해야 한다.

---

## 논블로킹 I/O + Selector

### 핵심 아이디어

멈추지 않고, **준비된 채널만 골라서 처리**한다. 하나의 스레드에서 하나의 요청만 처리하는 것이 아니라, 여러개의, 혹은 여러 종류의 요청을 하나의 스레드에서 처리할 수 있음.

### 주요 개념

- **ServerSocketChannel**: ServerSocket의 NIO 버전. `configureBlocking(false)`로 설정하면 클라이언트가 없어도 멈추지 않는다.
- **Selector**: 등록된 채널들을 감시하면서, 어떤 채널이 어떤 연산(ACCEPT, READ, WRITE)을 수행할 수 있는지 알려주는 감시자.
- **SelectionKey**: 채널이 Selector에 등록될 때 생성되며, 해당 채널이 어떤 연산에 관심이 있는지(OP_ACCEPT, OP_READ 등)를 나타낸다.

### 코드 흐름

```java
// 1. 서버 채널 열기 (논블로킹 설정)
ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.bind(new InetSocketAddress(8080));
serverChannel.configureBlocking(false);

// 2. Selector 생성 및 채널 등록
Selector selector = Selector.open();
serverChannel.register(selector, SelectionKey.OP_ACCEPT);

// 3. 이벤트 루프
while (true) {
    selector.select();  // 준비된 채널이 생길 때까지 대기

    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    Iterator<SelectionKey> iter = selectedKeys.iterator();

    while (iter.hasNext()) {
        SelectionKey key = iter.next();

        if (key.isAcceptable()) {
            // 새 클라이언트 연결 → accept 후 READ 이벤트 등록
            SocketChannel client = serverChannel.accept();
            client.configureBlocking(false);
            client.register(selector, SelectionKey.OP_READ);

        } else if (key.isReadable()) {
            // 데이터 읽을 준비가 된 클라이언트 처리
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            client.read(buffer);
        }

        iter.remove();  // 처리한 키 제거
    }
}
```

### selector.select()도 블로킹 아닌가?

맞다, select()는 블로킹이다. 하지만 차이가 있다.

| | 블로킹 I/O | selector.select() |
|---|---|---|
| 기다리는 대상 | 특정 클라이언트 **1명** | 등록된 **모든 채널** |
| 깨어난 후 | 그 1명만 처리 | 준비된 것 **전부** 처리 |

---

## 단일 스레드 논블로킹의 한계

스레드 1개로 모든 채널을 처리하면, 준비된 채널이 여러 개여도 **순서대로** 처리해야 한다. 한 채널의 처리가 오래 걸리면 나머지가 밀린다.

### 해결: 채널을 그룹으로 나누기

채널 1000개를 4개 그룹으로 나누고, 각 그룹마다 Selector + 스레드를 할당한다.

**블로킹 + 스레드 4개**: 동시에 최대 4개 클라이언트만 상대 가능. 각 스레드가 담당 클라이언트 1명의 응답을 기다리며 멈춰 있음.

**논블로킹 + 스레드 4개**: 동시에 1000개 클라이언트 전부 상대 가능. 각 스레드가 자기 그룹의 250개 채널 중 준비된 것만 골라서 처리함. 기다리는 동안 놀지 않음.

같은 스레드 수라도 논블로킹은 스레드가 **대기 시간 없이 실제 일하는 데** 시간을 쓰기 때문에 효율이 훨씬 높다.




참고 도서 : 주니어 백엔드 개발자가 반드시 알아야 할 실무지식 _ 최범균
