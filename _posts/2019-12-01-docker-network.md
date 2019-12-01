---
layout: single
title:  "Networking with standalone containers"
date:   2019-12-01 17:50:59 +0900
classes: wide
categories: etc
tags: docker
---

## Docker 공식문서 번역 - [Networking with standalone containers](https://docs.docker.com/network/network-tutorial-standalone/)

이 글은 Standalone 도커 컨테이너의 네트워크를 다루기 위한 튜토리얼 문서입니다.

**내용 중 제 개인 판단으로 불필요하다고 생각하는 부분은 생략했습니다.**

이 글에는 아래 네트워크에 대한 설명을 한다. 그러나 overlay network는 다른 문서에서 설명한다.

1. Default bridge network
2. User-defined bridge network
3. overlay network

## Default Bridge Network

이 예시에서 동일한 도커 호스트에서 서로 다른 alpine 컨테이너를 시작하고, 그 두 컨테이너끼리 어떻게 통신하는지 이해하기 위해서 테스트를 해볼 것이다. 도커가 설치되어 있고 실행 중이어야 한다.

### 1. 터미널을 열고 현재 도커 네트워크를 살펴본다

`docker network ls`를 실행해보면, 나의 경우는 아래 그림처럼 나온다.

![docker network ls](/assets/img/docker_docs/docker_network_ls.png)

도커 공식 문서에 나와있는 예시를 보면 아래와 같다.

```bash
$docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
17e324f45964        bridge              bridge              local
6ed54d316334        host                host                local
7092879f2cc8        none                null                local
```

여기서 뒤에 2개는 완전한 네트워크는 아니지만 도커 데몬 호스트의 네트워크 스택에 직접 연결된 컨테이너를 시작하거나, 네트워크 디바이스가 없는 컨테이너를 시작할 때 사용된다.이 튜토리얼에서는 두 개의 컨테이너를 bridge 네트워크에 연결한다.

### 2. 2개의 alpine 컨테이너(ash가 실행중인, ash는 bash같은 alpine의 default shell이다.)를 실행한다

-dit 옵션은 detached (백그라운드), interactive, TTY사용 하겠다 라는 뜻이다. 그리고 명령에서 --network flag를 설정하지 않았기 때문에, default bridge network에 연결된다.

예시로 살펴보면,

```bash
$docker run -dit --name alpine1 alpine ash
$docker run -dit --name alpine2 alpine ash
```

![run alpine](/assets/img/docker_docs/run_alpine.png)
![container ls](/assets/img/docker_docs/container_ls.png)

### 3. 이제 bridge network를 살펴보자

![inspect bridge](/assets/img/docker_docs/inspect_bridge.png)

위 그림에서 Container를 살펴보면 위에서 만든 alpine container 두개가 할당되어 있는 것을 볼 수 있다. 그리고 그 해당하는 ip도 나와있다. bridge network의 gateway(172.17.0.1), alpine1(172.17.0.2), alpine2(172.17.0.3)인 것도 확인할 수 있다.

### 4. 백그라운드로 돌고 있는 컨테이너를 attach 커맨드를 이용해서 연결 해보자

`docker attach alpine1` 을 실행 한다. 그 이후 / # 이 나오면 `ip addr show`를 실행 해본다.

![ip addr show](/assets/img/docker_docs/ip_addr_show.png)

첫줄에 LOOPBACK은 현재는 무시해도 된다. 그냥 마지막에 eth0@if13 의 inet 172.17.0.2를 보면, 위에서 살펴본 alpine1의 172.17.0.2와 동일한 것을 알 수 있다.

### 5. 이제 alpine1에서 인터넷에 연결되어 있는지 확인하기 위해 google.com에 ping 해본다

![ping google](/assets/img/docker_docs/ping_google.png)

다음으로 172.17.0.3에도 ping 해보면 성공하는 것을 알 수 있다. 물론, alpine2라는 식으로 컨테이너 네임으로 ping 해보면 실패한다.(`ping -c alpine2` 같은 방식)

![ping 3](/assets/img/docker_docs/ping_3.png)

이제 alpine1, alpine2를 stop, remove 한다.

```bash
$docker container stop alpine1 alpine2
$docker container rm alpine1 alpine2
```

default bridge network는 프로덕션 상황에서는 추천하지 않는다. 프로덕션 상황에서는 user-defined bridge network를 사용해라.

## User-defined bridge network

이 예시에서는 다시 alpine 컨테이너 두개로 시작하지만, 그 컨테이너들을 user-defined 네트워크(alpine-net이라고 부를..)에 붙일 것이다. 이 컨테이너들은 default bridge network에 전혀 연결되지 않는다. 그리고 브릿지 네트워크는 연결, alpine-net에는 연결되지 않은 3번째 alpine 컨테이너를 시작하고, 4번째 컨테이너는 둘다 연결된 컨테이너를 시작하는 방식으로 예시를 진행할 것이다.

### 1. 일단 alpine-net이라는 네트워크를 만든다

`$ docker network create --driver bridge alpine-net`을 실행한다. 사실 --driver bridge 옵션은 default 옵션이라 빼도 된다.(명시적으로 하기 위해 써두었다.)

