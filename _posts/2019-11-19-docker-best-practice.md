---
layout: post
title:  "Best practices for writing Dockerfiles"
date:   2019-11-19 18:55:59 +0900
classes: wide
categories: etc
tags: docker
toc: true
toc_sticky: true
---

## Docker 공식문서 번역 - [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

효율적인 이미지를 만들기 위해 추천하는 Best Practice를 작성한 문서입니다. **내용 중 제 개인 판단으로 불필요하다고 생각하는 부분은 생략했습니다.**

도커는 Dockerfile로부터 명령을 읽어서 자동적으로 이미지를 만든다.

도커 이미지는 Dockerfile 명령이 각각 나타내는 read-only 계층으로 구성된다. 각 계층은 이전 레이어로부터 차이(delta)를 stack 처럼 쌓습니다. 아래 Dockerfile을 예시로 들어서 설명한다.

```bash
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

각 명령은 한 계층을 만듭니다.

- `FROM`: ubuntu:18.04 도커 이미지로부터 한 계층을 만든다.
- `COPY`: 도커 클라이언트의 현재 디렉토리로부터 파일을 추가한다.
- `RUN`: make로 애플리케이션을 빌드한다.
- `CMD`: 컨테이너 내부에서 실행할 명령을 말한다. 이미지를 실행하고 컨테이너를 만들 때, 쓰기 가능한(writable) 계층을 추가한다. 실행중인 컨테이너에서 생긴 모든 변화(새 파일 쓰기, 파일 지우기, 파일 수정 등..)는 쓰기 가능한(writable) 컨테이너 계층에 쓰인다.

이미지 계층에 대해 더 알고 싶으면 [storage driver](https://docs.docker.com/storage/storagedriver/)에 대해 알아보자.

![container-layers](/assets/img/docker_docs/container-layers.jpg)

컨테이너와 이미지의 가장 큰 차이는 맨 위의 writable 계층이다. 새로운 데이터, 수정된 데이터는 모두 writable 계층에 작성된다. 컨테이너가 지워질 때, writable 계층도 같이 지워진다. 그 아래 이미지 계층은 바뀌지 않고 남아있다. 각 컨테이너들은 writable 계층을 따로 가지고 있기 때문에, read-only 계층은 공유하고 writable 계층은 따로 갖는다.

![sharing-layers](/assets/img/docker_docs/sharing-layers.jpg)

도커는 writable 컨테이너 계층과 이미지 계층의 내용을 관리하기 위해 storage driver를 사용한다. 각 스토리지 드라이버는 구현을 다르게 처리하고, 모든 드라이버는 스택형 이미지 계층과 copy-on-write 전략을 사용한다.

### 디스크에서 컨테이너 크기

실행 중인 컨테이너의 크기를 보려면 `docker ps -s` 커맨드를 쳐볼 수 있다.

- size: 디스크 위의 data의 양(writable layer)를 위해 사용된다.
- virtual size: read-only 이미지와 writable layer의 데이터를 위해 사용된다. 여러 컨테이너들은 read-only image 데이터들을 공유할 수 있다.

![docker-ps-s](/assets/img/docker_docs/docker-ps-s.png)

실행중인 컨테이너에서 사용되는 전체 디스크 공간은 각 컨테이너의 사이즈와 가상 사이즈 값의 조합이다. 만약 여러 컨테이너가 같은 이미지로부터 시작되었다면, 전체 디스크 사이즈는 각 컨테이너의 가상 사이즈 + 한개의 이미지 사이즈다.(같은 이미지를 사용하기 때문에, 여러 컨테이너를 띄운다고 해서 이미지 사이즈가 중복되지는 않는다는 말이다.)

## 임시 컨테이너 만들기

Dockefile로 정의된 이미지는 임시로 컨테이너를 만든다. 임시 라는 것은 컨테이너가 멈추고 없앤 후에 최소 설정 및 구성으로 재빌드하고 교체할 수 있다.

## 빌드 컨텍스트 이해

도커 빌드 커맨드를 날렸을 때, 현재 워킹 디렉토리는 빌드 컨텍스트라고 부른다. Default로 Dockerfile은 여기에 위치되었다고 가정하지만, -f옵션으로 다른 위치를 지정할 수도 있다. 도커파일이 실제 있는 위치와 관계없이, 현재 디렉토리 내부의 내용은 빌드 컨텍스트로서 도커 데몬에게 보내진다.

### 빌드 컨텍스트 예제

빌드 컨텍스트로 쓰일 디렉토리를 하나 만들고 그 디렉토리로 들어간다. 텍스트파일을 하나 만들고 Dockerfile도 하나 만들고 그것을 빌드해보는 예제를 들어본다.

```bash
mkdir myproject && cd myproject
echo "hello" > hello
echo -e "FROM busybox\nCOPY /hello /\nRUN cat /hello" > Dockerfile
docker build -t helloapp:v1 .

