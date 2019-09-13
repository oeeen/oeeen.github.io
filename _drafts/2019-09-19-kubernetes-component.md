---
layout: single
title:  "쿠버네티스 컴포넌트"
date:   2019-09-19 21:55:59 +0900
classes: wide
categories: etc
tags: kubernetes
---

## 마스터 컴포넌트

마스터 컴포넌트는 클러스터의 컨트롤 플레인을 제공한다. 마스터 컴포넌트는 클러스터에 관한 전반적인 결정을 수행하고 클러스터 이벤트를 감지하고 반응한다.

마스터 컴포넌트는 클러스터 내 어떠한 머신에서든지 동작 될 수 있다. 그런데 보통 간결성을 위하여 동일 머신 상에 모든 마스터 컴포넌트를 구동시키고, 사용자 컨테이너는 해당 머신 상에 동작시키지 않는다.

[참고 링크 - 고가용성 클러스터 구성하기](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)

### kube-apiserver

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
