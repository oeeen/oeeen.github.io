---
layout: single
title:  "쿠버네티스 개념"
date:   2019-09-21 20:55:59 +0900
classes: wide
categories: etc
tags: kubernetes
toc: true
toc_sticky: true
---

## Kubernetes in 5 mins

쿠버네티스의 핵심 개념은 Desired State Management를 기술해야한다는 것이다.

### kubernetes 구성요소

1. Kubernetes cluster service - API가 포함되어있다.
2. Worker
   1. Worker는? Container Host다. Kublet Process가 있다. Kubelet process는 kubernetes cluster service와 통신하기 위한 책임이 있다.

이것들이 Kubernetes environment를 구성하는 요소이다.

use case를 통해 살펴보자.
한 pod에는 하나 이상의 컨테이너 이미지가 있다.

```yaml
Deployment:
    pod1:
        - Container Img 1
        - Container Img 2
        Replicas=3
    pod2:
        - Container Img 3
        Replicas=2
```

이런 설정 파일을 API에 먹이고, 각 워커에 대해 설정하는 것이다.

## 개요

클러스터: 쿠버네티스에서 관리하는 컨테이너화된 애플리케이션을 실행하는 노드라고 하는 기계의 집합. 클러스터는 최소 1개의 워커 노드와 최소 1개의 마스터 노드를 가진다.

쿠버네티스를 사용하려면, 쿠버네티스 API 오브젝트로 클러스터에 대해 사용자가 `바라는 상태`를 기술해야 한다.
어떤 애플리케이션이나 워크로드를 구동시키려고 하는지, 어떤 컨테이너 이미지를 쓰는지, 복제의 수는 몇 개인지, 어떤 네트워크와 디스크 자원을 쓸 수 있도록 할 것인지 등을 의미한다. 바라는 상태를 설정하는 방법은 쿠버네티스 API를 사용해서 오브젝트를 만드는 것인데, 대개 kubectl이라는 커맨드라인 인터페이스를 사용한다. 클러스터와 상호 작용하고 바라는 상태를 설정하거나 수정하기 위해서 쿠버네티스 API를 직접 사용할 수도 있다.

Desired State를 설정하면, 쿠버네티스 컨트롤 플레인 은 Pod Lifecycle Event Generator (PLEG) 를 통해 클러스터의 현재 상태를 바라는 상태와 일치시킨다. 그렇게 함으로써, 쿠버네티스가 컨테이너를 시작 또는 재시작하거나, 주어진 애플리케이션의 복제 수를 스케일링하는 등의 다양한 작업을 자동으로 수행한다. 쿠버네티스 컨트롤 플레인은 클러스터에서 실행 중인 프로세스의 묶음(collection)으로 구성된다.

- 쿠버네티스 마스터는 클러스터 내 마스터 노드로 지정된 노드 내에서 구동되는 세 개의 프로세스 묶음이다. 해당 프로세스는 kube-apiserver, kube-controller-manager 및 kube-scheduler이다.
- 클러스터 내 마스터 노드가 아닌 각각의 노드는 다음 두 개의 프로세스를 구동시킨다.

1. 쿠버네티스 마스터와 통신하는 kubelet
2. 각 노드의 쿠버네티스 네트워킹 서비스를 반영하는 네트워크 프록시인 kube-proxy

![kubernetes concept](/assets/img/kubernetes/k8s-concept.png)

## [쿠버네티스 오브젝트](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)

쿠버네티스는 클러스터의 상태를 관리하기 위한 대상을 오브젝트로 정의한다.

1. 실행 중인 컨테이너화 된 애플리케이션, 노드
2. 해당 애플리케이션에 사용할 수 있는 리소스
3. 재시작 정책, 업그레이드 및 장애 허용과 같은 어플리케이션의 행동 정책

주요한 오브젝트들은 다음과 같다.

### [파드 (Pod)](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)

파드는 쿠버네티스 애플리케이션의 기본 실행 단위이다. 사용자가 만들 수거나 배포할 수 있는 가장 작고 단순한 오브젝트이다. 파드는 내 클러스터위의 프로세스들을 나타낸다.

파드는 애플리케이션의 컨테이너, 스토리지 자원, 네트워크 IP, 컨테이너를 실행시키기 위해 필요한 옵션들을 캡슐화 한다.

쿠버네티스 클러스터 내의 파드는 두 가지 주요하나 방법으로 사용된다.

1. one-container-per-Pod
   - Pod는 단일 컨테이너를 감싸는 wrapper이다.