mkdir -p dockerfiles context
mv Dockerfile dockerfiles && mv hello context
docker build --no-cache -t helloapp:v2 -f dockerfiles/Dockerfile context
```

만약에 빌드 컨텍스트에 이미지 빌드할 때 필요 없는 파일들이 포함되어 있다면, build context가 도커 데몬에게 보내질 때 불필요하게 빌드 시간, 이미지 푸시, 풀 시간, 런타임 컨테이너 크기 등이 늘어날 수 있다.

위 예제에서 context 폴더 안에 약 80mb크기의 파일(빌드에 상관없는)을 옮겨놓고 docker build를 했을 때 나온 결과는 아래와 같다.

![build-context](/assets/img/docker_docs/build-context.png)

`Sending build context to Docker daemon  80.45MB` 부분을 보면 build context의 사이즈를 확인할 수 있다.

## stdin을 이용한 Dockerfile

stdin을 이용해서 Dockerfile을 파이프화 하는 것으로 Dockerfile을 디스크에 만들지 않고 일회성으로 빌드를 할 수 있다.

`echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -` 와 같은 커맨드로 확인해볼 수 있다. 아래와 같은 커맨드로도 해결할 수 있다.

```bash
docker build -t myimage:latest -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```

이 방식은 도커 이미지가 특정한 파일이 필요하지 않거나, 빌드 컨텍스트에 아무런 파일이 필요하지 않을 때 유용할 수 있다. 만약 빌드 컨텍스트에서 특정한 파일을 제외하고 싶다면 .dockerignore를 가지고 제외할 수 있다.

만약 빌드 컨텍스트는 현재 위치로 설정하면서 stdin으로 이미지를 빌드 하고 싶다면 다음과 같은 방식으로 진행하면 된다.

```bash
docker build -t myimage:latest -f- . <<EOF
FROM busybox
COPY somefile.txt .
RUN cat /somefile.txt
EOF
```

그래서 다음과 같은 방식도 가능하다.

```bash
docker build -t myimage:latest -f- https://github.com/docker-library/hello-world.git <<EOF
FROM busybox
COPY hello.c .
EOF
```

<https://github.com/docker-library/hello-world.git>을 빌드 컨텍스트로 지정해서 해당 빌드 컨텍스트에서 파일을 복사하거나 이용할 수 있게 되었다.

이 예시에서 처럼 git repository를 빌드 컨텍스트로 사용하면, 도커는 해당 repository를 git clone 한다. 그리고 도커 데몬에 빌드 컨텍스트로서 그 파일들을 보낸다. 이 경우에는 host에 깃이 설치되어 있어야 한다.

## .dockerignore 파일을 이용한 제외

빌드 할 때 빌드 컨텍스트에서 제외하고 싶은 파일이 있다면 .dockerignore 파일을 이용해서 할 수 있다. 이 파일은 .gitignore와 비슷한 패턴으로 사용하면 된다.

## multi-stage 빌드 사용

멀티 스테이지 빌드는 최종 이미지의 사이즈를 크게 줄일 수 있게 해준다.(파일이나 사이에 있는 계층의 수를 줄이지 않고도..)

빌드 과정의 마지막 단계에서 이미지가 만들어지기 때문에, 빌드 캐시를 활용해서 이미지 계층을 최소화 할 수 있다. 예를 들어 내 빌드가 여러 계층으로 구성된다면, 그 계층들의 순서를 적게 변하는 것에서 -> 많이 변하는 것으로 순서를 정할 수 있다.

- 내 어플리케이션을 빌드하기 위해 필요한 툴 설치
- 라이브러리 의존성 설치
- 애플리케이션 실행

Go application을 위한 Dockerfile 예시를 보면 다음과 같다.

```bash
FROM golang:1.11-alpine AS build

