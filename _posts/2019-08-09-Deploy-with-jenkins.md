---
layout: single
title:  "나의 웹 어플리케이션을 배포해보자(with Jenkins, Docker) - 1"
date:   2019-08-09 00:30:59 +0900
classes: wide
categories: web
tags: 
---

# 나의 웹 어플리케이션을 jenkins로 배포 해보자

일단 목표는 젠킨스 서버를 띄우고 젠킨스 서버에서 git repository pull(push가 들어오면 - 특정 브랜치) 한 후 build 하고 자동으로 배포까지 진행되는 것을 목표로 했다. (**무중단 배포**는 나중일이다..)

그래서 일단 EC2 instance를 만들었다 (ubuntu 18.04)

인스턴스를 만들고, 해당 키를 저장해둔다.(나는 KEY-TRAINING-oeeen.pem 으로 했다.)

해당 키를 읽기 권한을 주고 ssh로 접속한다.

```bash
chmod 400 KEY-TRAINING-oeeen.pem
ssh -i  KEY-TRAINING-oeeen.pem ubuntu@[EC2-public-ip]
```

이제 EC2 ubuntu server에 접속이 되었다. 일단 jenkins 설치를 위해 docker를 설치한다

`sudo snap install docker`

도커를 깔았으니.. 이제 젠킨스 컨테이너를 띄운다. 그런데.. 젠킨스 서버가 docker를 포함하고 있었으면 좋겠다.

왜냐하면 젠킨스에서 빌드 완료된 jar file을 실행하는 컨테이너가 필요했기 때문이다..

젠킨스 컨테이너만을 사용하려면 

`sudo docker run --name myblog -d -p 8000:8080 -p 50000:50000 jenkins/jenkins:lts` 

로 실행하면 된다.

그런데 위와 같은 필요사항이 있었기 때문에! Dockerfile을 만드는 Docker 자체의 DSL을 살짝 공부해야 한다. 

## Dockerfile을 만들어보자

도커 파일은 [Dockerfile reference](https://docs.docker.com/engine/reference/builder/) 를 참고 했다.

docker build 커맨드는 Dockerfile과 context로부터 도커 이미지를 만드는 커맨드다! (The docker build command builds an image from a Dockerfile and a context)

`sudo docker build [OPTIONS] PATH | URL | -`
옵션 중에 내가 사용해본 옵션은 아래와 같다

옵션 | 설명
--- | ---
--tag, -t | 태그를 달 수 있다. Name and optionally a tag in the ‘name:tag’ format
--file, -f | Dockerfile 이름, Name of the Dockerfile (Default is ‘PATH/Dockerfile’)

아무튼 docker build를 이용해서 내가 원하는 도커 이미지를 만들어야 한다.


### **FROM**

* `FROM <image> [AS <name>]`
* `FROM <image>[:<tag>] [AS <name>]`
* `FROM <image>[@<digest>] [AS <name>]`

어떤 docker image로부터... 그 이미지의 이름은.. name으로.. 이런식이다.

### **RUN**

`RUN <command>` (shell form, the command is run in a shell, which by default is /bin/sh -c on Linux or cmd /S /C on Windows)

쉘 커맨드를 쓸 수 있다. default는 linux의 /bin/sh -c 나 윈도우의 cmd /S /C 라고 한다.

일단 [jenkins/jenkins:lts](https://github.com/jenkinsci/docker/blob/587b2856cd225bb152c4abeeaaa24934c75aa460/Dockerfile) 는 `FROM openjdk:8-jdk` 이다.

### **CMD**

* `CMD ["executable","param1","param2"] (exec form, this is the preferred form)`
* `CMD ["param1","param2"] (as default parameters to ENTRYPOINT)`
* `CMD command param1 param2 (shell form)`

**Dockerfile 내의 CMD 명령은 하나만 된다. 여러개를 쓰면 결국 맨 마지막에 있는 CMD 명령만 수행 된다!**

CMD 의 주 목적은 컨테이너 실행하는데 default를 제공 하기 위해서 쓴다. (The main purpose of a CMD is to provide defaults for an executing container.)

### **그 외..**

* LABEL - `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
* EXPOSE - `EXPOSE <port> [<port>/<protocol>...]` ex) EXPOSE 80/udp
* ENV 
```
ENV <key> <value>
ENV <key>=<value> ...
```
* ADD
```
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```
* COPY
```
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```
* ENTRYPOINT
```
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)
ENTRYPOINT command param1 param2 (shell form)
```
* VOLUME - `VOLUME ["/data"]`
* USER
```
USER <user>[:<group>] or
USER <UID>[:<GID>]
```
* WORKDIR - `WORKDIR /path/to/workdir`
* ARG - `ARG <name>[=<default value>]`
* ONBUILD - `ONBUILD [INSTRUCTION]`
* STOPSIGNAL - `STOPSIGNAL signal`
* HEALTHCHECK -
```
HEALTHCHECK [OPTIONS] CMD command (check container health by running a command inside the container)
HEALTHCHECK NONE (disable any healthcheck inherited from the base image)
```
* SHELL - `SHELL ["executable", "parameters"]`

이렇게 명령어가 너무 많다... 지금 다 알아보기는 힘들 것 같다. 사용하면서 필요하다고 느끼는 것을 차차 알아가면 될 것 같다.

아래와 같은 Dockerfile 을 만든다. 불필요한 명령어들이 있을 수 있다.

```
# Dokerfile
FROM jenkins/jenkins:lts
# jenkins 이미지에서 user를 jenkins로 바꾸는 부분이 있다. 다시 root로..
USER root
# docker 설치 과정
RUN apt-get update && \
apt-get -y install apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common && \
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
    $(lsb_release -cs) \
    stable" && \
apt-get update && \
apt-get -y install docker-ce
RUN apt-get install -y docker-ce
RUN usermod -a -G docker jenkins
USER jenkins
```

`docker build -t oeeen/jenkins:v1 .` 이렇게 만들어진 Dockerfile을 build하고.. 이 빌드 완료된 이미지를 실행 한다.

근데 나는 맥에서 도커 이미지를 빌드 했고, 이 이미지를 ec2 인스턴스에서 사용하고 싶었다. 그래서 dockerhub에 이미지를 올려야 했다.

아래 방법으로 docker hub에 푸시 한다.

```
docker login
# 일련의 작업 필요..

docker build -t oeeen/jenkins:v1 .
docker push oeeen/jenkins:v1
```

빌드를 실행하면 조금 시간이 걸리면서 아래 처럼 진행 된다..

![Docker Build](/assets/img/docker/docker_build.png)

docker hub에 push를 하면 또 약간의 시간이 걸리면서 아래처럼 된다!

![Docker Push](/assets/img/docker/docker_push.png)


이제 EC2 인스턴스에서 아래와 같은 명령으로 docker가 설치되어 있는 jenkins(oeeen/jenkins:v1)을 실행 할 수 있게 되었다!

```
# 도커 컨테이너 실행 명령어
sudo docker run 
    --name jenkins \
    -itd \
    -e JENKINS_USER=$(id -u) \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v $(pwd)/jenkins_home:/var/jenkins_home \
    -p 8000:8080 -p 50000:50000 \
    -u root \
    oeeen/jenkins:v1 # 도커 이미지 명
```

이제 docker가 설치된 jenkins 컨테이너가 준비되었다!!

이제 본격적으로 배포 과정을 진행할 수 있을 것 같다!