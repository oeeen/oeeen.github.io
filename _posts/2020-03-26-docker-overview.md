---
layout: single
title:  "Docker Overview"
date:   2020-03-26 18:50:59 +0900
classes: wide
categories: etc
tags: docker
toc: true
toc_sticky: true
---

**내용 중 제 개인 판단으로 불필요하다고 생각하는 부분은 생략했습니다.**

## Docker 공식문서 번역 - [Docker Overview](https://docs.docker.com/engine/docker-overview/)

도커는 애플리케이션 개발, 배포, 실행을 위한 플랫폼이다. 도커를 사용하면 애플리케이션을 인프라와 분리하여 빠르게 소프트웨어를 배포할 수 있다. 도커를 사용하면 코드 작성과 배포사이의 딜레이를 줄일 수 있다.

### 도커 플랫폼

도커는 컨테이너라는 격리된 환경에서 어플리케이션을 실행하고 패키징 하는 기능을 제공한다. 격리와 보안을 통해 호스트에서 여러 컨테이너를 동시에 실행할 수 있다. 컨테이너는 추가로 하이퍼바이저가 필요하지 않고 호스트 머신의 커널안에서 직접 실행되기 때문에 경량이다. 그래서 가상 머신을 사용하는 것보다 더 많은 컨테이너를 실행할 수 있다. 가상 머신 내에서도 도커 컨테이너를 띄울 수도 있다.

도커는 컨테이너의 라이프사이클을 관리하기 위한 플랫폼과 기능을 제공한다.

### 도커 엔진

도커 엔진은 다음과 같은 중요 요소가 있는 클라이언트-서버 어플리케이션이다.

- 데몬 프로세스(dockerd command)라고 하는 long-running 프로그램 유형의 서버다.
- 프로그램이 데몬과 통신하고 어떤 명령을 수행할 수 있게 하는 REST API
- CLI client(docker command)

![engine-flow](/assets/img/docker_docs/engine-components-flow.png)

CLI는 Docker REST API를 사용해서 스크립팅 or 직접 CLI 명령으로 Docker 데몬을 컨트롤한다. 다른 많은 도커 어플리케이션들은 API와 CLI를 사용한다.

데몬은 이미지, 컨테이너, 네트워크, 볼륨 같은 도커 오브젝트를 만들고 관리한다.

### 도커를 어디다 쓸까

#### 어플리케이션의 일관되고 빠른 배포

도커는 개발자가 컨테이너를 사용해서 표준화된 환경에서 어플리케이션과 서비스를 제공할 수 있도록 할 수 있게 한다.

다음과 같은 시나리오도 생각해볼 수 있다.

1. 개발자는 로컬에서 코드를 작성하고 컨테이너를 사용해서 동료와 작업을 공유한다.
2. 도커를 사용해서 어플리케이션을 테스트 환경으로 푸시 -> 자동/수동 테스트를 진행
3. 개발자가 버그를 발견하면 개발 환경에서 버그를 수정하고 테스트 환경에 재배포
4. 테스트가 완료되면 업데이트 된 이미지를 프로덕션 환경에 푸시한다.

#### 반응형 배포와 확장

도커의 컨테이너 기반 플랫폼은 쉽게 이식 가능한 워크로드를 허용한다. 도커 컨테이너는 개발자들의 로컬 노트북, 물리/가상 머신, 클라우드 등의 여러 환경에서 실행될 수 있다.

도커의 이식성과 경량성은 요구사항에 따라 거의 실시간으로 워크로드를 동적으로 관리하고 어플리케이션과 서비스를 확장/축소 할 수 있다.

#### 동일 하드웨어에서 더 많은 워크로드

도커는 가볍고 빠르다. 보통의 하이퍼바이저 기반의 가상 머신의 대체로 사용할 수 있으므로 더 많은 컨테이너를 배포할 수 있다.(동일한 하드웨어 자원으로)

### 도커 아키텍쳐

