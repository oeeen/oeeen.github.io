---
layout: single
title:  "나의 웹 어플리케이션을 배포해보자(with Jenkins, Docker) - 2"
date:   2019-08-21 21:00:59 +0900
classes: wide
categories: etc
tags: jenkins
---

# 나의 웹 어플리케이션을 jenkins로 배포 해보자 - 2

## Jenkins 설치

지난 번에 docker가 설치된 jenkins 컨테이너 이미지까지 준비를 했다.

```
sudo docker run \
    --name jenkins \
    -itd \
    -e JENKINS_USER=$(id -u) \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v $(pwd)/jenkins_home:/var/jenkins_home \
    -p 8000:8080 -p 50000:50000 \
    -u root \
    oeeen/jenkins:v2
```

![jenkins-run](/assets/img/jenkins/jenkins_run.png)

로 실행 시킨 후 웹 브라우저를 실행시켜 `[EC2 IP]:8000` 으로 접속한다.

다음과 같은 화면이 나온다.

![jenkins-initial](/assets/img/jenkins/jenkins_initial.png)

`sudo docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword`
로 초기 비밀번호를 얻는다.

그 결과로 나온 초기 비밀번호를 복사해서 administrator password에 붙여 넣는다.

화면이 넘어가면

![jenkins-select](/assets/img/jenkins/jenkins_select.png)

이 화면에서는 젠킨스가 추천해주는 플러그인들을 설치 하기로 한다.


![jenkins-install](/assets/img/jenkins/jenkins_installing.png)

위와 같은 화면이 나오면서 모두 설치가 완료 될 때까지 기다린다.

![jenkins-create-user](/assets/img/jenkins/jenkins_create_user.png)
설치가 완료 된 후 계정을 만들고 Save 한다. 다음번에 로그인할 땐 이 계정으로 로그인 하면 된다.

![jenkins-start](/assets/img/jenkins/jenkins_start.png)

![jenkins-start2](/assets/img/jenkins/jenkins_start2.png)

위와 같은 화면들을 거치면서 쭉쭉 진행하면 된다.




## Jenkins Project 생성

아래 순서로 따라가면 된다.

![project-start1](/assets/img/jenkins/jenkins_project1.png)

새로운 item 클릭

![project-start2](/assets/img/jenkins/jenkins_project2.png)

일단 우리는 Freestyle project를 고른다.

![project-start3](/assets/img/jenkins/jenkins_project3.png)

따로 체크할 것은 없다.

![project-start4](/assets/img/jenkins/jenkins_project4.png)

우리는 Git Repository를 기준으로 배포를 진행할 것이기 때문에 작성 해준다.
build를 진행할 브랜치 명도 적어준다.

현재는 pull 해서 test 후 build만 할 것이기 때문에 따로 Credentials은 필요 없는 것으로 알고 있다.

![project-start5](/assets/img/jenkins/jenkins_project5.png)

빌드 유발은 Github의 push event를 hook해서 자동 빌드가 되도록 할 예정이다.

![project-start6](/assets/img/jenkins/execute_shell.png)

Github에서 프로젝트를 pull 한 이후에 진행할 것을 shell로 진행 할 것이기 때문에 Execute Shell을 고른다.

![project-start7](/assets/img/jenkins/shell_contents.png)

```
./gradlew clean build
docker build --tag oeeen/sunbook:dep .
docker run --name sunbook -d -p 8080:8080 --link mydb:sunbook oeeen/sunbook:dep
```

> `./gradlew clean build` gradle로 clean 후에 build를 진행한다.

> `docker build --tag oeeen/sunbook:dep .` 프로젝트에 Dockerfile을 넣어 놓았기 때문에 Dockerfile 빌드를 시작한다.

> `docker run --name sunbook -d -p 8080:8080 --link mydb:sunbook oeeen/sunbook:dep`

--name sunbook --> 이름은 sunbook으로 하고

-d --> deamon으로 실행시키면서, 

-p 8080:8080 --> 컨테이너의 8080포트를 서버의 8080포트에 매핑시키고 

--link mydb:sunbook --> mydb라는 컨테이너를 link하는데 sunbook container 내부에서는 sunbook으로 연결되게 해놓는데

oeeen/sunbook:dep --> 실행할 컨테이너의 이름은 oeeen/sunbook:dep이다.

프로젝트 내부의 Dockerfile은 아래와 같다.

```
# Dockerfile
FROM openjdk:8

COPY ./build/libs/sunbook-0.0.1-SNAPSHOT.jar /usr/src/app/

WORKDIR /usr/src/app

CMD java -jar -Dspring.profiles.active=deploy /usr/src/app/sunbook-0.0.1-SNAPSHOT.jar
```


mydb라는 컨테이너는 아래처럼 실행시켰다.

```
sudo docker run -p 3306:3306 \
    -v /home/ubuntu/sql/:/docker-entrypoint-initdb.d \
    -e MYSQL_DATABASE=sunbook 
    -e MYSQL_ALLOW_EMPTY_PASSWORD=yes \
    -e MYSQL_ROOT_PASSWORD=root \
    -itd \
    --name mydb mysql:5.7 \
    --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

ec2의 home directory의 sql 폴더 아래에 있는 .sql을 초기 sql문으로 실행 한다.


home directory의 sql 폴더 아래에 sql.sql 이라는 파일을 추가해줬다. 

```
# sql.sql
create user '[userName]'@'%' identified by '[password]';
grant all privileges on *.* to '[userName]'@'%';
flush privileges;
```

[userName]을 가진 user를 추가 해주고, password를 설정 해주고 해당 유저에게 모든 권한을 주었다. 원래는 권한을 주는 것도 적절한 선에서 주어야 하는데 지금은 구동시키는 것이 주 목적이기 때문에 모든 권한을 줬다.


![build-now](/assets/img/jenkins/build_now.png)

그렇게 한 후 Build Now를 누른다.


![front-build](/assets/img/jenkins/project_sunbook.png)

아래 Build History에 나오는 숫자를 눌러서 빌드 상태를 볼 수 있다.


![build1](/assets/img/jenkins/build1.png)

여기서 나는 Console Output을 눌러서 로그들을 보았다.

![build2](/assets/img/jenkins/build2.png)

이런 로그들이 나온다...

![build3](/assets/img/jenkins/build3.png)


결국 Finished: SUCCESS 했다.

이제 [서버IP]:8080 으로 접속해보면 우리가 만든 application이 떠있는 것을 알 수 있다!


다음으로 할 일은 Github push가 들어오면 그것으로 build triggering 되도록 변경 해야 한다.

그리고 이미 서버가 떠있는 상태에서 새로 Github에 push가 들어올 때 새롭게 빌드를 하고 서버가 죽지 않은 상태에서 새로운 커밋 내용들이 반영되도록 **무중단 배포**를 할 예정이다.