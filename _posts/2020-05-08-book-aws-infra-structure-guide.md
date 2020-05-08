---
layout: single
title:  "서비스 운영이 쉬워지는 AWS 인프라 구축 가이드"
date:   2020-05-08 10:30:59 +0900
categories: etc
classes: wide
tag: book aws
toc: true
toc_sticky: true
---

[서비스 운영이 쉬워지는 AWS 인프라 구축 가이드](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791158391317&orderClick=LEa&Kc=) 라는 책을 읽고 가볍게 정리한 글입니다.

## EC2를 생성하려면 꼭 알아야하는 개념

1. AMI (Amazon Machine Image)
2. 보안 그룹 (Security Group)
3. 키 페어 (Key pair)

### AMI (Amazon Machine Image)

AMI는 EC2 인스턴스의 기반이 되는 이미지다. 기본적으로 유명한 서버 OS들을 많이 제공하고, 개인이 커스텀해서 환경 구성 후 이미지로 만들어서 재사용할 수도 있다.

### 보안 그룹

보안을 위해 IP와 포트 번호를 이용해서 특정 IP, 포트로만 접속을 허용, 금지 할 수 있다. 생성된 보안 그룹으로 여러 서버에 적용할 수 있다.

### 키 페어

서버에 접속하기 위한 열쇠다. 공개 키 암호화 기법으로 서버에는 공개키, 사용자는 개인키를 들고 접속한다.

## ELB (Elastic Load Balancing)

AWS 서비스 중 로드밸런서의 역할을 하는 서비스다. L4 스위치 같은 장비를 직접 구입해서 사용하지 않고도 AWS의 ELB를 사용하여 로드밸런서의 기능을 사용할 수 있다. ELB의 대상 그룹에는 인스턴스나 Auto Scaling 그룹이 포함될 수 있다.

로드 밸런서는 관리하는 서버 중 정상적으로 동작하는 서버에만 요청을 전달해준다. 정상으로 동작하고 있는지 확인하기 위해 Health Check 과정을 거치게 된다. 로드 밸런서는 자기가 관리하는 서버들에게 주기적으로 정상 동작하고 있는지 물어본다. Health Check 주기는 변경할 수도 있고, 몇 번 연속 비정상 코드를 응답해야 비정상으로 변경할지도 설정할 수 있다. 반대로 비정상 -> 정상의 경우도 몇 번 연속 정상 응답을 해야 정상으로 바꿀지도 정할 수 있다.

ELB에서 health check 할 때 nginx같은 웹서버만 정상이어도 서버가 정상으로 판단한다면, nginx에서 해당 health check에 대한 응답을 처리하면 되고, nginx 뒷단의 was까지 정상이어야 서버가 정상으로 판단한다면, health check에 대한 응답을 was에서 처리하면 된다.

### NGINX의 HTTP Health Check

책에 있는 내용은 아니지만, NGINX의 HTTP Health Check에 대해 더 알아보면 아래와 같다.

#### Passive Health Checks

NGINX는 실패한 연결을 다시 시도한다. 만약 실패한 연결이 다시 붙지 않으면, NGINX는 해당 서버를 unavailable로 표시하고 해당 서버로 요청을 보내는 것을 일시적으로 멈춘다(다시 active 표시가 될 때까지).

upstream server 아래에 있는 서버들의 설정은 그 옆에 쓰면 된다.(아래 나와있는 것처럼)

fail_timeout – max_fails 를 넘어서 connection에 실패할 때, 서버를 unavailable로 표시해놓는 시간이다. (default는 10초)
max_fails – 연결 실패의 최대 숫자이다. 이 카운트를 넘어서면 해당 서버는 unavailable 표시가 된다. (default는 1회)

```conf
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
}
```

#### Active Health Check

NGINX Plus는 주기적으로 upstream server의 헬스 체크를 할 수 있다.(각 서버로 헬스체크 요청을 보내고 맞는 응답이 오는지 검증하는 방식) 아래와 같은 방식으로 사용하면 된다.

```conf
server {
    location / {
        proxy_pass http://backend;
        health_check;
    }
}
```

위 예시는 모든 요청을 upstream group인 backend로 보낸다.(health check on) default 설정으로 매 5초마다 NGINX Plus는 "/" 경로로 요청을 보낸다.(backend group안에 있는 서버들) 만약 통신 에러나, 타임아웃이 발생하면(200~399 외의 응답이 오면) health check fail이다. 그러면 server는 unhealthy 표시가 되고, 해당 서버로 요청을 더이상 보내지 않는다.(다음 health check를 통과할 때까지)

