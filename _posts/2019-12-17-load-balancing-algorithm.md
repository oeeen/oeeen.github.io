---
layout: single
title:  "Load balancing Algorithm"
date:   2019-12-16 23:00:00 +0900
classes: wide
categories: etc
tags: session
---

## 로드 밸런싱 알고리즘

- Round Robin
- Least Connection
- Least Response Time
- Least Bandwidth
- Hash
- ...

![Load balancing](/assets/img/loadbalancing/load-balancing.png)

### Round Robin

서버 부하 분산의 가장 간단한 방법. 같은 서비스를 제공하는 여러 대의 독립적인 서버들이 설정된다. 모두 같은 도메인 네임을 사용하지만 각각 독립적인 IP 주소를 갖고, DNS가 각각의 독립적인 IP의 리스트를 가지고 있고, 이걸 도메인 네임과 연결 시켜준다. 그래서 이 도메인 네임으로 요청이 들어오면, 이 주소는 순서대로 각각 독립적인 서버로 나누어서 들어간다. 단순하게 순서대로 요청을 분배하기 때문에 같은 트래픽으로 분산시키지 못한다.

### Weighted Round Robin

위의 라운드 로빈에 기반해서 가중치를 둔 라운드 로빈이다. 처음에 각 서버에 가중치를 두고, 그 가중치에 따라 분배한다. 예를 들어 A, B, C, D 서버가 있고, 각각 가중치를 5,2,2,1 로 둔다고 하면 요청이 10개 들어왔을 때 A 5개, B 2개, C 2개, D 1개로 나누어져서 들어간다. 서버간의 처리능력이 다를 때, 더 성능이 좋은 서버에 가중치를 높게 주면 좋다.

### Least Connection

요청을 분산할 때 현재 서버의 부하를 고려한다. 현재 시간의 액티브 세션의 수가 가장 적은 서버로 서빙한다.

### Least Time

부하분산을 위해서 액티브 세션과 과거 응답에 대한 가중 평균 응답 시간에 대한 조합을 수학적으로 조합한다.(nginx plus 기준) 그 조합으로 나온 결과가 가장 낮은 서버로 보낸다. nginx plus 기준으로 least_time 지시자를 통해서 응답 헤더 / 응답의 마지막 바이트 중 어떤 것을 기준으로 할 것인지 정할 수 있다.

### Least Bandwidth

현재 서비스 중인 트래픽의 양이 가장 적은 서버로 부하 분산 한다. Least Packet Method는 주어진 시간 동안 가장 적은 패킷을 받은 서비스를 부하 분산의 대상 서버로 선택한다.

### Hash

들어오는 패킷으로부터 여러 데이터의 해시를 기반으로 할 수 있다. 예를 들어 Source와 Destination IP 주소의 조합으로도 hash key를 만든다. Port Number, URL, Domain Name 등도 가능하다. 이 키를 특정 클라이언트와 특정 서버에 할당한다. 세션이 끝나면 이 키는 재생성 되어야 하는데, 클라이언트의 요청은 전에 사용했던 서버로 할당 된다.

### 참고자료

- [Nginx docs](https://www.nginx.com/blog/choosing-nginx-plus-load-balancing-techniques/)
- [citrix](https://www.citrix.com/ko-kr/glossary/load-balancing.html)