도커는 클라이언트-서버 아키텍처를 사용한다. 도커 클라이언트는 도커 데몬과 통신한다. 도커 데몬은 도커 컨테이너를 빌드, 실행, 배포하는 작업을 수행한다. 도커 클라이언트와 데몬은 같은 시스템에서 실행되거나 도커 클라이언트를 원격 도커 데몬에 연결할 수도 있다. 도커 클라이언트 및 데몬은 REST 소켓, UNIX 소켓, 네트워크 인터페이스를 통해 통신한다.

![Docker Architecture](/assets/img/docker_docs/architecture.svg)

#### 도커 데몬

도커 데몬(dockerd)는 도커 API 요청을 수신하고 도커 오브젝트(이미지, 컨테이너, 네트워크, 볼륨 같은)를 관리한다. 데몬은 다른 데몬과 통신해서 도커 서비스를 관리할 수 있다.

#### 도커 클라이언트

도커 클라이언트는 많은 도커 유저들이 도커와 통신하는 주요한 방법이다. `docker run`같은 명령을 사용할 때 클라이언트는 이 명령어를 dockerd에 보내서 명령을 수행한다. 도커 커맨드는 도커 API를 사용한다. 도커 클라이언트는 하나 이상의 데몬과 통신할 수 있다.

#### 도커 Registry

도커 레지스트리는 도커 이미지를 저장한다. Docker Hub는 누구든지 사용할 수 있는 public registry이다. 도커는 기본적으로 docker hub에서 이미지를 찾는다. 개인 레지스트리를 만들어서 사용할 수도 있다. 만약 도커 데이터센터를 사용한다면, 그것은 Docker Trusted Registry를 포함한다(?)

`docker pull` 이나 `docker run` 커맨드는 필요한 이미지를 미리 설정된 레지스트리에서 가져온다. `docker push` 커맨드는 설정된 레지스트리에 푸시한다.

### 도커 오브젝트

도커를 사용하면 이미지, 컨테이너, 네트워크, 볼륨, 플러그인 및 기타 객체를 생성, 사용하게 된다.

#### IMAGES

이미지는 도커 컨테이너를 만들기 위한 Read-Only 템플릿이다. 이미지는 보통 다른 이미지를 기반으로 하고, 추가 커스터마이징이 있다. 예를 들어 ubuntu 이미지를 기반으로 apache web server와 내 어플리케이션을 설치하고 그 어플리케이션을 실행하기 위한 세부 정보를 설치할 수 있다.

자신의 이미지나, 다른사람이 만들어서 레지스트리에 올린 이미지만 사용할 수 있다. 본인의 이미지를 만들려면 Dockerfile을 작성해야한다. Dockerfile 내부의 명령어는 이미지에 layer를 만든다. Dockerfile을 바꾸고 rebuild하면 바뀐 부분의 layer만 rebuild된다. 이런 부분이 다른 가상화 기술과 다르게 이미지를 작고, 빠르고, 경량으로 만들게 해준다.

#### CONTAINERS

컨테이너는 실행가능한 이미지 인스턴스이다. 도커 API나 CLI를 사용해서 create, start, stop, move, delete 할 수 있다. 컨테이너를 하나 이상의 네트워크에 연결할 수 있고, 스토리지를 연결하고, 현재 상태를 기반으로한 새 이미지를 만들 수도 있다.

기본적으로 컨테이너는 다른 컨테이너와 그 호스트 머신과 잘 격리되어 있다.

컨테이너는 이미지를 만들거나 시작할 때 제공하는 구성 옵션과 그것의 이미지를 통해 정의된다.(이미지를 통해서 띄울수 있다는 뜻인듯) 컨테이너를 제거하면, 스토리지에 저장하지 않은 상태의 변경은 사라진다.(볼륨을 연결해서 호스트 스토리지에 저장하지 않은 변경은 사라진다는 뜻)

`docker run -i -t ubuntu /bin/bash` 과 같은 명령으로 docker run 명령을 실행할 수 있다. 이 명령을 수행하면 다음과 같은 일이 일어난다.