2. 함께 작동해야하는 여러 컨테이너들을 담는 Pod
   - 자원을 공유할 필요가 있거나, 연관되어 있는 여러 컨테이너들을 하나의 애플리케이션으로 캡슐화 한다.

각 Pod는 주어진 애플리케이션의 단일 인스턴스를 실행하는 것을 의미한다. 만약에 동일한 애플리케이션의 여러 인스턴스를 만들고 싶으면 여러 파드를 사용해야 한다. 쿠버네티스에서 이것을 `replication`이라고 부른다.

### [서비스](https://kubernetes.io/docs/concepts/services-networking/service/)

네트워크 서비스로 파드 Set에서 실행 중인 애플리케이션을 노출하는 추상적인 방법.

쿠버네티스는 파드 Set에 IP, DNS name을 각각 준다. 그리고 로드 밸런싱 할 수 있다.

### [볼륨](https://kubernetes.io/docs/concepts/storage/volumes/)

컨테이너에 있는 디스크 파일은 임시이고, 그것은 컨테이너에서 실행될 때 꽤 심각한 문제가 나타난다.

1. 컨테이너가 터지면, kubelet은 그 컨테이너를 다시 실행 시킨다. 그러나 파일은 사라진다. - 컨테이너는 깨끗한 상태에서 실행된다.
2. 한 파드 내에 함께 실행중인 컨테이너가 공유하고 싶은 파일이 있을 수 있다. 이런 문제를 해결할 수 있다.

### [네임스페이스](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

쿠버네티스는 하나의 물리적 클러스터에 다중 가상 클러스터를 지원한다. 이런 가상 클러스터들을 네임스페이스라고 부른다.

네임스페이스는 다양한 팀, 다양한 프로젝트에 흩어져있는 많은 사용자들이 함께 사용하는 환경에서 사용하려는 의도에서 사용한다.

적은 수의 사용자를 위한 클러스터에서는 굳이 사용할 필요 없다.

### 컨트롤러 - 오브젝트를 기반으로 부가 기능 및 편의 기능 제공

1. Replica Set
2. Deployment
3. Stateful Set
4. Daemon Set
5. Job

## 쿠버네티스 컨트롤 플레인

쿠버네티스 마스터와 kubelet 프로세스와 같은 쿠버네티스 컨트롤 플레인의 다양한 구성 요소는 쿠버네티스가 클러스터와 통신하는 방식을 관장한다. 컨트롤 플레인은 시스템 내 모든 쿠버네티스 오브젝트의 레코드를 유지하면서, 오브젝트의 상태를 관리하는 제어 루프를 지속적으로 구동시킨다. 컨트롤 플레인의 제어 루프는 클러스터 내 변경이 발생하면 언제라도 응답하고 시스템 내 모든 오브젝트의 실제 상태가 사용자가 바라는 상태와 일치시키기 위한 일을 한다.

예를 들어, 쿠버네티스 API를 사용해서 디플로이먼트를 만들 때에는, 바라는 상태를 시스템에 신규로 입력해야한다. 쿠버네티스 컨트롤 플레인이 오브젝트 생성을 기록하고, 사용자 지시대로 필요한 애플리케이션을 시작시키고 클러스터 노드에 스케줄링한다. 그래서 결국 클러스터의 실제 상태가 바라는 상태와 일치하게 된다.

### 쿠버네티스 마스터

클러스터에 대해 바라는 상태를 유지할 책임은 쿠버네티스 마스터에 있다. kubectl 커맨드라인 인터페이스와 같은 것을 사용해서 쿠버네티스로 상호 작용할 때에는 쿠버네티스 마스터와 통신하고 있는 셈이다.

“마스터”는 클러스터 상태를 관리하는 프로세스의 묶음이다. 주로 모든 프로세스는 클러스터 내 단일 노드에서 구동되며, 이 노드가 바로 마스터이다. 마스터는 가용성과 중복을 위해 복제될 수도 있다.

### 쿠버네티스 노드

클러스터 내 노드는 애플리케이션과 클라우드 워크플로우를 구동시키는 머신(VM, 물리 서버 등)이다. 쿠버네티스 마스터는 각 노드를 관리한다. 직접 노드와 직접 상호 작용할 일은 거의 없을 것이다.

## 참고자료

- [Kubernetes in 5mins - youtube](https://www.youtube.com/watch?v=PH-2FfFD2PU)
- [쿠버네티스 개념](https://kubernetes.io/ko/docs/concepts/)
- [쿠버네티스 표준용어집](https://kubernetes.io/ko/docs/reference/glossary/?all=true#term-cluster)
