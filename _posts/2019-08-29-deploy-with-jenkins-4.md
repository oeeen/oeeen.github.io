---
layout: single
title:  "나의 웹 어플리케이션을 배포해보자(with Jenkins, Docker) - 4"
date:   2019-08-29 21:00:59 +0900
classes: wide
categories: web
tags: jenkins docker nginx
---

# 나의 웹 어플리케이션을 jenkins로 배포 해보자 - 4

이번에는 nginx를 이용해서 무중단 배포를 흉내 낼 예정이다. 

nginx의 proxy_pass를 배포 시점마다 바꿔주고 nginx reload를 하는 것이 좋은 방식인 것 같지만! 현재 상황에서는 굉장한 어려움에 처할 수 있으므로 패스한다.

그래서 지금은 nginx의 upstream server를 이용해서 무중단 배포 인 척(?)을 하려고 한다.

무중단 배포 흉내내기는 단계 별로 진행 된다.

## 1. docker, docker-compose가 깔려있는 jenkins 이미지
일단 이전까지 사용했던 jenkins 이미지에 docker-compose가 추가로 설치된 이미지가 필요하다.

그래서 이전에 만들었던 oeeen/jenkins:v2에서 docker-compose 도 설치 한 image를 만드는 Dockerfile을 만들었다.