# 프로젝트에 필요한 툴 설치
# 의존성 업데이트를 위해 `docker build --no-cache .` 실행
RUN apk add --no-cache git
RUN go get github.com/golang/dep/cmd/dep

# 프로젝트 의존성 복사 List project dependencies with Gopkg.toml and Gopkg.lock
# 이 계층은 Gopkg가 업데이트 됬을 때만 재 빌드 된다.
COPY Gopkg.lock Gopkg.toml /go/src/project/
WORKDIR /go/src/project/
# 라이브러리 의존성 설치
RUN dep ensure -vendor-only

# 전체 프로젝트 복사 및 빌드
# 이 계층은 프로젝트 디렉토리가 변경 됬을 때만 재 빌드 된다.
COPY . /go/src/project/
RUN go build -o /bin/project

# 단일 계층 이미지가 생성된다.
FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
```

## 불필요한 패키지 설치하지 마라

복잡도와 의존성, 파일 크기, 빌드 시간을 줄이기 위해서 추가 또는 불필요한 패키지를 설치하지 않아야 한다.(그냥 가지고 있으면 좋을 거 같아서~ 이런 이유;) 예를 들어 데이터베이스 이미지에는 텍스트 에디터가 필요 없다.

## 어플리케이션 결합도 줄이기

각 컨테이너는 하나의 관심사만을 가져야 한다. 어플리케이션 간 결합도를 줄이는 것이 수평적으로 늘리는 것과 컨테이너를 재사용하는 것을 쉽게 만들어 준다. 예를 들어 웹 어플리케이션은 웹 어플리케이션, 데이터베이스, 인메모리 캐시를 분리해서 구성 할 수 있다.

각 컨테이너를 하나의 프로세스로 제한 하는 것은 경험상 좋은 법칙이지만, 강하게 제한 하는 것은 아니다. 예를 들어 컨테이너들이 초기화 과정을 통해 생겨날 수도 있고, 어떤 프로그램들은 추가 프로세스를 생성할 수 도 있다. 예를 들어 아파치는 요청 당 하나의 프로세스를 추가로 생성할 수도 있다.

가능하면 컨테이너를 깔끔하고 하나의 모듈 형태로 관리할 수 있도록 해야한다. 만약 컨테이너가 서로에게 의존한다면 도커 네트워크를 이용해서 각 컨테이너 간 통신을 할 수 있도록 해야 한다.

## 계층의 수를 최소화 하기

예전 버전의 도커에서는 이미지의 계층 숫자를 최소화 하는 것이 중요했다.

가능하다면, multi-stage 빌드를 사용하고, 최종 이미지에 필요한 요소만 복사해라. 이것은 최종 이미지의 크기를 키우지 않으면서 중간 빌드 단계에 디버깅 정보와 도구들을 포함하게 해준다.

## multi-line 요소 정렬

가능하다면, 여러 줄로 인수들을 정렬하여 변경 사항을 반영하기 쉽게 만들어라. 이것은 패키지의 중복을 피하거나 수정을 쉽게 만들어준다. 또한 PR을 읽기 쉽고 리뷰하기 쉽게 해준다. 스페이스와 `\`를 활용하는게 도움이 된다. 아래 이에 대한 예시가 있다.

```bash
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

## 빌드 캐시 활용

이미지를 빌드할 때, 도커는 당신의 Dockerfile 내에 있는 명령어를 지정된 순서대로 수행한다. 각 명령들이 검사될 때, 도커는 재사용 가능한 캐시를 존재하는 이미지에서 찾아본다.(새로운 중복이미지를 만드는 대신에)

만약 네가 캐시를 쓰고 싶지 않다면, `--no-cache=true` 옵션을 활용할 수 있다. 그러나 도커에게 캐시를 쓸 수 있게 할 것이라면, 언제 캐시를 쓸 수 있는지, 없는지, 맞는 이미지는 찾을 수 있는지를 아는 것이 굉장히 중요하다. 도커가 따르는 기본적인 규칙은 다음과 같다.