![network create](/assets/img/docker_docs/network_create.png)

### 2. 그리고 alpine-net의 네트워크를 상세히 살펴보기 위해 다음 명령을 실행한다. `docker network inspect alpine-net`

![inspect alpine](/assets/img/docker_docs/inspect_alpine_net.png)

좀 살펴보면 Gateway가 172.22.0.1로 되어있는 것을 볼 수 있는데, 이건 각자의 시스템 상태마다 다르게 나올 것 이다.

### 3. 이제 처음에 하기로 했던 4개의 컨테이너를 만들어본다

--network 옵션을 생각하면서 넣어보자. 하나의 docker run 커맨드 중에는 하나의 네트워크에만 연결할 수 있다. 그래서 alpine4는 그 이후 커맨드로 추가해주어야 한다.

```bash
$docker run -dit --name alpine1 --network alpine-net alpine ash
$docker run -dit --name alpine2 --network alpine-net alpine ash
$docker run -dit --name alpine3 alpine ash
$docker run -dit --name alpine4 --network alpine-net alpine ash
$docker network connect bridge alpine4
```

![network container](/assets/img/docker_docs/docker_network_container.png)

![user-defined container ls](/assets/img/docker_docs/user_defined_container_ls.png)

### 4. network inspect

이렇게 컨테이너를 만든 이후에 network inspect 해보자.

alpine-net은 아래와 같다. containers라는 곳에 alpine1, alpine2, alpine4가 추가 된 것을 볼 수 있다.

![inspect_alpine_net_after](/assets/img/docker_docs/inspect_alpine_net_after.png)

bridge network는 아래와 같다. containers에 alpine3, alpine4가 추가 된 것을 볼 수 있다.

![inspect_bridge_after](/assets/img/docker_docs/inspect_bridge_after.png)

### 5. alpine-net 같은 user-defined network에서는 컨테이너들끼리는 IP address 말고도 컨테이너 이름으로도 통신할 수 있다

이런 능력(?)을 automatic service discovery라고 한다. alpine1을 연결해서 이걸 테스트 해보자. 생각으로는 alpine1은 alpine2와 alpine4와 IP address로 통신 할 수 있을 것이다.

```bash
$docker container attach alpine1

# ping -c 2 alpine2

PING alpine2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.085 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.090 ms

--- alpine2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.085/0.087/0.090 ms

# ping -c 2 alpine4

PING alpine4 (172.18.0.4): 56 data bytes
64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.076 ms
64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.091 ms

--- alpine4 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.076/0.083/0.091 ms

# ping -c 2 alpine1

PING alpine1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.026 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.054 ms

--- alpine1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.026/0.040/0.054 ms
```

![ping alpine net](/assets/img/docker_docs/ping_alpine_net.png)

### 6. 같은 네트워크가 아닌 네트워크에 통신

그러나 같은 네트워크가 아닌 alpine3에 연결하려고 하면 실패 할 것이다.

![bad_address_alpine3](/assets/img/docker_docs/bad_address_alpine3.png)

이제 alpine1에서 작업은 끝났으니 detach한다.(Ctrl + p -> q)

### 7. alpine4는 default bridge와 alpine-net에 모두 연결되어 있다

그러면 아마 다른 모든 컨테이너에 연결할 수 있을 것 같다.. 하지만 alpine3에는 ip address가 필요할 것이다. 테스트를 해보자

```bash
$docker container attach alpine4

# ping -c 2 alpine1

PING alpine1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.074 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.082 ms

--- alpine1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.074/0.078/0.082 ms

# ping -c 2 alpine2

PING alpine2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.075 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.080 ms

--- alpine2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.075/0.077/0.080 ms

# ping -c 2 alpine3
ping: bad address 'alpine3'

# ping -c 2 172.17.0.2

PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.089 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.075 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.075/0.082/0.089 ms

# ping -c 2 alpine4

PING alpine4 (172.18.0.4): 56 data bytes
64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.033 ms
64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.064 ms

--- alpine4 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.033/0.048/0.064 ms
```

![ping_in_alpine4](/assets/img/docker_docs/ping_in_alpine4.png)

여기서 `ping -c alpine3`이 실패했는데, 위의 docker inspect bridge에서 나온 결과에서처럼 172.17.0.2에 바로 ping 하면 서로 통신을 할 수 있는 것을 알 수 있다.

그 다음은 인터넷 연결이 되어있는지 확인하기 위해 각 alpine 컨테이너에서 ping google 하는 내용이 나온다.(불필요 하다고 생각)

이제 모든 실습이 끝났으니, 컨테이너와 네트워크를 지우고 초기상태로 만든다.

```bash
$docker container stop alpine1 alpine2 alpine3 alpine4
$docker container rm alpine1 alpine2 alpine3 alpine4
$docker network rm alpine-net
```

내용은 매우 간단한 내용이었으나, 코드 길이만 길어서 읽기 어려운 글이 된 것 같다. 도커 네트워크 문서 중 좀 도움이 될만한 문서를 찾아 번역해보아야겠다.

### 출처

- [Networking with standalone containers](https://docs.docker.com/network/network-tutorial-standalone/)
