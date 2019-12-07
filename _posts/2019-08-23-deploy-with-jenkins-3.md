---
layout: single
title:  "나의 웹 어플리케이션을 배포해보자(with Jenkins, Docker) - 3"
date:   2019-08-23 21:00:59 +0900
classes: wide
categories: web
tags: jenkins
---

## 나의 웹 어플리케이션을 jenkins로 배포 해보자 - 3

이전 포스트에서는 젠킨스를 활용해서 git clone -> build -> 서버 띄우기 까지 진행했다.

이번 포스트에서는 github에서의 push event를 hooking 해서 push가 들어오면 자동으로 위에서 했던 배포 과정이 진행 되도록 설정한다.

이전 포스트에서 젠킨스 프로젝트가 github push를 polling 하고 있도록 설정 해두었다.

그렇기 때문에 이제 깃허브에서 설정만 해주면 된다.

프로젝트 레포지토리에 들어가서 setting에 들어간다.

![setting](/assets/img/jenkins/setting.png)

![setting2](/assets/img/jenkins/setting_page.png)

webhooks 를 누르고 Add webhook을 눌러준다.

![webhook_1](/assets/img/jenkins/webhook_page.png)

여기서 이제 Payload URL은 아래와 같이 설정한다.

`http://[EC2-IP]:8080/github-webhook/`

8080 포트는 jenkins의 기본 포트이다. + Content type은 application/json

![webhook_2](/assets/img/jenkins/webhook_page2.png)

이렇게 셋팅만 해주면, 해당 프로젝트의 레포에 푸시 이벤트가 들어오면 자동으로 빌드를 수행한다. (지금은 빌드후에 어플리케이션까지 띄우기 때문에 서버까지 뜬다.)

- 하지만 지금은 서버가 떠있는 상태를 체크하지 않고 그냥 도커 컨테이너를 띄우기 때문에 약간의 문제가 발생 할 수 있다.

이 문제는 앞으로 무중단 배포를 진행하면서 실행 되고 있는 포트가 아닌 다른 포트로 어플리케이션을 띄우는 방식으로 해결할 예정이다.

## 젠킨스에 access token 추가. (단순 푸시 이벤트 후킹에는 **필요 없다**)

젠킨스에 내 계정에 접근할 수 있는 권한을 주기 위해서 깃허브에 personal access token을 설정한다.

사실 푸시 이벤트만 hooking 해서 배포하기 때문에 이 과정은 필요 없다.

하지만 나중을 위해 적어둔다.

아래와 같은 순서로 진행한다.

![github1](/assets/img/jenkins/github_1.png)

Developer setting에 들어간다.

![github2](/assets/img/jenkins/github_2.png)

Personal Access Token을 누른다.

![github3](/assets/img/jenkins/github_3.png)

새로운 토큰을 발급 받는다. 아래 사진처럼 체크하고 저장한다.

![github4](/assets/img/jenkins/github_4.png)

새로운 토큰이 발급된다. (이 토큰은 이 페이지에 재 접속한다고 해도 다시 볼 수 없으니.. 따로 적어놓거나 해야할 것 같다.)

![github5](/assets/img/jenkins/github_5.png)

이제 젠킨스 관리 페이지에 들어간다.

Jenkins 관리 - 시스템 설정에 들어간다

![jenkins-setting](/assets/img/jenkins/jenkins_setting.png)

Github 관련 설정에서

Github server를 추가한다.

![github-servier](/assets/img/jenkins/github_server.png)

credential Add를 누른다

![add-credential](/assets/img/jenkins/add_credential.png)

Kind를 Secret text로 변경 하고 Secret 값을 아까 위에서 얻어놓았던 github personal access token을 입력해준다.

다 했으면 심심하니.. Test Connection도 눌러준다. 잘 연결이 될 것이다.

![setting-done](/assets/img/jenkins/setting_done.png)

이 과정은 프로젝트 관리 관련해서 github 계정이 필요할 때 사용 할 것 같은데, 현재 push 이벤트만 hooking 해서 빌드 자동화만 하기 때문에 필요 없는 과정이다.

다음 포스트는 nginx를 이용한 무중단 배포에 대해 쓸 예정이다.

- [4편](https://smjeon.dev/web/deploy-with-jenkins-4/)