- 이미 캐시 안에 있는 부모 이미지와 함께 시작한다면, 다음 명령은 그 부모 이미지로부터 파생된 모든 자식 이미지와 비교하면서 정확하게 동일한 명령을 사용해서 빌드 되었는지 확인한다. 그렇지 않으면 캐시는 무효화 된다.
- 대부분의 경우에, 단순히 Dockerfile 내부의 명령을 비교하는 것 자식 이미지를 비교하는 것으로 충분하다. 그러나 어떤 명령들은 더 많은 설명과 검사가 필요하다.
- ADD, COPY 명령에 대해 이미지 안의 파일 내용은 검사되고, 체크섬은 각 파일마다 계산 된다. 마지막으로 수정 및 액세스 타임은 이 체크섬에 고려되지 않는다. 캐시를 찾는 동안 체크섬은 이미 존재하는 이미지의 체크섬과 비교된다. 만약 파일안에 어떤 것이라도 바뀌었다면(내용이나 메타데이터) 캐시는 무효화된다.
- ADD, COPY 명령 외에는 캐시 검사는 캐시 일치를 확인하기 위해 파일 내용을 살펴보지 않는다. 예를 들어 컨테이너 내부에서 `RUN apt-get -y update command`를 수행하면서 업데이트된 파일들은 캐시 일치를 확인할 때 검사하지 않는다. 이 경우에는 명령 문자 그 자체를 일치하는데 검사한다. (그러니까 Dockerfile 내의 `RUN apt-get -y update command`라는 명령이 있는지 확인)

캐시가 무효화 되면, 이어지는 모든 도커파일 명령어들은 새 이미지는 만들고 캐시는 사용되지 않는다.

## Dockerfile 명령들

효율적이고 유지보수에 능한 Dockerfile을 만드는데 도움이 되도록 설계할 수 있도록 하는 추천사항들이다.

### FROM

가능하다면 네 이미지를 위해서 오피셜 이미지를 기반으로 사용해라. Alpine image는 완전한 리눅스 배포판이면서 용량도 작기 때문에 Alpine 이미지를 사용하는 것을 추천한다.

### LABEL

너는 프로젝트의 이미지를 관리하고, 라이센스 정보를 기록하고, 자동화를 돕거나 다른 이유를 위해 이미지에 라벨을 넣을 수 있다. 각 라벨들은 하나 이상의 key-value 쌍으로 되어있다. 뒤에 나오는 예시는 각각 다른 포맷들을 보여준다. space를 포함한 문자열은 따옴표로 감싸거나, 이스케이프 해야한다. 내부의 "도 이스케이프 처리 해야한다.

```bash
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor1="ACME Incorporated"
LABEL vendor2=ZENITH\ Incorporated
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
```

도커 1.10 이후부터는 하나의 LABEL 명령안에 모두 조합하는 것을 추천한다.(추가적인 계층이 생기지 않도록)

