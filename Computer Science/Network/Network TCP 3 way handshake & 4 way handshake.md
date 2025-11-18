## TCP

두 개의 호스트를 연결하고 데이터 스트림을 교환하게 해주는 중요한 네트워크 프로토콜.
데이터와 패킷이 보내진 순서대로 전달하는 것을 보장

### 핵심 역할

- **순서 보장**: 패킷이 뒤죽박죽 도착해도 보낸 순서대로 다시 조립 가능
- **신뢰성**: 패킷 분실이나 에러 발생 시 재전송을 통해 완벽한 전달을 보증
- **혼잡 제어**: 네트워크가 혼잡하지 않도록 데이터 전송량을 조절

---

## TCP 3-way Handshake

TCP/IP 프로토콜을 이용하여 통신을 하는 응용프로그램이 데이터를 전송하기 전에 먼저 정확한 전송을 보장하기 위해 상대방 컴퓨터와 사전에 세션을 수립하는 과정을 의미

양쪽(Client와 Server) 모두 데이터를 전송할 준비가 되었다는 것을 보장하고, 실제로 데이터 전달을 시작하기 전에 한쪽에서 상대방이 받을 준비가 되었다는 것을 알 수 있도록 한다

양쪽 모두 상대편에 대한 초기 순차 일련 번호를 얻을 수 있도록 한다

TCP 접속을 성공적으로 성립하기 위하여 아래와 같은 과정이 반드시 필요

### 과정

1. **Client → Server : SYN(SYNchronize)**
    
    Client는 Server에 접속을 요청하는 SYN 패킷을 보낸다. 이때 Client는 SYN을 보내고 SYN/ACK을 기다리는 SYN-SENT(SYN을 보냄) 상태가 된다
    
2. **Server → Client : SYN ACK(ACKnowledgement)**
    
    Server는 SYN 요청을 받고 Client에게 요청을 수락한다는 ACK와 SYN flag가 설정된 패킷을 발송하고 Client가 다시 ACK으로 응답하기를 기다린다. 이때 Server는 SYN-RECEIVED(SYN을 받음) 상태가 된다
    
3. **Client > Server : ACK**
    
    Client는 Server에게 ACK을 보내고 이후로부터는 연결이 이루어지고 데이터가 오가게 되는 것이다. 이때의 Server 상태가 연결 성립(ESTABLISHED) 이다.
    

| 상태 | 설명 |
| --- | --- |
| Listen | 포트가 열린 상태로 연결 요청 대기 중 |
| SYN-SENT | SYN 요청을 보낸 상태 |
| SYN-RECEIVED | SYN 요청을 받고 상대방의 응답을 기다리는 중 |
| ESTABLISHED | 연결이 확인되고 세션이 연결됨 |
| CLOSED | 서비스 PORT가 닫힌 상태 |

---

## 4-Way Handshake

3-Way Handshake가 TCP 연결을 위해 사용했다면, 4-Way Handshake는 세션을 종료하기 위하여 수행하는 절차이다

### 과정

Client와 Server는 세션이 연결되어 있다고 가정. 둘 다 ESTABLISHED 상태

1. **Client → Server : FIN**
    
    Client가 연결을 종료하겠다고 FIN 플래그를 전송한다. 이때 Client는 FIN-WAIT 상태가 된다
    
2. **Server → Client : ACK**
    
    Servers는 FIN 플래그를 받고, 일단 확인 메시지 ACK를 보내고 자신의 통신이 끝날 때까지 기다리는 데 이 상태가 Server의 CLOSE-WAIT 상태이다
    
3. **Server → Client : FIN**
    
    Server가 연결을 종료할 준비가 되면, 연결 해지를 위한 준비가 되었음을 알리기 위해 Client에게 FIN 플래그를 전송한다. LAST-ACK 상태가 된다
    
4. **Client → Server : ACK**
    
    Client는 Server로부터 FIN 플래그를 받으면 연결 해지 준비 확인했다는 ACK 플래그를 보낸다. 이후 TIME-WAIT 상태가 되며 일정 시간 경과 후 CLOSED 상태가 된다