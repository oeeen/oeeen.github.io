---
layout: single
title:  "프로세스 vs. 쓰레드"
date:   2019-09-23 19:55:59 +0900
classes: wide
categories: etc
tags: process thread
---

## Process vs. Thread

컴퓨터공학과를 나왔다고 하면 누군가가 나에게 당혹감을 주기 위해서 인지. 정말로 궁금해서 인지. 프로세스와 쓰레드의 차이를 묻곤 한다.

그러나 항상 머리속으로는 알고 있다고 생각하지만 막상 답을 주려고 하면 어떻게 설명해야 할 지 모르겠다.

## Process

위키에서 정의를 보면 아래와 같다.

> 프로세스(process)는 컴퓨터에서 연속적으로 실행되고 있는 컴퓨터 프로그램을 말한다. 종종 스케줄링의 대상이 되는 작업(task)이라는 용어와 거의 같은 의미로 쓰인다. 여러 개의 프로세서를 사용하는 것을 멀티프로세싱이라고 하며 같은 시간에 여러 개의 프로그램을 띄우는 시분할 방식을 멀티태스킹이라고 한다. 프로세스 관리는 운영 체제의 중요한 부분이 되었다.

정리 해보면

1. 실행중인 프로그램
2. 커널에 의해 관리되고 등록된 엔티티
3. 시스템 자원을 사용하고, 요청할 수 있게 허가된 엔티티
4. PCB에 할당된 엔티티

간단히 말해서 `실행중인 프로그램`이다.

![Process_Concept](/assets/img/os/process_concept.png)

### *PCB(Process Control Block): 커널 영역에 각 프로세스들의 정보를 가지고 있는 영역

프로세스는 아래와 같이 구성된다.

1. Program code: text
2. Global data: data
3. Temporary data: stack
   1. local variable, function parameters, return address
4. Heap
   1. 실행하는 동안 동적으로 할당되는 메모리 영역
5. 프로그램 카운터를 포함한 프로세서 레지스터의 값들
6. 기타..

Process State에 대해 알아볼 내용도 많이 있지만, 이 포스팅에서는 생략한다.

프로세스의 구조는 다음과 같다.

![Process](/assets/img/os/process.png)

멀티 프로세싱은 CPU에서 여러 프로세스들을 OS의 스케쥴링 기법에 따라 처리 해주는 것이다.

## Thread

> 스레드(thread)는 어떠한 프로그램 내에서, 특히 프로세스 내에서 실행되는 흐름의 단위를 말한다. 일반적으로 한 프로그램은 하나의 스레드를 가지고 있지만, 프로그램 환경에 따라 둘 이상의 스레드를 동시에 실행할 수 있다. 이러한 실행 방식을 멀티스레드(multithread)라고 한다.

정리 해보면

1. 프로세스 내에서 실행되는 여러 흐름의 단위
2. 프로세스가 자원을 이용하는 실행의 단위

![Thread](/assets/img/os/thread.png)

위 그림에서처럼 프로세스 내에 쓰레드는 각각 스택만 따로 있고 Code, Data, Heap 영역은 공유한다.
같은 프로세스 안에 있는 다른 쓰레드들은 서로 Code, Data, Heap을 공유하지만, 프로세스는 다른 프로세스의 메모리에 직접 접근할 수 없다.

그래서 장점이 있다.

1. 자원을 효율적으로 사용할 수 있다.
2. 사용자에 대한 응답성이 향상된다.
3. 프로세스 간 전환보다 Context Switching에 대한 비용이 줄어든다.

단점으로는

1. 자원에 대한 동시 접근의 처리를 해주어야 한다.
2. 디버깅 하기 어렵다....

프로세스 내부의 멀티 쓰레드의 흐름은 아래와 같다.

![thread in process](/assets/img/os/thread_in_process.svg)

## Multi-process 와 Multi-thread

멀티프로세스와 멀티스레드는 양쪽 모두 여러 흐름이 동시에 진행된다는 공통점을 가지고 있다. 하지만 멀티프로세스에서 각 프로세스는 독립적으로 실행되며 각각 별개의 메모리를 차지하고 있는 것과 달리 멀티스레드는 프로세스 내의 메모리를 공유해 사용할 수 있다. 또한 프로세스 간의 전환 속도보다 스레드 간의 전환 속도가 빠르다.

프로세스는 OS로부터 CPU, 메모리 등 `자원`을 할당 받는다. 쓰레드는 프로세스 내에서 실행되는 흐름의 단위이므로, `프로세스 내부의 자원`을 쓰레드끼리 서로 공유하면서 실행된다.

한 프로세스에서 멀티 쓰레드로 실행을 하면, 시스템 자원을 효율적으로 관리할 수 있다. 프로세스를 생성하여 자원을 할당하는 시스템 콜이 줄어들어 자원을 효율적으로 관리할 수 있다.

그리고 프로세스 간 전환보다 쓰레드 간 전환의 비용이 더 적다. 그 대신 멀티 쓰레드에서의 자원의 관리에 신경을 써야한다. 여러 쓰레드에서의 동시 접근 가능성을 고려하며 개발을 해야한다.

## 참고 자료

- [WIKI - Thread](https://ko.wikipedia.org/wiki/%EC%8A%A4%EB%A0%88%EB%93%9C_(%EC%BB%B4%ED%93%A8%ED%8C%85))
- [https://magi82.github.io/process-thread/](https://magi82.github.io/process-thread/)
- [https://gmlwjd9405.github.io/2018/09/14/process-vs-thread.html](https://gmlwjd9405.github.io/2018/09/14/process-vs-thread.html)
- [https://brunch.co.kr/@kd4/3](https://brunch.co.kr/@kd4/3)