```bash
# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

여러 개의 라벨 명령어를 사용하지 않고, 하나의 라벨 명령에 여러 키-벨류 쌍으로 작성하는 것을 추천한다.

### RUN

길거나 복잡한 RUN 문을 분리해서 여러 줄로 나누어서 Dockerfile을 읽기 쉽고, 이해하기 쉽고 유지보수 하기 쉽게 해야한다.

### APT-GET

대부분의 RUN의 use-case에서는 apt-get을 사용한다. apt-get은 패키지를 설치하기 때문에 RUN apt-get 명령은 몇 가지 주의할 점이 있다.

부모 이미지중 필수 패키지 중 다수는 권한 없는 컨테이너에서는 업그레이드를 할 수 없기 때문에, RUN apt-get 업그레이드를 피해야한다. 만약 부모 이미지에 포함된 패키지가 오래되었다면 그 이미지의 유지보수자에게 연락해라. 만약 apt-get update와 apt-get install을 항상 같은 RUN 문 안에서 함께 써라. 예를 들면 아래와 같다.

```bash
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo
```

하나의 RUN 명령안에 apt-get update를 따로 쓰는 것은 캐싱 이슈의 원인이 될 수 있고, 그 이후에 따라오는 apt-get install 명령은 실패할 수도 있다. 예를 들어

```bash
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl
```

위와 같은 도커파일로 이미지를 빌드 한 후에, 약간 수정이 필요해서 아래 파일처럼 수정하게 되었다고 생각해보자.

```bash
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl nginx
```

도커는 처음 Dockerfile과 수정된 명령이 동일한 것으로 보고 캐시에서 꺼내서 재사용한다. 그 결과 apt-get update는 수행되지 않는다. apt-get update가 수행되지 않았기 때문에, 빌드는 curl과 nginx 패키지의 예전 버전을 얻게 된다.

그래서 `RUN apt-get update && apt-get install -y`는 도커파일이 최신의 패키지 버전을 얻을 수 있게 해준다.(cache busting이라고 한다) cache busting은 패키지 버전을 특정 지음으로써도 할 수 있다.(version pinning) 예를 들면 아래와 같다.

```bash
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo=1.3.*
```

이 모든 것을 고려한 RUN 명령의 예시는 아래와 같다.

```bash
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```

이 명령에서 `/var/lib/apt/lists`를 삭제 하면서 계층에 apt 캐시를 저장하지 않으면서 이미지 사이즈를 줄여준다. RUN 명령이 apt-get update로 시작하기 때문에 패키지 캐시는 apt-get install 전에 항상 리프레쉬 된다.

> 오피셜 Debian, Ubuntu 이미지는 자동적으로 apt-get clean 명령을 수행하기 때문에 명시적으로 따로 수행할 필요는 없다.

### CMD

CMD 명령은 네 이미지에 포함된 소프트웨어를 인자들과 함께 돌리는데 사용되어야 한다. CMD는 거의 `CMD ["executable", "param1", "param2"…]`의 형태로 사용된다. 그러므로 만약 이미지가 아파치나 Rails 같은 서비스를 위한 것이라면 `CMD ["apache2","-DFOREGROUND"]`이런 식으로 실행 해야한다. 실제로 이 형태를 서비스 기반 이미지에 추천한다.

대부분의 다른 경우에는 CMD는 shell 커맨드를 쓴다. 예를 들어 `CMD ["perl", "-de0"], CMD ["python"], or CMD ["php", "-a"]` 이런 것이다. ENTRYPOINT를 쓰는 것에 익숙하지 않다면, `CMD ["param", "param"]`와 같은 식으로 ENTRYPOINT와 조합해서 쓰는 방식은 쓰지 않는 편이 좋다.

### EXPOSE

EXPOSE 명령은 컨테이너가 연결을 수신하는 포트를 나타낸다. 결과적으로 네 애플리케이션에서 사용하는 전통적인 포트를 쓰면 된다. 전에 구축했던 스프링 웹 프로젝트는 80번 포트를 열어두면 된다. mysql 컨테이너는 3306 포트를 사용하면 된다.

### ENV

새 소프트웨어를 실행하기 더 쉽게 만들기 위해서 컨테이너가 설치가는 소프트웨어를 위한 환경변수를 업데이트 하는데 ENV를 사용할 수 있다. 예를 들어 `ENV PATH /usr/local/nginx/bin:$PATH`는 `CMD ["nginx"]`를 작동하게 해준다. 그러니까 컨테이너화 하고 싶은 소프트웨어에 필요한 환경변수를 주는데 유용하게 사용할 수 있다.

마지막으로 ENV는 유지보수를 편하게 하기 위해 버전 넘버를 셋팅 해놓을 수 있다. 아래 예시가 있다.

```bash
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

java에서 `static final String MESSAGE = "상수입니다.";`와 같은 상수를 선언하는 것과 비슷하게, ENV를 이용해서 컨테이너의 버전을 표현할 수 있다.

각 ENV 라인은 RUN 명령처럼 새로운 중간 계층을 만든다. 이는 그 위 계층에서 환경 변수를 해제 하지 않는다면 이 계층의 환경변수는 계속해서 유지된다는 것을 의미한다. Dockerfile을 만듦으로 이를 테스트 해볼 수 있다. 예시가 아래 있다.

```bash
FROM alpine
ENV ADMIN_USER="mark"
RUN echo $ADMIN_USER > ./mark
RUN unset ADMIN_USER
```

```bash
$docker run --rm test sh -c 'echo $ADMIN_USER'
mark
```

![env-exercise](/assets/img/docker_docs/env-exercise.png)

이것을 막기 위해서 환경변수를 unset 해줘야한다. 하나의 RUN 명령에서 환경변수를 set하고, 사용하고 unset까지 해줘야 한다. 아래 예시에서 처럼 하나의 RUN 명령 내에서 환경 변수를 셋하고 사용하고 해제까지 해야한다.