아래 설정 처럼 특정 포트로 들어오는 요청만 health check를 할 수도 있다.

```conf
server {
    location / {
        proxy_pass   http://backend;
        health_check port=8080;
    }
}
```

또 `health_check uri=/some/path`와 같은 방식으로 특정 경로로 들어오는 요청만 헬스 체크를 할 수도 있다.

그 외에 Custom Condition을 추가할 수도 있다. health_check block 에서 다음과 같이 사용할 수 있다.

```conf
http {
    #...
    match server_ok {
        status 200-399;
        body !~ "maintenance mode";
    }
    server {
        #...
        location / {
            proxy_pass http://backend;
            health_check match=server_ok;
        }
    }
}
```

위에서는 status code가 200-399 사이인지, 그리고 body에 `maintenace mode`라는게 포함 되어 있지 않다면 server_ok가 통과한다.(health check 성공)

match에서는 status code, 헤더필드, 응답의 body를 체크할 수 있다.

## 도메인

클라이언트가 요청을 보내는 서버마다 고유한 IP주소를 가지고 있지만, IP주소는 알다시피 123.123.123.123 이런식으로 숫자로 되어 있어 우리 같은 개발자들에겐 친숙하지만, 일반인들에겐 친숙하지 않다...

그래서 도메인 주소, 예를 들어 `https://smjeon.dev` 와 같은 쉽게 외울 수 있는 도메인 주소가 필요하다. 또한 도메인 주소가 없다면, 서버의 IP주소가 변경될 때 기존 사용자들은 접속할 수 없게 된다.

도메인 주소의 작동방식은 아래와 같다.

![Domain](/assets/img/aws/domain.png)

DNS 서버는 도메인과 그 도메인에 연결된 IP 주소들을 관리하는 서버이다. 나 같은 경우에는 GoDaddy를 이용하여 두 개의 도메인을 소유하고 있다.

![GoDaddy](/assets/img/aws/godaddy.png)

## 중단 배포와 무중단 배포

단순히 생각하면 무중단 배포가 무조건적으로 사용자 입장에서는 좋다.

그러면 왜 중단 배포를 하게될까?

무중단 배포를 하기에는 너무 많은 비용이 발생하는 경우가 있다. 데이터베이스 스키마가 변경되거나, 특정 기능의 핵심로직이 변경되는 경우 등, 구 버전과 신 버전이 동시에 서비스 되면 안되는 경우에는 중단 배포를 하거나 다른 처리를 해야한다.

A, B 기능을 서비스하는 어플리케이션에 A, B와 연관이 없는 C기능을 배포한다면 무중단 배포를 해도 문제가 없다. 그러나 B에서 사용하는 테이블이 변경된다거나 하면 구버전과 신버전 사이에 테이블이 문제가 된다. 데이터베이스의 정합성이 깨지게 되는 것이다.

