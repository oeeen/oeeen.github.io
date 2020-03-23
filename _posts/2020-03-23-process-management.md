---
layout: single
title:  "Process Management"
date:   2020-03-23 19:55:59 +0900
classes: wide
categories: etc
tags: process
toc: true
toc_sticky: true
---

## Process Management

### 프로세스 생성

*Copy-on-write

Write가 발생 했을 때 Copy를 하겠다.. Write가 없을 때까지는 부모의 것을 그대로 공유하고 있는 것이다. 내용이 수정되면 부모의 code, data, stack을 copy해서 만든다. 물론 통째로 복사하는 것은 아니고 작은 페이지 단위로 복사를 하지만 이 부분은 다음에 알아본다.

부모 프로세스가 자식 프로세스를 생성한다. 프로세스의 계층 구조 형성. 프로세스는 자원을 필요로 함

1. 자식은 부모의 주소 공간을 복사한다. (유닉스에서 fork()로 새로운 프로세스를 생성)
2. 자식은 그 공간에 새로운 프로그램을 올린다. (exec()로 새로운 프로그램을 메모리에 올림)

#### fork()

fork() system call로 프로세스는 생성된다.

```c
int main() {
    int pid;
    pid = fork();
    if (pid == 0) printf("child");
    else if (pid > 0) printf("parent");
}
```

이런 식으로 구성되어있다고 하면, 부모 프로세스가 fork() 를 수행하면 자식 프로세스가 생성된다. 그리고 부모의 컨텍스트를 복사한 후 그 이후부터 수행을 한다. 그래서 자식 프로세스는 fork()문 이후부터 수행하게 되고, 부모 프로세스도 fork()를 수행 후 다음 라인을 쭉 실행한다.

fork()의 return value는 부모 프로세스의 경우에는 양수(>0)가 나오고, 자식 프로세스는 0이 나온다.

#### exec()

어떤 프로그램을 완전히 새로운 프로그램으로 태어나게 해준다..?

```c
int main() {
    printf("1");
    execlp("echo", "echo", "3", (char *) 0);
    printf("2");
}
```

라고 되어있다면, 1 출력 이후 echo로 넘어가서 3을 출력하고 끝날 것이다.(2는 출력되지 않는다.)

#### wait()

커널은 자식 프로세스가 종료될 때 까지 sleep 시킨다.(blocked 상태)

대표적인 예시는, terminal에서 단순 top 명령만 치더라도 terminal process는 sleep 상태로 들어가고 top process가 끝날 때까지 blocked 상태로 들어간다고 생각하면 된다.

#### exit()

프로그램을 종료시키는 시스템 콜

1. 자발적 종료
   - 마지막 statement 수행 후 exit()
   - 명시적으로 적어주지 않아도 main의 return 위치에 컴파일러가 넣어준다.
2. 비자발적 종료
   - 부모 프로세스가 자식 프로세스를 강제 종료시킴
   - kill, break 등
   - 부모가 종료하는 경우(부모가 종료하기 전 자식들이 먼저 종료됨)

### 프로세스 간 협력

1. 독립적 프로세스
   - 프로세스는 각자의 주소 공간을 가지고 수행되므로 원칙적으로 하나의 프로세스는 다른 프로세스의 수행에 영향을 미치지 못함.
2. 협력 프로세스
   - IPC를 통해 하나의 프로세스가 다른 프로세스의 수행에 영향을 미칠 수 있음.
3. IPC (Inter process Communication)
   1. Message passing - 커널을 통한 메세지 전달
   2. Shared Memory

#### Message Passing

Direct Communication - kernel을 통해 특정 프로세스에 메시지를 보낸다.

Indirect Communication - kernel의 mailbox에 넣어두고 받을 프로세스를 명시하지 않고 메시지를 넣어둔다.

#### Shared Memory

프로세스 간에 공통으로 사용하는 공유 메모리 공간으로 프로세스 간 협력을 한다. 그림만 봐도 이해가 잘 된다.

![Shared Memory](/assets/img/os/shared_memory.png)

### Interrupt

1. 외부 소스로부터 인터럽트가 들어옴.
2. 인터럽트를 받은 프로세스 멈춤
3. Interrupt Handling
   1. Interrupt source와 이유 파악
   2. Interrupt service 결정
   3. Invoke ISR(Interrupt Service Routine)

### 참고자료

- [WIKI - Copy On Write](https://en.wikipedia.org/wiki/Copy-on-write)
- [생활 코딩](https://opentutorials.org/module/2974/17297)