```bash
FROM alpine
RUN export ADMIN_USER="mark" \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
CMD sh
```

```bash
$docker run --rm test sh -c 'echo $ADMIN_USER'

```

![env-exercise2](/assets/img/docker_docs/env-exercise2.png)

### ADD or COPY

ADD와 COPY는 기능적으로 비슷하지만 일반적으로 COPY가 선호된다. 왜냐하면 ADD보다 더 투명하기 때문이다. COPY는 컨테이너 내부에 로컬 파일을 복사하는 것을 하는 반면 ADD는 어떤 불명확한 기능을(tar 추출, remote URL 지원 같은..)를 가지고 있다. 결과적으로 ADD의 가장 바람직한 사용은 `ADD rootfs.tar.xz /.`와 같은 명령으로 local tar 파일을 자동 추출하는 것이다.

만약 도커파일 내에 컨텍스트로부터 여러 파일들을 사용하는 단계가 여러개 있다면, 한번에 복사하지 말고 여러번에 걸쳐 각각 복사 해야한다. 이것은 필요한 파일이 변경되는 경우에만 빌드 단계의 캐시가 무효화 되게 만든다. 예를 들어,

```bash
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

이미지 크기가 중요하기 때문에 원격 URL에서 패키지를 가져오는데 ADD를 사용하는 것은 추천하지 않는다. 대신 curl이나 wget을 사용한다. 파일을 추출한 후에 필요 없는 파일을 지울 수 있고, 이미지에 다른 계층에 필요 없는 파일을 추가할 필요가 없다.

```bash
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

위처럼 하는 것 대신에 아래처럼 하는 것이 좋다.

```bash
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

파일이나 디렉토리로부터 파일을 추가하는 것도 ADD는 필요없고 COPY를 쓰면 된다. ~~(근데 그러면 ADD는 왜 쓰지?)~~

### ENTRYPOINT

ENTRYPOINT의 좋은 사용법은 이미지의 메인 커맨드를 설정하고, 이미지를 커맨드처럼 사용할 수 있게 하는 것이다.(CMD를 기본 플래그로 사용할 수 있다.)

예시를 들어보면

```bash
ENTRYPOINT ["docker-compose"]
CMD ["-p practice up -d"]
```

이런식으로도 할 수 있다.

도커 문서에 나와있는 예시는 아래와 같다.

```bash
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

그리고 이제 `$ docker run s3cmd` 이 명령으로 실행할 수 있다. 아니면 `$ docker run s3cmd ls s3://mybucket`과 같이 인자를 함께 넘길 수도 있다.

ENTRYPOINT는 헬퍼 스크립트와 함께 사용할 수도 있다. 아래 예시를 통해 확인할 수 있다.

```bash
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

이 헬퍼 스크립트를 복사해서 ENTRYPOINT를 통해 컨테이너가 시작할 때 등록할 수 있다.

```bash
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```

이 스크립트는 Postgres와 여러 방식으로 상호작용을 할 수 있도록 해준다.

`$ docker run postgres`, `$ docker run postgres postgres --help`, `$ docker run --rm -it postgres bash`

### USER

서비스가 권한 없이 실행될 수 있다면, USER를 root권한이 없는 유저로 변경하는 것이 좋다. 이미지 안에서 UID/GID는 결정되지 않은 값이 설정된다. 그래서 이 값이 중요하다면 따로 명시적으로 정해주어야 한다. 복잡성을 줄이기 위해 root권한 <-> 일반 유저 왔다갔다를 자주하지 말아야한다.

### WORKDIR

RUN cd 대신 WORKDIR을 사용해야 한다. 그리고 WORKDIR에는 항상 절대 경로를 사용해야 한다.

### ONBUILD

현재 Dockerfile이 빌드 된 후에 ONBUILD 커맨드가 실행된다. ONBUILD 커맨드는 현재 이미지로부터 파생된 어떤 자식 이미지에서도 실행 된다. ONBUILD는 상위가 하위 명령에게 전달하는 명령이라고 생각하면 된다. 도커 빌드는 자식 도커파일의 다른 명령보다 먼저 ONBUILD 명령을 수행한다.

### 다음 주제

- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

### 출처

- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
