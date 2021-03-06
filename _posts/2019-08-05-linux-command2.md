---
layout: single
title:  "Linux Command Line 2"
date:   2019-08-05 01:00:00 +0900
classes: wide
categories: etc
tag: linux
toc: true
toc_sticky: true
---

## 명령어들

이 포스팅에서 살펴볼 명령어는 다음과 같다.

* type - 명령어의 이름이 어떻게 표시되는지 확인
* which - 실행 프로그램의 위치 표시
* man - 명령어의 man 페이지 표시
* apropos - 적합한 명령어 리스트 표시
* info - 명령어 정보 표시
* whatis - 명령어에 대한 짧은 설명 표시
* alias - 명령어에 별칭 붙이기

## type

type 명령어는 쉘에 내장된 형식으로 명령어 이름을 입력하면 쉘이 실행하게 될 명령어가 어떤 타입인지를 보여준다.

![type](/assets/img/command/type.png)

## which

which는 실행할 프로그램의 정확한 위치를 알려준다.
built-in에는 동작하지 않는다.

![which](/assets/img/command/which.png)

## help

각 쉘 빌트인 마다 내장된 도움말 기능을 본다.

## man

man 명령어가 가장 쓸모 있다고 생각한다.

`man git`

![which](/assets/img/command/man_git.png)

대부분의 리눅스 시스템에서 man 명령어는 매뉴얼 페이지를 표시하기 위해 less 명령어를 사용한다.

man 페이지 구조

섹션 | 내용
--- | ---
1 | 사용자 명령어
2 | 커널 시스템 콜 API
3 | C 라이브러리 API
4 | 장치 노드 및 드라이버와 같은 특수 파일
5 | 파일 포맷
6 | 스크린세이버와 같은 게임이나 미디어 파일
7 | 그 외 여러 종류
8 | 시스템 관리용 명령어

원하는 섹션을 보고 싶으면 `man [section#] [command]` 와 같이 하면 된다.

## apropos - 적합한 명령어 찾기

검색어에 따라 일치하는 명령어의 man 페이지 목록을 검색하는 명령어다. 대략적인 정보만을 보여주긴 하지만 가끔 도움이 된단다.
`man -k` 와 동일한 기능을 한다.

## whatis - 간략한 명령어 정보 표시

이름 그대로 뭔지 알려준다.
특정 키워드에 부합하는 man 페이지에 대하여 그 이름과 한 줄의 간략한 정보를 보여준다.

## info - 프로그램 정보 표시

info 페이지를 보여준다.

* info 명령어

명령어 | 실행
------ | ------
? | 명령어 도움말 보기
PAGE UP or Backspace | 이전 페이지 보기
PAGE DOWN or Spacebar | 다음 페이지 보기
n | 다음 - 다음 노드 보기
p | 이전 - 이전 노드 보기
u | 위로 - 현재 표시된 노드의 상위 노드(주로 메뉴) 보기
ENTER | 현재 커서 위치에 있는 하이퍼링크로 이동하기
q | 종료

## alias

명령어 모음을 별칭으로 지정 해둘 수 있다.
아직까지 딱히 사용할 필요를 못느껴서 쓰지는 않고 있다.
명령의 구조는 `alias name='string'` 과 같다.

alias로 등록한 명령을 삭제하려면 unalias 하면 된다.
별칭으로 등록된 모든 명령을 확인하려면 그냥 `alias` 하면 된다.

그런데 이런 방식으로 등록하면 쉘 세션이 종료되면 alias도 같이 사라진다.
이런 문제를 해결 할 수 있는 방법이 있는데... 다음 기회에 알아보자.