[무중단 배포](https://smjeon.dev/etc/deployment/)에 관해서는 이전에 TIL로 정리해둔 글이 있다. 이를 참고하면 좋을 것 같다.

## AWS IAM (Identity and Access Management)

AWS는 보통 회사당 하나의 계정을 사용한다. 회사 내 AWS를 사용하는 사람들에게 모두 같은 권한을 줄 수는 없기 때문에, 최고 관리자가 root 계정을 관리하고 그 외의 사용자는 별도로 계정을 발급받아 제한된 권한으로 AWS를 이용한다.

사용자별로 AWS에서 제공하는 서비스들, 자원 등에 대해 권한을 지정해서 관리할 수 있게 해준다. IAM의 용어는 아래와 같이 있다.

이름 | 설명
--- | ---
권한 | 어떤 서비스나 자원에 어떤 작업을 할 수 있는지 명시해두는 규칙
정책 | 권한들의 모음, 사용자나 그룹에 권한을 직접 적용할 수 없고 권한들로 만든 정책을 적용해야함
사용자 | AWS의 서비스나 자원을 사용하는 객체
그룹 | 여러 사용자에게 공통으로 권한을 부여할 수 있는 묶음 사용자 라고 생각하면 됨
역할 | 사용자와 비슷하지만, 서비스나 다른 AWS계정의 사용자라는 점에서 차이가 있다. 예를 들어 EC2 인스턴스에서 S3에서 파일을 읽어오려면 S3의 파일 Read 권한으로 정책을 만들어 해당 정책으로 역할을 만들어 EC2 인스턴스에 지정해야한다.
인스턴스 프로파일 | 사용자 -> 사람 구분, 그 사람에게 권한을 준다. 인스턴스 프로파일 -> EC2 인스턴스를 구분, 그 인스턴스에 권한을 준다.

## AWS CodeDeploy

Jenkins 같은 CI 도구들 보다 좋은 점은 더 손쉽게 AWS의 서비스들과 연동할 수 있다는 점이다.

동작 방식은 아래와 같다.

1. 애플리케이션의 소스코드 최상단에 AppSpec.yml 이라는 파일을 추가한다.(배포 명세 같은 것)
2. CodeDeploy에 특정 버전 배포를 요청
3. CodeDeploy는 배포를 진행할 EC2 인스턴스들에 설치되어 있는 CodeDeploy agent들에게 요청받은 버전을 배포 요청
4. CodeDeploy Agent들을 요청 받은 버전의 프로젝트를 Github과 같은 코드 저장소에서 내려받고, AppSepc.yml에 나와있는 명세대로 배포를 진행한다.
5. CodeDeploy Agent들은 배포 진행 후 CodeDeploy에게 알려준다.

## Secret 관리

비밀 값은 유출될 경우 서비스의 안전에 위협이 되는 값들이다. 예를 들어 내가 했던 프로젝트에서 kakao.yml 이나 github.yml 등의 인증을 위한 Client Secret 같은 값이 포함된다. 나도 그랬듯 .gitignore 파일에 해당 파일이 커밋되지 않도록 막아둔 후에, 따로 배포 관리를 통해 이 secret들을 관리해야한다. 나의 경우에는 Jenkins 환경 변수로 추가해서 관리하기도 했고, Github Actions를 사용할 때는 github repository의 secrets를 이용하는 방법을 사용했었다.

이런 Secret을 관리하는 방법에는 여러가지가 있다.

### Blackbox

Secret 을 담고있는 파일을 암호화해서 VCS에 올리는 방법이다. 많이 사용되는 툴로 Blackbox가 있다. GPG(GNU Privacy Guard)라는 암호화 프로그램을 이용해 비밀 값들이 담긴 파일을 암호화한다.

### Vault

애플리케이션은 HTTP API를 통해 Vault에 인증을 요청한다.(로그인 개념) 인증이 완료되면, Vault API를 이용할 수 있다. 그러면 해당 API를 통해 Secret을 저장하거나 읽어오면 된다.

### AWS Secrets manager

AWS에서 제공하는 Secret 관리 서비스다. Secret을 AWS Secrets manager에 저장해두고 애플리케이션에서 AWS API를 호출하여 사용하는 방식이다.

1. 관리자가 Secrets Manager에 Secret을 생성, 그 안에 key, value 형식의 Secret을 등록한다.
2. 해당 비밀 값들은 AWS KMS(Key Management System)를 통해 암호화된다.
3. 해당 Secret을 사용하는 서버에서 AWS CLI나 SDK를 이용하여 Secrets Manager에 Secret을 요청한다.
4. 해당 IAM이 권한이 있다고 판단하면 Secrets Manager가 Secret을 복호화하여 응답으로 준다.

## 모니터링

모니터링의 목적은 안정적인 서비스 운영이다. 장애 발생 후 복구까지 시간이 오래걸린다면, 서비스의 사용자들이 신뢰를 잃고 떠날 수 있다. 큰 장애가 발생하기 이전에 미리 징후를 찾아내서 최대한 예방해야하고, 장애가 발생하더라도 바로 원인을 파악하고 고쳐야한다.

### AWS CloudWatch

AWS에서 제공하는 AWS 내 자원과 애플리케이션에 대한 모니터링, 관리 서비스다. 서비스를 사용하며 발생하는 모든 로그와 정보들을 한눈에 볼 수 있도록 시각화 하는 모니터링 서비스다.

CloudWatch의 지표들은 "언제" 어떤 "항목"의 "값"이 무엇이었는지를 기록한 값이다. 기본적으로 AWS에서 제공하는 지표 외에도 사용자가 지정한 지표를 직접 기록할 수도 있다.

**실제로 해보고 어떤 식으로 사용하는지 알아야 도움이 될 듯하다.**

## Elastic Beanstalk

개발자들이 최대한 빨리 애플리케이션을 시작할 수 있는 환경을 구성하고 개발에 더욱 집중할 수 있게 해주는 서비스다.

Elastic Beanstalk를 사용하면 서버 구성에 시간을 쏟을 필요 없이 현재 애플리케이션의 언어에 맞는 환경만 선택하고 애플리케이션 소스코드를 업로드하면, 기타 환경 구성이 완료된 서버를 생성하고, 애플리케이션 배포까지 자동으로 해주고, Auto Scaling까지 자동으로 진행한다.

### 장점

1. 빠른 서버 환경 구축
2. 서버 운영 지식이 없더라도, 다중 서버, 보안 그룹이 구성되어 있는 서버환경을 구축할 수 있다.
3. 인프라에 신경을 덜 써도 되기 때문에 개발에 더욱 집중할 수 있다.
4. Elastic Beanstalk을 사용하더라도 추가요금을 내지는 않는다.
5. 나만의 환경을 구축하는 것도 가능하다.

### 단점

1. 세부 설정에 대한 유연성이 떨어진다.
2. 기본 제공 설정을 수정할 수 있긴 하지만, 수정하기가 어려운편
3. 첫 배포가 아니라 이미 배포된 상태라면 Elastic Beanstalk 환경에 맞추는 게 더 힘들 수도 있다.
4. 한 마디로 말해서 기존 설정을 따라가야 하기 때문에, 내가 커스텀한 환경과 안 맞는게 문제다.

## DevOps

Development와 Operation의 합성어로, 개발과 운영을 하나로 합쳐서 일하는 철학, 도구, 환경, 문화 등의 조합

개발부터 배포까지 모든 단계에 자동화와 모니터링을 도입해서 더 짧은 개발 주기, 더 많은 배포 빈도, 안정적인 소프트웨어를 배포하자는 목표를 갖고 있다.

서버 구성, 배포, 테스트 등의 반복적이고 단순 작업을 최대한 자동화하여 배포에 들어가는 인적 비용을 최소화 하자는 것부터 시작한다. 그렇기 위해서는 일단 테스트 코드를 포함한 자동화 환경이 잘 갖춰져 있어야 한다.

### CI (Continuous Integration)

개발자들이 코드를 메인 브랜치에 머지하는 간격을 최대한 짧게 하여 버그를 빠르게 찾을 수 있게 하는 것이다. 짧은 주기의 커밋을 통해 코드 머지시 발생하는 conflict를 최소화 할 수 있고, 자동화된 테스트 환경을 통해 자신이 개발한 기능의 문제가 발생하는 지에 대한 피드백을 빠르게 받을 수 있다.

대신 이를 위해서는 반드시 제품의 품질을 보장할 수 있다고 믿을 수 있는 테스트들이 작성되어 있어야 한다.

### CD (Continuous Deployment)

CI pipeline에서 빌드 및 유닛 테스트까지 진행되어 넘어온 코드를 운영 서버에 배포하기 전, 스테이지 서버에 배포하고 해당 서버에서 UI 테스트, 인수테스트 등의 다양한 테스트를 자동으로 진행한다. 해당 테스트를 모두 통과 했다면 운영 서버에 배포하기 위한 준비를 한다.

운영 서버에 사람이 수동으로 할 수도 있고, 자동으로 운영 서버까지 배포될 수도 있다. 개발자가 기능을 개발하면 빠르게 운영 서버까지 배포가 일어나고 이를 통해 사용자에게 최신 버전의 서비스를 최대한 빠르게 전달할 수 있다는 장점이 있다.

이 과정도 또한 반드시 테스트가 잘 작성되어 있어야 한다.

## 정리

책 내용도 매우 좋고, 나 같은 초보자들이 읽기에도 쉬운 책인 듯 하다. 인프라에 대해 전혀 아무런 지식이 없는 상태에서 읽을 경우의 난도는 잘 모르겠으나, 나 또한 지식이 그렇게 많은 상태에서 읽은 것은 아니기에 초보자들이 쉽게 다가갈 수 있는 책인 것 같다.

매 챕터마다 실습 부분이 있는데, 실습을 해보며 따라해본다면 더 큰 도움이 될 것 같다.
