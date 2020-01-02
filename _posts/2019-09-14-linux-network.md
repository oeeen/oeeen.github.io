---
layout: single
title:  "Linux 네트워킹"
date:   2019-09-14 23:00:59 +0900
classes: wide
categories: etc
tag: linux
toc: true
toc_sticky: true
---

## 네트워킹

하단에 나오는 명령어들은 다음과 같다.

- ping
- traceroute
- netstat
- wget
- ssh

## ping

패킷을 지정된 간격에 따라 계속 전송한다.

![ping](/assets/img/network/ping.png)

## traceroute - 네트워크 패킷 경로 추적하기

![traceroute](/assets/img/network/traceroute.png)

각 라우터 별로 호스트명, IP 주소, 로컬에서 라우터까지의 왕복시간에 대한 3번의 테스트 결과를 볼 수 있다. 라우터를 식별할 수 없는 경우는 *로 표시된다.

## netstat - 네트워크 설정 및 통계 정보 확인하기

netstat은 다양한 네트워크 설정 사항이나 통계 정보를 확인할 수 있다.

`man netstat`을 통해 옵션들의 내용을 알아본다.

Option | Description
--- | ---
-a | 활성화된 모든 TCP 연결 정보를 보여줌
-i | 모든 설정된 네트워크 인터페이스들의 상태를 보여줌
-l | IPv6 전체 주소 출력
-m | 메모리 관리 루틴에 의해 기록된 통계를 보여준다. (네트워크 스택은 메모리 버퍼의 풀을 관리한다.) 더 자세한 정보를 보려면 -mm or -m -m 옵션 확인
-n | network 주소를 숫자로써 보여준다.
-p | 프로토콜에 관한 통계를 보여준다.
-r | 라우팅 테이블을 보여준다.
-s | 프로토콜 별 상세한 네트워크 통계를 보여줌
-v | 더 상세하게
-w | wait seconds의 간격에 네트워크 프로토콜이나 인터페이스를 보여준다.
-x | 확장된 link-layer reachability 정보를 보여준다.

- 참고: [http://www.ktword.co.kr/abbr_view.php?m_temp1=2220](http://www.ktword.co.kr/abbr_view.php?m_temp1=2220)

## wget - 비대화식 네트워크 다운로더

웹이나 FTP 사이트를 통해 컨텐츠를 다운로드할 때 유용한 프로그램이다.

`wget http://linuxcommand.org/index.php` 명령을 수행하면 다음과 같다.

![wget](/assets/img/network/wget.png)

![wget result](/assets/img/network/wget_result.png)

## ssh - 원격 컴퓨터에 안전하게 로그인

전에 ec2에 접속했던 것 처럼 접속하면 된다.

```bash
#!/bin/bash
serverip=[EC2 IP]
keyname=[EC2 KEY]
findpath=$(find . -name $keyname | head -n 1)
echo $findpath
chmod 400 $findpath
ssh -i $findpath ubuntu@$serverip
```

[EC2 IP]에 ec2 외부 아이피를 넣고, [EC2 KEY]에 Key 넣고 위 스크립트를 실행시키면 ssh로 우분투에 접속할 수 있다.

### SSH 터널링

SSH로 원격 호스트와 연결할 경우 이뤄지는 작업 중 하나는 로컬 시스템과 원격 시스템 간에 암호화된 터널이 생성되는 것이다. 보통 이 터널은 로컬 시스템에 입력된 명령어가 안전하게 원격 시스템으로 전송되게 하며 또 그 결과를 안전하게 가져올 수 있도록 한다. SSH 프로토콜은 이러한 기본적 기능 외에도 로컬과 원격지 사이에 일종의 VPN을 생성하여 네트워크 트래픽의 대다수 형식이 암호화된 터널을 통해 전송되는 것을 지원한다.
