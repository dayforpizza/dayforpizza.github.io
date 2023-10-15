---
title: "Deny vs Drop"
categories:
    - Threat Analysis
tags:
    - deny
    - drop
header:
  teaser: /assets/img/deny_vs_drop.jpg
toc: true
---

# 침해 위협 대응
## <font color="#606060">Deny(Reject)와 Drop 차이</font>
<br>

![Deny_vs_Drop](/assets/img/deny_vs_drop.jpg)
```
Deny (Reject): 통신 시도가 올 경우 RST 패킷을 줘서 세션을 끊는 것을 말한다.
Drop: 통신 시도가 들어올 경우 해당 패킷을 버려서 어떠한 응답도 하지 않는 것을 말한다.
```
- 이 둘의 차이를 이해하려면 세션을 맺는 과정과 네트워크의 이해가 필요하다.

## <font color="#606060">애플리케이션 차단</font>
![WEB_Deny](/assets/img/app_deny.jpg)
- 웹 서비스와 같은 L7 통신의 경우 TCP(L4) 단에서 차단하지 않고 HTTP Response 를 통해 차단하여 정상적으로 세션을 끊는 경우가 많다
- TCP 연결 이후 클라이언트가 GET 요청을 할 때 비정상적인 방법으로 요청을 하는 경우 서버에서 검증 후 403 Bad Request 등의 HTTP 응답을 주고 4 Way Handshake(FIN, FIN/ACK)으로 연결을 끊는다.
- 웹방화벽같은 경우 200 OK 와 같은 정상 응답 헤더에 차단 페이지를 보여 주는 경우도 있다.