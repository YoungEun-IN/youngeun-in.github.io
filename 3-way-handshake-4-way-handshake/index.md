# 3 way handshake, 4 way handshake


## 3-Way Handshake

3-Way Handshake는 TCP 통신을 이용하여 데이터를 전송하기 위해 네트워크 연결을 설정(Connection Establish) 하는 과정이다. 
양쪽 모두 데이터를 전송할 준비가 되었다는 것을 보장하고, 실제로 데이터 전달이 시작하기 전에 한 쪽이 다른 쪽이 준비되었다는 것을 알 수 있도록 한다.
즉, TCP/IP 프로토콜을 이용해서 통신을 하는 응용 프로그램이 데이터를 전송하기 전에 먼저 **정확한 전송을 보장하기 위해 상대방 컴퓨터와 사전에 세션을 수립하는 과정**을 의미한다.

### 작동 방식

![image](https://user-images.githubusercontent.com/46465928/163965059-8ae4fa1b-dc75-4389-ba97-9971f29bbccb.png)

1. Step 1 (SYN)
  - 클라이언트는 서버와 커넥션을 연결하기 위해 SYN을 보낸다. (seq : x)
    - 송신자가 최초로 데이터를 전송할 때 Sequence Number를 임의의 랜덤 숫자로 지정하고, SYN 플래그 비트를 1로 설정한 세그먼트를 전송한다.
    - PORT 상태
      - Client : CLOSED- SYN_SENT 로 변함
      - Server : LISTEN

2. Step 2 (SYN + ACK)
  - 서버가 SYN(x)을 받고, 클라이언트로 받았다는 신호인 ACK와 SYN 패킷을 보냄 (seq : y, ACK : x + 1)
    - 접속요청을 받은 Q가 요청을 수락했으며, 접속 요청 프로세스인 P도 포트를 열어달라는 메세지를 전송 (SYN-ACK signal bits set)
    - ACK Number필드를 Sequence Number + 1 로 지정하고 SYN과 ACK 플래그 비트를 1로 설정한 새그먼트 전송 (Seq=y, Ack=x+1, SYN, ACK)
    - PORT 상태
      - Client : CLOSED
      - Server : SYN_RCV

3. Step 3 (ACK)
  - 클라이언트는 서버의 응답은 ACK(x+1)와 SYN(y) 패킷을 받고, ACK(y+1)를 서버로 보낸다.
    - 마지막으로 접속 요청 프로세스 P가 수락 확인을 보내 연결을 맺는다. (ACK)
    - 이때, 전송할 데이터가 있으면 이 단계에서 데이터를 전송할 수 있다.
    - PORT 상태
      - Client : ESTABLISED
      - Server : SYN_RCV ⇒ ACK ⇒ ESTABLISED

## 4-Way Handshake

4-Way Handshake은 **연결을 해제(Connecntion Termination)하는 과정**이다. 

### 작동방식 (Abrupt)
RST(TCP reset) 세그먼트가 전송되면 갑작스러운 연결 해제가 수행되는데, RST 세그먼트는 다음과 같은 경우에 전송된다.

- 존재하지 않는 TCP 연결에 대해 비SYN 세그먼트가 수신된 경우
- 열린 커넥션에서 일부 TCP 구현은 잘못된 헤더가 있는 세그먼트가 수신된 경우
  - RST 세그먼트를 보내, 해당 커넥션을 닫아 공격을 방지한다.
- 일부 구현에서 기존 TCP 연결을 종료해야 하는 경우
  - 연결을 지원하는 리소스 부족할때
  - 원격 호스트에 연결할 수 없고 응답이 중지되었을때

### 작동방식 (Graceful)

![image](https://user-images.githubusercontent.com/46465928/163968315-9d5b4649-2578-4aba-be50-442a3493d62d.png)

{{< admonition note "Half-Close 기법" true >}}
위 그림에서 처음 보내는 종료 요청인 (1) FIN 패킷에 실질적으로 ACK가 포함되어 있는 것을 알 수 있는데, 이는 Half-Close 기법을 사용하기 때문이다.
즉, 연결을 종료하려고 할 때 완전히 종료하지 않고 반만 종료한다.

Half-Close 기법을 사용하면 종료 요청자가 처음 보내는 FIN 패킷에 승인 번호를 함께 담아서 보내게 되는데, 이때 승인 번호의 의미는 "일단 연결은 종료할건데 귀는 열어둘게. 이 승인 번호까지 처리했으니까 더 보낼 거 있으면 보내"가 된다.
이후 수신자가 남은 데이터를 모두 보내고 나면 다시 요청자에게 FIN 패킷을 보냄으로써 모든 데이터가 처리되었다는 신호 FIN를 보낸다. 그럼 요청자는 그때 나머지 반을 닫으면서 좀 더 안전하게 연결을 종료할 수 있다.
{{< /admonition >}}

1. STEP1 (Client → Server : FIN(+ACK)
  - 서버와 클라이언트가 연결된 상태에서 클라이언트가 close()를 호출하여 접속을 끊는다(끊으려한다).
  - 이때, 클라이언트는 서버에게 연결을 종료한다는 FIN 플래그를 보낸다.
    - 이때 FIN 패킷에는 실질적으로 ACK도 포함되어있다.

2. STEP2 (Server → Client : ACK)
  - 서버는 FIN을 받고, 확인했다는 ACK를 클라이언트에게 보내고 자신의 통신이 끝날때까지 기다린다. (이상태가 TIME_WAIT 상태)
    - Server(수신자)는 ACK Number 필드를 (Sequence Number + 1)로 지정하고, ACK 플래그 비트를 1로 설정한 세그먼트를 전송한다.
  - 서버는 클라이언트에게 응답을 보내고 CLOSE_WAIT 상태에 들어갑니다. 그리고아직 남은 데이터가 있다면 마저 전송을 마친 후에 close( )를 호출
  - 클라이언트에서는 서버에서 ACK를 받은 후에 서버가 남은 데이터 처리를 끝내고 FIN 패킷을 보낼 때까지 기다린다. (FIN_WAIT_2)

3. STEP3 (Server → Client : FIN)
  - 데이터를 모두 보냈다면, 서버는 연결이 종료에 합의 한다는 의미로 FIN 패킷을 클라이언트에게 보낸 후에, 승인 번호를 보내줄 때까지 기다니는 LAST_ACK 상태로 들어간다.

4. STEP4 (Client → Server : ACK)
  - 클라이언트는 FIN을 받고, 확인했다는 ACK를 서버에게 보낸다.
  - 아직 서버로부터 받지 못한 데이터가 있을 수 있으므로 TIME_WAIT을 통해 기다린다. (실질적인 종료과정 CLOSED에 들어간다.)
    - 이때 TIME_WAIT 상태는 의도치 않은 에러로 인해 연결이 데드락으로 빠지는 것을 방지
    - 만약 에러로 인해 종료가 지연되다가 타임이 초과되면 CLOSED로 들어간다.

## 참고
https://velog.io/@averycode/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-TCPUDP%EC%99%80-3-Way-Handshake4-Way-Handshake

