---
layout: post
title:  "쿠버네티스 알아보기 - 1"
date:   2019-10-07 20:55:59 +0900
classes: wide
categories: etc
tags: kubernetes
---

큰 서비스를 하는 회사에서는 서버 자원을 효율적으로 쓰기 위해 가상화 기술에 초점을 두었다. 이 가상화 기술에 대한 시간 흐름에 대한 변화는 아래 글에 나와있다.

[쿠버네티스 개념 정리](https://smjeon.dev/etc/kubernetes-basic/)

그래서 지금은 컨테이너 배포의 시대이다! 그리고 쿠버네티스는 여러 컨테이너들을 관리해 주는 솔루션이다. 그러면 왜 쿠버네티스를 쓸까?

## Why Kubernetes

## Container

1. 기존의 서비스의 시스템 환경을 그대로 컨테이너로 만들어서 안정성을 올릴 수 있다.
   1. 배포하는 환경에 상관 없이 일관성있는 배포를 진행 할 수 있다.
2. 여러 컨테이너 간의 호스트 자원을 공유 할 수 있다.
3. 마이크로 서비스 관점에서..

예를 들어보면 A, B, C 모듈이 한 서비스로 구성되어 있고 만약 이 중 C만 트래픽이나 리소스의 소모가 높다면, 사실 C만 여러개로 확장하면 되지만 기존 VM 구조에서는 한 서비스로 구성 되어 있는 A, B, C를 모두 확장해야 한다.

![VM_deploy](/assets/img/k8s_basic/VM.png)

그러나 컨테이너 관점에서는 C 모듈만 Pod로 묶어서 확장하면 된다

![kubernetes](/assets/img/k8s_basic/kubernetes.png)

---

## 쿠버네티스 컴포넌트

## 마스터 컴포넌트

마스터 컴포넌트는 클러스터의 컨트롤 플레인을 제공한다. 마스터 컴포넌트는 클러스터에 관한 전반적인 결정을 수행하고 클러스터 이벤트를 감지하고 반응한다.

마스터 컴포넌트는 클러스터 내 어떠한 머신에서든지 동작 될 수 있다. 그런데 보통 간결성을 위하여 동일 머신 상에 모든 마스터 컴포넌트를 구동시키고, 사용자 컨테이너는 해당 머신 상에 동작시키지 않는다.

[참고 링크 - 고가용성 클러스터 구성하기](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)

### kube-apiserver

Kubernetes API Server는 파드, 서비스, 레플리케이션 컨트롤러를 포함한 API 객체를 확인, 구성한다.

API Server는 REST 동작을 서비스하고, 다른 모든 컴포넌트와 상호작용하는 클러스터의 공유 상태의 프론트엔드를 제공한다.

수평적 스케일을 위해 설계되었다.

### etcd

### kube-scheduler

### kube-controller-manager

## 노드 컴포넌트

노드 컴포넌트는 동작중인 파드를 유지시키고 쿠버네티스 런타임 환경을 제공하며, 모든 노드 상에서 동작한다.

### kubelet

### kube-proxy

## 애드온

쿠버네티스 리소스를 이용하여 클러스터 기능을 구현한다. 클러스터 단위의 기능을 제공하기 때문에 애드온에 대한 네임스페이스 리소스는 kube-system 네임스페이스에 속한다.

## 참고자료

- [쿠버네티스 컴포넌트](https://kubernetes.io/ko/docs/concepts/overview/components/)