도커 compose 설치법: [docker-compose-install](https://docs.docker.com/compose/install/)

**Dockerfile**

```
#Dockerfile - docker, docker-compose based on jenkins image
FROM jenkins/jenkins:lts
USER root
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
RUN sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN sudo chmod +x /usr/local/bin/docker-compose
RUN usermod -a -G docker jenkins
USER jenkins
```

`sudo docker build -t oeeen/jenkins:v3 .` 를 실행해서 이미지를 빌드하고, `sudo docker push oeeen/jenkins:v3` 로 docker hub에 push 한다. 

그리고 다시 jenkins를 실행 시킨다!

**젠킨스 실행**

```
sudo docker run \
    --name jenkins \
    -itd \
    -e JENKINS_USER=$(id -u) \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v $(pwd)/jenkins_home:/var/jenkins_home \
    -p 8080:8080 -p 50000:50000 \
    -u root \
    oeeen/jenkins:v3
```

## 2. 빌드 자동화 + a

이번에는 8080포트를 그대로 사용했다. 그 이유는 기존에 사용했던 8000번 포트는 지금 연습용 ec2의 보안 그룹에서는 특정 IP에만 열려있기 때문에, github push event를 hooking하지 못했기 때문이다. 그래서 모든 ip에 대해 열려있는 8080포트를 jenkins용으로 사용하기로 했다!

앞의 포스팅에서 했던 것처럼 jenkins의 프로젝트 셋팅을 해준다. 그리고 github repository의 webhook 셋팅도 8080포트로 해주면, 자연스럽게 push event에 걸려서 build가 진행 될 것이다.

이제 할 일은 빌드는 자동으로 되면서 현재 떠있는 어플리케이션의 포트를 확인하고, 쉬고 있는 포트로 서버를 띄워주고 기존에 떠있던 서버(구 서버)는 종료하는 것이다.

일단 jenkins project의 build 부분을 아래 처럼 변경 했다.

![build_auto](/assets/img/jenkins/build_auto.png)

```
./gradlew clean build --info
chmod +x ./deploy.sh
./deploy.sh
```

deploy.sh는 project 디렉토리에 담아두고 같이 푸시했다.

deploy.sh 파일 내용은 다음과 같다.

```
#!/bin/bash

DOCKER_APP_NAME=sunbookApp
DOCKER_DB_NAME=mydb

EXIST_BLUE=$(docker-compose -p ${DOCKER_APP_NAME}-blue -f docker-compose.blue.yml ps | grep Up)

EXIST_DB=$(docker-compose -p ${DOCKER_DB_NAME} -f docker-compose.db.yml ps | grep Up)

if [ -z "$EXIST_DB" ]; then
    echo "DB setting"
    docker-compose -p ${DOCKER_DB_NAME} -f docker-compose.db.yml up -d
fi

if [ -z "$EXIST_BLUE" ]; then
    echo "blue up"
    docker-compose -p ${DOCKER_APP_NAME}-blue -f docker-compose.blue.yml up -d

    sleep 10

    docker-compose -p ${DOCKER_APP_NAME}-green -f docker-compose.green.yml down
else
    echo "green up"
    docker-compose -p ${DOCKER_APP_NAME}-green -f docker-compose.green.yml up -d

    sleep 10

    docker-compose -p ${DOCKER_APP_NAME}-blue -f docker-compose.blue.yml down
fi
```

간단하게 설명하면,

1. DB가 존재하지 않으면 DB compose로 db 컨테이너를 띄운다. db가 있으면 그냥 넘어간다.
2. blue라는 이름의 서버가 떠있으면 green이라는 이름의 서버를 띄운다. 
3. 10초 기다린다.
4. 기존에 떠있던 blue라는 이름의 서버를 내린다.
5. 반대도 동일하다. (green이 있으면 blue를 띄우고 green을 내린다.)

이 스크립트가 실행되기 위해서는 이제 docker-compose.blue.yml, docker-compose.green.yml, docker-compose.db.yml 파일들이 필요하다.

각 파일들을 살펴보면

**docker-compose.blue.yml**

```
version: '3'

services:
  sunbookApp:
    image: sunbook
    volumes:
      - /home/ubuntu/jenkins_home/workspace/sunbook/build/libs:/usr/src/app
    ports:
      - "8081:8080"
networks:
  default:
    external:
      name: mydb
```

**docker-compose.green.yml**

```
version: '3'

services:
  sunbookApp:
    image: sunbook
    volumes:
      - /home/ubuntu/jenkins_home/workspace/sunbook/build/libs:/usr/src/app
    ports:
      - "8082:8080"
networks:
  default:
    external:
      name: mydb
```

**docker-compose.db.yml**

```
version: '3'

networks:
  default:
    external:
      name: mydb
services:
  mydb:
    image: mysql:5.7
    volumes:
      - /home/ubuntu/sql/:/docker-entrypoint-initdb.d
    environment:
      - MYSQL_DATABASE=sunbook
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
      - MYSQL_ROOT_PASSWORD=root
    command:
      ['mysqld', '--character-set-server=utf8mb4', '--collation-server=utf8mb4_unicode_ci']
```

이렇게 구성했다.

내용을 조금 살펴보면 blue는 8081포트로, green은 8082포트로 열어둔다. 그리고 sunbook이라는 이미지가 필요하다.

sunbook 이미지는 다음의 Dockerfile로 만들었다.

**sunbook 이미지용 Dockerfile**

```
FROM openjdk:8

MAINTAINER smjeon <oeeen3@gmail.com>

VOLUME ~/deploy/sunbook

COPY ./start-server.sh /usr/local/bin
RUN ln -s /usr/local/bin/start-server.sh ~/start-server.sh
CMD ["start-server.sh"]
```

먼저 `/home/ubuntu/deploy/sunbook` 디렉토리를 만들어준다. 그리고 `~/deploy` 디렉토리 내부에 start-server.sh라는 파일이 필요한데, 이 파일의 내용은 다음과 같다.

**start-server.sh**

```
#!/bin/bash

java -jar -Dspring.profiles.active=deploy /usr/src/app/sunbook-0.0.1-SNAPSHOT.jar
```

이 파일에서 실제로 서버를 띄운다.

위의 Dockerfile 을 `sudo docker build -t sunbook .` 명령으로 빌드 한다. 

Database는 초기 셋팅 그대로 만든다. 

대신 network는 blue, green 서버와 통신하기 위해서 하나의 네트워크로 묶어주었다.

여기서 mydb라는 network가 필요하기 때문에 docker host에서 `sudo docker network create mydb` 명령어로 네트워크를 하나 만들어 준다.

이 과정까지 끝나면 프로젝트 폴더의 상태는 다음과 같이 된다.

![tree](/assets/img/jenkins/project_architecture.png)

그리고 db를 mydb라는 이름으로 띄워 놓았기 때문에 DB 정보는 `jdbc:mysql://mydb:3306/sunbook?serverTimezone=UTC&allowPublicKeyRetrieval=true&useSSL=false` 로 셋팅해두었다.

이렇게 까지 셋팅을 하면 빌드가 될 때마다 8081, 8082 포트를 왔다 갔다 하면서 서버가 켜질 것이다.

## 3. nginx 셋팅하기

이제 8081, 8082 포트로 번갈아가며 실행이 되긴 할텐데.. ec2 기본 보안 그룹에 막혀있어서.. 8081, 8082포트를 막 들어갈 수가 없다..

그래서 nginx의 proxy_pass와 load balancing 기능만을 활용하기로 했다.

일단 docker host에 nginx를 설치 한다. `sudo apt install nginx`

설치가 완료되면 `$cd /etc/nginx`로 이동한다.

nginx 폴더 내부에는 아래 그림처럼 되어있다.

![nginx_directory](/assets/img/jenkins/nginx_directory.png)

이 디렉토리에서 nginx.conf 파일을 바꾼다.

**nginx.conf**

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;


events {
	worker_connections 768;
	# multi_accept on;
}

http {
	upstream sunbook {
	    server localhost:8081;
	    server localhost:8082;
	}

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	server {
    	    listen 80;

    	    access_log /var/log/nginx/ec2-nginx.log;
    	    error_log /var/log/nginx/ec2-nginx-error.log;

    	    proxy_max_temp_file_size 0;
    	    proxy_buffering off;

            client_max_body_size 100M;

    	    root /usr/src/app/public;

    	    location / {
	        proxy_pass http://sunbook;
	    	proxy_set_header X-Real-IP $remote_addr;
	    	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    	proxy_set_header Host $http_host;
            }
        }

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	#include /etc/nginx/conf.d/*.conf;
	#include /etc/nginx/sites-enabled/*;
}
```

불 필요한 내용이 있을 수 있지만 nginx에 대해 정확하게 알지 못하기 때문에 쉽사리 지울 수 없었다.

기존 내용에서 변경된 내용만 본다면, 아래와 같다.

```
upstream sunbook {
    server localhost:8081;
    server localhost:8082;
}

location / {
    proxy_pass http://sunbook;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
}
```

설명을 좀 붙이자면 upstream에 sunbook이라는 이름으로 추가해서 localhost:8081, localhost:8082의 경로를 reverse proxy로 둔다.

원래는 load balancing을 위해 두는 것으로 알고 있는데, 나는 무중단 배포 흉내내기를 위해 사용하기로 했다.

그리고 80포트의 /로 들어오는 모든 요청을 http://sunbook 으로 돌린다. 

http://[ec2 ip]:80으로 들어오는 모든 요청은 http://[ec2 ip]:8081, http://[ec2 ip]:8082로 upstream에 설정한 설정값에 따라 load balancing 된다.

하지만 우리는 두 포트 중 하나만 살아 있을 예정이므로.. http://[ec2 ip]:80로 들어오는 요청이 두 포트 중 살아있는 포트로 요청이 갈 것이다.

이렇게 nginx.conf 파일을 바꾸고 `sudo service nginx reload`로 nginx를 다시 로드한다.

그러면 이제 http://[ec2 ip]:80으로 접속을 해보면 현재 떠있는 서버로 요청이 가기 때문에 우리의 어플리케이션을 다시 볼 수 있을 것이다!

이 과정까지 모두 끝내면 github repository에 push가 들어오면, 새롭게 빌드를 하고 새로 빌드된 서버가 뜰 때까지는 기존 서버가 살아있는 것을 볼 수 있을 것이다!

jojoldu님 블로그에서 하신 방식 처럼 proxy_pass에 들어가는 값을 빌드할 때마다 변경 해주고 싶었지만 현 상황에서 불가능 했던 이유가 있다.

* 도커 컨테이너 내부에서 도커 호스트의 /etc/nginx/nginx.conf를 수정해야 한다. (가능하긴 하다)
* 도커 컨테이너 내부에서 도커 호스트의 nginx service를 reload 해야한다. `service nginx reload` 명령어가 필요하다. (방법을 아직 모르겠다.)

그래서 아쉽지만 현재 상태로 구현하게 되었다.

이번 내용 자체가 새로운 기술을 많이 배우게 되어서 깊이 있는 공부를 하지는 못했지만, 뭔가 새로운 index를 하나 잡은 것 같다.

---

참고자료: 
* [jojoldu님 블로그](https://jojoldu.tistory.com/267)
* [subicura님 블로그](https://subicura.com/2016/06/07/zero-downtime-docker-deployment.html)
* [jeff0720님 블로그](https://velog.io/@jeff0720/Travis-CI-AWS-CodeDeploy-Docker-%EB%A1%9C-%EB%B0%B0%ED%8F%AC-%EC%9E%90%EB%8F%99%ED%99%94-%EB%B0%8F-%EB%AC%B4%EC%A4%91%EB%8B%A8-%EB%B0%B0%ED%8F%AC-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-2)
* [도커 컴포즈를 활용하여 완벽한 개발 환경 구성하기](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#%EB%8F%84%EC%BB%A4-%EC%BB%B4%ED%8F%AC%EC%A6%88%EB%A1%9C-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0) 