1. ubuntu 이미지가 로컬에 없으면 `docker pull ubuntu` 명령을 실행 한 것처럼 레지스트리에서 이미지를 가져온다.
2. `docker container create` 명령처럼 새 컨테이너를 만든다.
3. 도커는 read-write 파일 시스템을 컨테이너에 마지막 레이어로 연결하고, 이 파일 시스템을 통해 컨테이너가 로컬 파일시스템에서 파일과 디렉토리를 만들거나 수정할 수 있다.
4. 네트워크 옵션을 따로 설정하지 않았으므로 default 네트워크 인터페이스를 만든다. 컨테이너는 호스트의 네트워크를 사용해서 외부 네트워크에 연결할 수 있다.
5. 컨테이너를 시작하고 /bin/bash를 실행한다. 컨테이너는 대화형(-i), 터미널(-t)로 연결된다.
6. exit를 통해 컨테이너를 중지할 수 있지만 삭제는 되지 않는다. 다시 시작하거나 제거할 수 있다.

#### SERVICES

서비스를 사용하면 여러 도커 데몬에서 컨테이너를 확장할 수 있다. 도커 데몬은 여러 매니저와 워커와 함께 Swarm으로 작동한다.(??) Swarm의 각 멤버는 도커 데몬이고 데몬은 각 도커 API를 사용해서 통신한다. 서비스는 특정 시점에 사용해야하는 서비스 복제본 수 같은, 원하는 상태를 정의할 수 있다. 기본적으로 서비스는 모든 워커 노드에서 로드밸런싱 된다. 소비자에게 도커서비스는 단일 어플리케이션처럼 보인다. Swarm Mode는 Docker 1.12이상에서 지원한다.

### 기반 기술

도커는 Go로 쓰여있고, 리눅스 커널의 여러 기능을 활용하여 기능을 제공한다.

#### Namespaces

Docker uses a technology called namespaces to provide the isolated workspace called the container. When you run a container, Docker creates a set of namespaces for that container.

These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace.

Docker Engine uses namespaces such as the following on Linux:

- The pid namespace: Process isolation (PID: Process ID).
- The net namespace: Managing network interfaces (NET: Networking).
- The ipc namespace: Managing access to IPC resources (IPC: InterProcess Communication).
- The mnt namespace: Managing filesystem mount points (MNT: Mount).
- The uts namespace: Isolating kernel and version identifiers. (UTS: Unix Timesharing System).

도커는 컨테이너라는 격리된 작업공간을 제공하기 위해 네임스페이스라는 기술을 사용한다.컨테이너를 실행할 때, 도커는 그 컨테이너를 위한 네임스페이스 Set을 만든다. 그 네임스페이스는 격리된 layer를 제공한다. 컨테이너의 각 측면은 별도의 네임스페이스에서 실행되고 그 네임스페이스로 접근이 제한된다.

도커 엔진은 리눅스에서 다음과 같은 네임스페이스를 사용한다.

- pid namespace: 프로세스 격리(Process ID)
- net namespace: 네트워크 인터페이스 관리(networking)
- ipc namespace: IPC 자원 접근 관리(Interprocess Communication)
- mnt namespace: 마운트포인트 관리(mount)
- uts namespace: 커널 격리, 버전 식별자(Unix Timesharing System)

#### Control groups

리눅스의 도커엔진은 control group이라는 기술에 의존한다. cgroup은 어플리케이션은 특정 리소스의 집합으로 제한한다. control group을 통해 도커 엔진은 하드웨어 자원을 컨테이너와 공유하고 한계와 제한을 선택적으로 할 수 있다. 예를 들어 특정 컨테이너에 사용 가능한 메모리를 제한 할 수 있다.

#### Union file systems

UnionFS는 layer를 만들어서 가볍고 빠르게 만드는 파일 시스템이다. 도커 엔진은 UnionFS를 사용해서 컨테이너의 building block을 제공한다. 도커엔진은 AUFS, btrfs, vfs, DeviceMapper를 포함한 다양한 UnionFS의 변형을 사용할 수 있다.

#### Container format

도커 엔진은 namespace, control group, UnionFS를 컨테이너라고 하는 wrapper로 조합한다. 기본 컨테이너 형식은 libcontainer다. Docker는 미래에 BSD Jail or Solaris Zones 같은 기술을 통합해서 다른 컨테이너 형식을 지원할 수도 있다.

전반적으로 무슨 소리인지 잘 모르겠다;
