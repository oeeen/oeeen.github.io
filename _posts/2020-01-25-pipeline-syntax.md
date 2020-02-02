---
layout: single
title:  "Pipeline Syntax - Scripted pipeline"
date:   2020-01-25 18:50:59 +0900
classes: wide
categories: etc
tags: jenkins
toc: true
toc_sticky: true
---

Pipeline Syntax 문서에서 Scripted pipeline 부분을 번역했습니다. 정확한 정보를 원하시는 분들은 [공식문서](https://jenkins.io/doc/book/pipeline/syntax/#scripted-pipeline)를 참고해주시면 감사하겠습니다.

## Pipeline Syntax

이 섹션은 파이프라인 시작할 때 필요한 정보들을 기반으로 만들어졌고, 참조용으로만 사용한다. 예시에서 파이프라인 문법을 사용하는 방법에 대한 자세한 정보는 [Using a Jenkinsfile](https://smjeon.dev/etc/jenkinsfile/)을 참조해라. Pipeline plugin 2.5 기준으로 pipeline은 declarative, scripted pipeline 문법을 지원한다.

Pipeline의 가장 중요한 부분은 `steps`이다. 기본적으로 steps는 jenkins에게 무엇을 해야하는지 알려주고, declarative와 scripted pipeline 문법을 위한 기본 구성 요소이다.

### Scripted Pipeline

Declarative pipeline과 비슷하게 Scripted pipeline도 파이프라인 하위 시스템 위에 구축되어 있다. Declarative와는 달리, Scripted pipeline은 groovy로 작성된 범용 DSL이다. 그루비가 제공하는 대부분의 기능이 scripted pipeline 사용자는 사용할 수 있고, 이것은 CD pipeline을 작성하는데 더 표현적이고 유연한 도구가 될 수 있다.

#### Flow Control

Scripted pipeline은 그루비 또는 다른 대부분의 스크립트 언어처럼 Jenkinsfile의 위부터 아래로 순차적으로 실행된다. 그러므로 흐름제어를 제공하는 것은 다음처럼 그루비 표현식처럼 한다. 예를 들어:

```groovy
node {
    stage('Example') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }
}
```

Scripted pipeline 흐름 제어를 관리하는 다른 방식은 그루비의 예외 처리 방법이 있다. `Steps`가 fail하면 어떤 이유든지 예외를 던진다. 그 예외에 대한 처리 동작은 그루비에서 try/catch/finally 블럭을 사용한다. 예를 들어:

```groovy
node {
    stage('Example') {
        try {
            sh 'exit 1'
        }
        catch (exc) {
            echo 'Something failed, I should sound the klaxons!'
            throw
        }
    }
}
```

#### Steps

위에서 말했듯이, 파이프 라인의 가장 근본적인 부분은 `steps`이다. 기본적으로 `steps`는 Jenkins에게 무엇을 해야하는지 알려주고 declarative와 scripted pipeline 문법을 위한 기본 구성 요소이다.

Scripted pipeline은 구문에 특별한 steps가 있지 않다. [Pipeline steps reference](https://jenkins.io/doc/pipeline/steps/)에는 파이프라인과 플러그인이 제공하는 steps의 목록이 포함되어있다.

#### Differences from plain Groovy

파이프라인을 실행하는 것이 Jenkins master의 restart로부터 살아남을 수 있는 durability를 제공하기 위해, Scripted pipeline은 데이터를 master로 직렬화 해야한다. 이 설계 요구사항 때문에, 그루비 문법(`collection.each { item -> /* perform operation */`)이 완전하게 지원되지는 않는다.

#### Syntax Comparison

젠킨스 파이프라인이 처음 만들어졌을 때 그루비는 재단으로 선정되었다. 젠킨스는 관리자와 일반 유저 모두에게 고급 스크립팅을 제공하기 위해 오랫동안 내장 그루비 엔진을 탑재했다. 또한 젠킨스 파이프라인의 구현자들은 그루비가 현재 Scripted pipeline DSL의 견고한 토대가 된다는 것을 알았다.

완전한 기능을 프로그래밍 환경이기 때문에, Scripted pipeline은 젠킨스 사용자에게 엄청난 유연함과 확장성을 제공하낟. 그루비 러닝 커브는 일반적으로 너무 어려울 수 있으니, Declarative pipeline은 파이프라인 작성자가 더 쉽고 접근성 있게 의견을 나눌 수 있도록 하기 위해 만들어졌다.

둘 다 근본적으로는 동일한 파이프라인의 하위 시스템이다. 둘다 `Pipeline as code`을 구현한 것이다. 둘다 파이프라인의 내장, 또는 플러그인이 제공하는 steps를 사용할 수 있다. 둘 다 공유 라이브러리도 사용할 수 있다.

그러나 둘의 다른 점은 문법과 유연성이다. Declarative는 좀 더 엄격한 구조를 사용해야 하므로, 간단한 CD 파이프라인에는 적합하다. Scripted pipeline은 그루비 문법을 기준으로 작성되기 때문에, Declarative pipeline과 달리 프로그래머에게 좀 더 유연한 작성을 하게 할 수 있다. 그러나 그루비 문법에 대한 학습이 필요하고, 그루비 언어에 대한 역량에 따라 이 pipeline을 작성하는데 능력이 달라질 수 있다.

### 참고자료

- [Pipeline Syntax](https://jenkins.io/doc/book/pipeline/syntax/)
