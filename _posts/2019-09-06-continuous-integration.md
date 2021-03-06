---
layout: single
title:  "Continuous Integration"
date:   2019-09-06 23s:00:59 +0900
classes: wide
categories: etc
tags: ci
toc: true
toc_sticky: true
---

## 지속적 통합

소프트웨어 공학에서, 지속적 통합(continuous integration, CI)은 지속적으로 퀄리티 컨트롤을 적용하는 프로세스를 실행하는 것이다. - 작은 단위의 작업, 빈번한 적용. 지속적인 통합은 모든 개발을 완료한 뒤에 퀄리티 컨트롤을 적용하는 고전적인 방법을 대체하는 방법으로서 소프트웨어의 질적 향상과 소프트웨어를 배포하는데 걸리는 시간을 줄이는데 초점이 맞추어져 있다. 대표적인 CI 툴에는 젠킨스(Jenkins)가 있다.

출처: [지속적 통합 - Wiki](https://ko.wikipedia.org/wiki/%EC%A7%80%EC%86%8D%EC%A0%81_%ED%86%B5%ED%95%A9)

> 구글을 찾다보면 가장 많이 나오는 말이 Integration hell을 해결하기 위한 솔루션이라고 한다.
> 단순하게 이해하면 개발 팀원들이 작성한 코드를 최대한 자주 통합하는 소프트웨어 개발 방법이다. 버전 관리와 자동화된 빌드, 테스트, 리포팅을 통해 최대한 빨리 오류를 발견하고 발생한 문제를 조기에 처리할 수 있다. 이렇게 해서 통합에 드는 시간을 줄이고 발생하는 문제를 최소화 할 수 있다.

CI Process를 그림으로 보면 아래와 같다.

![CI Process](/assets/img/ci/process.png)

### 이미지 출처

- [RedHat Developer](https://developers.redhat.com/blog/2017/09/06/continuous-integration-a-typical-process/)

## 왜 CI가 필요할까

빠르게 고객(여기서는 실제 고객이 될 수도 있고, 팀원이 될 수도, 프로젝트 매니저가 될 수도 있다.)에게 업데이트를 제공하기 위해서..

개발자가 로컬에서만 계속 코드를 수정하고, 작업이 완료된 후에 변경 사항을 마스터에 머지한다면.. 머지하기가 어렵고, 만약 버그가 생겼다면 버그가 오랜 기간동안 축적될 수 있기 때문이다.

## 장점

팀 단위로 일하면서, 함께 일하는 팀원이 많고, 규모가 큰 프로젝트일 경우 장점이 크다. 그러나 혼자 일하더라도 CI를 구축하면 많은 장점이 있다.

1. 빌드/테스트 프로세스의 자동화로 코드 작성 후 커밋만 하면 된다.
   - ![improve-productivity](/assets/img/ci/improve-productivity.png)
2. 짧은 주기로 통합할 수 있으며, 조기에 문제를 발견하고 조치할 수 있다.
   - ![find-bug](/assets/img/ci/CICD_find-bugs.png)
3. 업데이트를 더 빠르게 제공한다.
   - ![update](/assets/img/ci/CICD_deliver-updates.png)
4. 빌드와 테스트를 개인 환경과 독립적으로 구성할 수 있다. 팀 프로젝트시 겪었던 `로컬에선 되는데 서버에선 안돼요` 문제를 빠르게 발견할 수 있다.
5. 다른 개발자가 수정한 내용을 커밋해도 자동으로 빌드하고 통합 테스트가 진행된다.

### 출처

- [AWS - CI/CD](https://aws.amazon.com/ko/devops/continuous-integration/)

## 일반적인 CI 적용

1. 소스의 변경 내역은 버전 관리 서버를 통해 관리한다. (Github 같은..)
2. 소스가 변경되면 수시로 커밋한다.
3. 미완성 소스의 커밋 -> 빌드가 깨질까봐 완성전까지는 커밋하지 않는 나쁜 습관이 생길 수 있다.
   1. git flow 같은 브랜칭 전략을 통해 해결하자
4. **수시로 빌드한다.** (이 과정이 자동화되므로 지속적 통합이 가능하다.)
5. **빌드마다 자동화된 테스트가 구동된다.** (테스트 환경은 실제 운영 환경과 최대한 유사하게 구축한다.)
6. 모든 프로젝트 참여자가 빌드 산출물과 빌드 결과를 확인할 수 있도록 한다. (우리가 해볼 수 있는 건 슬랙 알림이나, 메일 알림, 사내 메신저가 있다면 사내 메신저 알림 정도?)
7. 빌드가 깨질 경우 다음 단계로 진행할 수 없으므로 깨진 빌드를 수정하는 일을 우선순위를 높게 수정한다.
8. 빌드 후 실제 운영서버에 배포해야 한다면, CI 서버에서 이를 자동화 한다. (운영 서버에 배포하는 것은 고려할 사항이 많으므로.. 신중하게 한다.)

> 개인적으로 중요하다고 생각하는 부분이 4, 5번이다. 수시로 빌드한다. 그리고 빌드할 때마다 테스트가 구동된다. 그래서 테스트가 실패한다면 -> 빌드도 실패해야 한다.
이런 과정 때문에라도 더더욱 테스트가 중요해지고, 꼼꼼하게 테스트 코드를 작성해야 하는 것 같다.

## 상용 CI 도구들

1. Bamboo
   - 아틀라시안의 제품이다. 기업에서 사용하려면 구매해야 하고, 오픈소스 프로젝트를 진행한다면 무료이다. 아틀라시안의 제품이어서 아틀라시안의 지라와의 완벽한 통합을 지원한다고 한다. 또한 아마존의 EC2 서비스를 지원해서 EC2 내 가상머신에 에이전트를 설치할 수 있다. 지라와 연동이 가능하므로, 지라에서 Bamboo build와 연결하거나 반대의 경우도 가능하다.
2. Jenkins
   - 허드슨의 주요 개발자가 나와서 만든 프로젝트다. 사용자 및 플러그인 개발자가 많아서 다양한 기능이 제공되고, 관련자료도 많다. 이 블로그에도 관련 글이 있다.

## 참고자료

- [https://12bme.tistory.com/151](https://12bme.tistory.com/151)
- [https://martinfowler.com/articles/continuousIntegration.html](https://martinfowler.com/articles/continuousIntegration.html)
- [https://aws.amazon.com/ko/devops/continuous-integration/](https://aws.amazon.com/ko/devops/continuous-integration/)
