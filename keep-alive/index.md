# Keep-Alive


## TCP Keep-Alive
**두 종단 간의 연결을 유지하는 것이 목적**이다. 연결된 TCP 소켓을 체크할 수 있고 TCP 연결이 여전히 진행중인지 혹은 끊어졌는지를 결정한다.

TCP Keepalive는 **연결된 세션의 재활용 측면에서만 아니라 좀비 커넥션의 삭제에도 도움을 준다.** TCP 연결을 끊으려면 FIN 패킷이 필요하다. 하지만 다양한 이유로 FIN 패킷을 받을 수 없는 상황이 된다면 FIN을 전달할 수 없어 계속 연결된 것처럼 남아있게 된다. TCP Keepalive 옵션을 사용한다면 일정시간이 확인 패킷을 보내는 로직을 통해 일정시간 동안 응답이 없다면 연결을 종료하기 때문에 좀비 커넥션을 방지할 수 있다.

### TCP Keep-Alive 파라미터
* net.ipv4.tcp_keepalive_time
  * keepalive 소켓의 유지시간
* net.ipv4.tcp_keepalive_probes
  * 패킷을 보낼 최대 전송 횟수
  * 네트워크 패킷은 여러 사유로 인해 유실될 수 있으므로 재전송 매커니즘을 넣는다.
* net.ipv4.tcp_keepalive_intvl
  * 재전송 패킷을 보내는 주기

### TCP Keep-Alive 동작 과정
최초로 세션이 연결된 다음 tcp_keepalive_time초 동안 기다린다. 그리고 확인 패킷을 보내게 된다. 확인 패킷에 대한 응답이 오지 않으면 tcp_keepalive_intvl 간격으로 tcp_keepalive_probes 만큼 패킷을 더 보낸다. tcp_keepalive_probes의 마지막 패킷에 대해서 응답이 오지 않으면 연결을 끊는다.
 
## HTTP Keep-Alive
최대 얼마동안 연결을 유지하도록 하는게 목적이다. HTTP 응답 Header의 예는 다음과 같다. 아래 응답을 받고 10초가 되기 전에 또 요청 했다면 Keep-Alive: timeout=10, max=19가 포함된 응답을 받을 것이다.

```
HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8
Date: Thu, 15 Jan 2015 16:45:29 GMT
Content-Length: 1845
Keep-Alive: timeout=10, max=20
```

### HTTP Keep-Alive 파라미터
2개의 Keep-Alive 파라미터는 콤파(,)분리되어 응답된다.
* timeout
  * 연결의 유지를 위한 최소한의 시간(초).
* max
  * 연결을 종료하기 전까지 보낼 수 있는 최대 요청 수.

### HTTP Keep-Alive 장점
HTTP/1.1에서 Keep-Alive로 인한 장점은 다음과 같다.
1. 3-way handshake로 인한 지연시간을 줄일 수 있다.
2. CPU 사용량을 줄여준다 : 새로운 TCP 연결을 만들기 위해서는 CPU, 메모리 사용과 같은 많은 리소스가 필요하다. 연결된 커넥션을 재사용하기 때문에 리소스 사용을 줄여준다.

## TCP Keep-Alive vs HTTP Keep-Alive
**HTTP Keep-Alive는 HTTP 프로토콜의 기능**이다. Keep-Alive 기능을 구현하는 웹 서버는 마지막 HTTP 응답(해당 HTTP 요청이 있는 경우)을 보낸 이후 시간 범위에 대해 주기적으로(수신 HTTP 요청에 대해) 연결/소켓을 확인해야 한다. 구성된 연결 유지 시간(초)까지 HTTP 요청이 수신되지 않으면 웹 서버는 연결을 닫는다. 웹 서버에서 '닫기'를 수행한 후에는 더 이상의 HTTP 요청이 불가능하다. 

반면 **TCP Keep-Alive는 TCP 계층에서 OS에 의해 관리된다.** 작은 패킷을 보내 TCP 연결을 열린 상태로 유지한다. 또한 패킷이 전송될 때 연결이 끊어지는 즉시 발신자에게 알림이 전송되도록 확인하는 역할을 한다.

## 참고
https://devidea.tistory.com/60

https://eminentstar.tistory.com/50?category=578513

https://stackoverflow.com/questions/9334401/http-keep-alive-and-tcp-keep-alive

