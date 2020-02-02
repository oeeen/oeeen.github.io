---
layout: single
title:  "Docker Basic"
date:   2019-08-07 17:30:59 +0900
classes: wide
categories: web
toc: true
toc_sticky: true
---

## Docker Basic

도커는 컨테이너 기반의 오픈소스 가상화 플랫폼이다.

- 격리된 공간에서 프로세스가 동작하는 기술.
- 가상화 기술의 하나지만 기존 방식과는 차이가 있다.

전가상화든 반가상화든 추가적인 OS를 설치하여 가상화 하는 방법은 성능 문제가 있어서, 이를 개선 하기 위해 프로세스를 격리하는 방식이 등장했다.

리눅스에서는 이 방식을 리눅스 컨테이너라고 하고 단순히 프로세스를 격리시키기 때문에 가볍고 빠르게 동작한다. CPU나 메모리는 딱 프로세스가 필요한 만큼만 추가로 사용하고 성능적으로도 거의 손실이 없다.

하나의 서버에 여러개의 컨테이너를 실행하면 서로 영향을 미치지 않고 독립적으로 실행되어 마치 가벼운 VM을 사용하는 느낌을 준다. 실행 중인 컨테이너에 접속하여 명령어를 입력할 수 있고, apt-get이나 yum으로 패키지를 설치할 수 있으며 사용자도 추가하고 여러개의 프로세스를 백그라운드로 실행할 수도 있다. CPU나 메모리 사용량을 제한할 수 있고 호스트의 특정 포트와 연결하거나 호스트의 특정 디렉토리를 내부 디렉토리인 것 처럼 사용할 수도 있다.

## 이미지

이미지는 컨테이너 실행에 필요한 파일과 설정값등을 포함하고 있는 것으로 상태값을 가지지 않고 변하지 않는다. 컨테이너는 이미지를 실행한 상태라고 볼 수 있고 추가되거나 변하는 값은 컨테이너에 저장된다. 같은 이미지에서 여러개의 컨테이너를 생성할 수 있고 컨테이너의 상태가 바뀌거나 컨테이너가 삭제되더라도 이미지는 변하지 않고 그대로 남아있다.

도커 이미지는 레이어 라는 개념을 사용하고 유니온 파일 시스템을 이용하여 여러개의 레이어를 하나의 파일 시스템으로 사용할 수 있게 해준다.

이미지는 여러개의 읽기 전용 레이어로 구성되고 파일이 추가되거나 수정되면 새로운 레이어가 생성된다.

예를 들어..

Ubuntu 이미지가 A + B + C의 집합이라면, ubuntu 이미지를 베이스로 만든 nginx 이미지는 A + B + C + nginx가 된다. webapp 이미지를 nginx 이미지 기반으로 만들었다면,, A + B + C + nginx + source 레이어로 구성된다. 여기서 webapp을 수정하면 source 레이어만 수정버전을 다운받으면 된다. 매우 효율적으로 이미지를 관리할 수 있다.

컨테이너를 생성할 때도 레이어 방식을 사용하는데 기존의 이미지 레이어 위에 읽기/쓰기 레이어를 추가한다. 이미지 레이어를 그대로 사용하면서 컨테이너가 실행중에 생성하는 파일이나 변경된 내용은 읽기/쓰기 레이어에 저장되므로 여러개의 컨테이너를 생성해도 최소한의 용량만 사용한다.

도커는 이미지를 만들기 위해 Dockerfile이라는 파일에 자체 DSL\*(Domain-specific language)언어를 이용하여 이미지 생성 과정을 적는다.

## Docker Run 옵션

일단 Docker를 써보자. Docker run 명령에는 다양한 옵션이 있다.

![Docker Run Help](/assets/img/docker/docker_1.png)
![Docker Run Help2](/assets/img/docker/docker_2.png)

이렇게나 설명이 잘 나와있다.

이 중에 내가 써본 것들만 설명을 써보면..

옵션 | 설명
--- | ---
-i | 상호 작용 할 수 있게 한다. (Keep STDIN open even if not attached)
-t | tty 사용 (Allocate a pseudo-TTY, 터미널 환경 쓸수 있게 된다.)
-d | Container를 백그라운드로 실행하고 Container ID를 출력한다. (Run container in background and print container ID)
-e | 환경변수 설정. (예를 들어 mysql을 사용한다고 할때 root의 패스워드 설정이라거나,, 여러 환경변수를 설정할 수 있다.) - (Set environment variables)
-v | 볼륨 연결. (Container와 나의 로컬 서버의 볼륨을 연결해줄 수 있다. 그러니까 docker를 띄운 그 서버에서 container의 해당 볼륨을 접근할 수 있게 된다) - (Bind mount a volume)
-p | 포트 연결. (Container의 포트와 호스트의 포트를 연결? 매핑 해준다고 생각하면 된다. ex. 8000:8080) - (Publish a container's port(s) to the host)
-u | Username or UID
--name | 컨테이너 이름 설정 (Assign a name to the container)

```bash
sudo docker run --name jenkins -itd -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd)/jenkins_home:/var/jenkins_home -p 8000:8080 -u root

sudo docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

이런 명령어들을 썼었다.
