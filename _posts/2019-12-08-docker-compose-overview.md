---
layout: single
title:  "Overview of Docker Compose"
date:   2019-12-08 12:50:59 +0900
classes: wide
categories: etc
tags: docker
---

## Docker 공식문서 번역 - [Overview of Docker Compose](https://docs.docker.com/compose/)

Compose는 멀티 컨테이너 도커 어플리케이션을 정의하고 실행하기 위한 도구다. Compose에서는 어플리케이션 서비스를 설정하기 위해 yaml 파일을 사용한다. 그리고 단일 커맨드로 설정파일(yaml파일)에 있는 모든 서비스를 만들고 실행할 수 있다.

Compose를 사용하는 것은 기본적으로 3단계의 과정이 있다.

1. 어플리케이션 환경을 어디서든 재사용 할 수 있도록 도커파일로 정의한다.
2. 어플리케이션을 만드는 서비스들을 각각 독립된 환경에서 같이 실행할 수 있도록 docker-compose.yml 파일 내에 정의한다.
3. docker-compose up을 실행하고 전체 어플리케이션을 실행시킨다.

docker-compose.yml 파일 예시는 아래와 같다.

```yml
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

Compose file에 대한 더 많은 정보를 위해서는 [compose file reference](https://docs.docker.com/compose/compose-file/)를 참고해라.

Compose는 내 어플리케이션의 전체 라이프사이클을 관리하기 위한 커맨드들이 있다.

- 서비스 시작, 정지, 리빌드
- 실행 중인 서비스들의 상태 보기
- 실행 중인 서비스의 로그 스트림
- 서비스에 일회성 명령 수행

## Features

Compose를 효과적으로 사용하기 위한 특징은 아래와 같다.

- Single Host에서 격리된 환경의 여러 컨테이너
- 컨테이너가 만들어질 때 볼륨 데이터 예약
- 바뀔 때만 컨테이너 재생성
- 변수와 환경 간의 구성 이동(Variables and moving a composition between environments)(?)

### Single Host에서 격리된 환경의 여러 컨테이너 (Multiple isolated environments on a single host)

컴포즈는 서로 환경을 독립시키기 위해 프로젝트 이름을 사용한다. 이 프로젝트 이름을 몇개의 다른 컨텍스트에서 사용할 수 있다.

- 개발 호스트에서 단일 환경의 여러 복사본을 실행시키기를 원할 때 (예를 들어, 프로젝트의 여러 feature branch의 안정적인 복사본을 실행시키기를 원할 때)
- CI server에서, 빌드 간에 서로 간섭하지 않도록 하기 위해서 프로젝트 이름을 고유한 빌드 넘버로 정할 수 있다.
- 공유 호스트 또는 개발 호스트에서, 동일한 이름을 사용할 수 있는 다른 프로젝트가 서로 간섭하지 않도록 할 때

기본 프로젝트 이름은 프로젝트 디렉토리의 기본 이름이다. -p 옵션이나 COMPOSE_PROJECT_NAME 환경변수를 사용해서 커스텀 프로젝트 이름을 정할 수 있다.

### 컨테이너가 만들어질 때 볼륨 데이터 예약 (Preserve volume data when containers are created)

컴포즈는 내 서비스가 사용하는 모든 볼륨을 예약한다. docker-compose up이 실행될 때 이전에 실행했던 컨테이너가 있다면, 이전 컨테이너로부터 새로운 컨테이너로 볼륨을 복사한다. 이 과정은 볼륨안에 있는 어떤 데이터들도 잃어버리지 않는다는 것을 보장한다.

만약 docker-compose를 윈도우 머신에서 사용한다면, 필요한 조건에 따라 필요한 환경변수를 조정해야한다.

### 바뀔 때만 컨테이너 재생성 (Only recreate containers that have changed)

컴포즈는 컨테이너를 만들기 위해 사용되는 설정을 캐싱한다. 만약 서비스가 바뀌지 않은 채로 다시 시작한다면 컴포즈는 기존에 존재하는 컨테이너를 재사용한다. 컨테이너를 재사용한다는 것은 환경을 매우 빠르게 변경할 수 있다는 것을 의미한다.

### 변수와 환경 간의 구성 이동 (Variables and moving a composition between environments)

컴포즈는 컴포즈 파일 내에 변수들을 지원한다. 이런 변수들을 다른 사용자나 다른 환경을 위해 설정파일(compose file)을 커스터마이징 하기 위해 사용할 수 있다.

```yml
db:
  image: "postgres:${POSTGRES_VERSION}"
```

이런 식으로 작성하고, 쉘 커맨드로 POSTGRES_VERSION=9.3 이런식으로 변수를 넘길 수 있다. 더 자세한 정보는 [compose-file 문서](https://docs.docker.com/compose/compose-file/#variable-substitution)를 참고하면 된다.

Compose file을 `extends` field를 사용거나 여러개의 컴포즈 파일을 만들어서 extend 할 수 있다. 자세한 내용은 [extends](https://docs.docker.com/compose/extends/)를 참고하면 된다.

## 일반적인 사용 사례

컴포즈는 여러 방법으로 사용 된다. 몇몇 일반적인 사용 사례는 아래에서 설명한다.

### 개발 환경

소프트웨어를 개발하고 있다면, 어플리케이션을 독립된 환경에서 실행하는 것과 서로 간에 상호작용을 하는 일은 중요하다. 컴포즈 커맨드라인 툴은 독립된 환경을 만드는 것과 서로 간에 상호작용 하는 것을 할 수 있게 해준다.

컴포즈 파일은 어플리케이션의 서비스간 의존성(DB, cache, web service API등) 설정들을 문서화 해준다. 컴포즈 커맨드라인 툴을 사용함으로써 각 의존성에 대해 단일 커맨드(`docker-compose up`)로 하나 이상의 컨테이너(DB, API server, cache server등의 컨테이너)를 실행시킬 수 있다.

이런 특징들을 개발자들에게 프로젝트를 시작하는 편리한 방법을 제공한다. 컴포즈는 여러 페이지에 걸친 개발자 시작 가이드를 몇 개의 커맨드와 컴포즈 파일로 줄일 수 있게 해준다.

### 자동화된 테스트 환경

CI/CD 과정에서 중요한 부분 중 하나는 자동화된 테스트이다. 자동화된 end-to-end 테스트는 테스트를 실행 시킬 환경이 필요하다. 컴포즈는 이 테스트를 위한 독립된 환경을 만들고 파괴하는 편리한 방법을 제공한다. 컴포즈 파일 내에 전체 환경을 정의만 해놓는다면, 이런 환경을 몇개의 커맨드만으로 만들고 없앨 수 있다.

```bash
#!/bin/bash
docker-compose up -d
./run_tests
docker-compose down
```

### 단일 호스트 배포 (Single Host Deployments)

컴포즈는 전통적으로 개발과 테스트 워크플로우에 집중했으나, 버전업이 되면서 좀 더 프로덕션 지향적인 기능에 발전이 있었고 좀 더 신경을 쓰고 있다.

아래는 무슨 내용인지 모르겠습니다.

Compose has traditionally been focused on development and testing workflows, but with each release we’re making progress on more production-oriented features. You can use Compose to deploy to a remote Docker Engine. The Docker Engine may be a single instance provisioned with Docker Machine or an entire Docker Swarm cluster.

프로덕션 지향적인 기능에 대해 더 자세히 알고 싶다면, [compose in production](https://docs.docker.com/compose/production/)을 보면 된다.
