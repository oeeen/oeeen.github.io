---
layout: single
title:  "Jenkinsfile"
date:   2020-01-24 12:50:59 +0900
classes: wide
categories: etc
tags: docker
toc: true
toc_sticky: true
---

Jenkinsfile에 대한 작성 문서를 제 마음대로 번역한 문서입니다. 제 생각에 불필요하다고 생각하는 말은 제거하고, 가끔가다 제 생각이 몇 군데 들어있습니다. 정확한 정보를 원하시는 분들은 [공식문서](https://jenkins.io/doc/book/pipeline/jenkinsfile/)를 참고해주시면 감사하겠습니다.

## Using a Jenkinsfile

이 글에서는 Pipeline을 이용하여 시작하기 위한 정보들과, 더 유용한 step, 일반적인 패턴, 몇개의 Jenkinsfile의 예제를 설명한다.

Jenkinsfile을 Source Control을 통해 관리함으로써 얻을 수 있는 이점들이 있다.

1. 코드리뷰/파이프라인 상의 반복
2. 파이프라인 오류 추적
3. 프로젝트의 여러 구성원들이 보고 편집할 수 있는 파이프라인의 단일 소스 관리

파이프라인은 두 문법을 지원한다. Declarative와 Scripted pipeline이다. 둘다 Contiunous delivery 파이프라인을 만드는 것을 도와준다. 둘 다 웹 UI나 Jenkinsfile에서 파이프라인을 정의하는데 사용할 수 있지만, 일반적으로 Jenkinsfile을 생성하고 파일을 Git과 같은 소스 관리 레파지토리에서 관리하는 것이 best practice이다.

### Creating a Jenkinsfile

Jenkinsfile은 Jenkins 파이프라인의 정의를 포함하고 있는 텍스트 파일이다. 이 뒤로 나오는 CD(Continuous delivery) 파이프라인의 기본 3단계를 고려해봐라.

#### Jenkinsfile (Declarative Pipeline)

```jenkinsfile
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```

모든 파이프라인에 이런 3단계가 있는 것은 아니지만, 대부분의 프로젝트에서 이것들을 정의하는 것은 좋은 시작방법이다. 아래 섹션에서는 간단한 파이프라인의 생성과 실행을 설명할 것이다.

이미 소스 컨트롤 레파지토리가 있고, 이와 같은 지시에 따라 젠킨스 파이프라인이 정의 되어있다고 가정한다.

Groovy syntax 하이라이팅을 지원하는 텍스트 에디터를 사용해서 프로젝트 루트에 새 Jenkinsfile을 만들어라.

위의 Declarative 파이프라인 예제에는 CD 파이프라인을 구현하는 데 필요한 최소한의 구조가 포함되어 있다. 필요한 agent directive는 Jenkins에게 파이프라인을 위한 실행자와 작업 공간을 할당하도록 시킨다. Agent Directive가 없으면, Declarative 파이프라인이 유효하지 않을 뿐만 아니라, 어떤 일도 할 수 없다. 기본적으로 agent directive는 원본 레포지토리가 checkout 되어 다음 단계에서 사용할 수 있도록 한다.
위의 선언적 파이프라인 예제에는 연속적인 공급 파이프라인을 구현하는 데 필요한 최소한의 구조가 포함되어 있다. 필요한 에이전트 지침은 Jenkins에게 Pipeline에 실행자 및 작업 공간을 할당하도록 지시한다. 에이전트 지침이 없으면 선언 파이프라인도 유효하지 않을 뿐만 아니라 어떤 일도 할 수 없을 것이다! 기본적으로 에이전트 지침은 원본 리포지토리가 체크아웃되어 후속 단계의 단계에 사용할 수 있도록 보장한다.

stages, steps 라는 지시어는 유효한 declarative 파이프라인을 위해 필요하다.(어느 단계에서 무엇을 실행하는지를 설명하는..)

Scripted 파이프라인으로 작성된 더 고급 사용의 경우에는 예제의 맨 위에 있는 `node`라는 것이 실행자와 작업공간을 할당하기 위한 중요한 첫 단계이다. 애초에 node 없이는 파이프라인은 아무 일도 할 수 없다. 노드 내에서 첫 번째 일은 프로젝트의 소스 콛르르 체크아웃하는 것이다. Jenkinsfile이 소스 컨트롤(git과 같은) 직접 pull되고 있기 때문에, 파이프라인은 이 파일을 통해 가장 최신의 리비전에 접근할 수 있게된다.

#### Jenkinsfile (Scripted Pipeline)

```jenkinsfile
node {
    checkout scm
    /* .. snip .. */
}
```

위에서 checkout step은 소스 컨트롤(git)에서 소스코드를 체크아웃 할 것이다. scm은 이번 파이프라인이 실행 될 때 트리거 한 특정 리비전을 클론하기 위한 특별한 변수이다.(그러니까 우리가 github push로 이 파이프라인을 트리거한 그 버전)

### Build

많은 프로젝트에서 파이프라인 작업의 시작은 build 단계이다. 일반적으로 이 단계에서는 소스코드의 조합, 컴파일이나 패키징 작업이 이루어진다. Jenkinsfile은 GNU/Make, Maven, Gradle 같은 빌드 툴을 대체하는 것이 아니라 프로젝트의 여러 개의 개발 라이프사이클(빌드, 테스트, 배포 등)을 하나로 모아주는 레이어라고 볼 수 있다.

Jenkins는 일반적인 어떤 빌드 도구들도 호출 할 수 있는 플러그인들을 가지고 있지만, 이 예제에서는 간단하게 sh로 호출 할 것이다. sh는 유닉스/리눅스 기반에서, 윈도우 기반에서는 bat을 사용하면 될 것이다. 우리가 실제로 gradle 프로젝트를 배포한다고 하면 gradle을 빌드 단계에서 사용하면 될 것이다.

Jenkins has a number of plugins for invoking practically any build tool in general use, but this example will simply invoke make from a shell step (sh). The sh step assumes the system is Unix/Linux-based, for Windows-based systems the bat could be used instead.

#### Jenkinsfile (Declarative Pipeline) - Build

```jenkinsfile
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
    }
}
```

이 단계는 make 명령을 호출하고 정상종료 되는 경우에만 계속 진행 된다. archiveArtifacts는 이 다음에 나오는 패턴(`**/target/*.jar`)과 일치하는 파일을 잡아서 나중에 찾을 수 있도록 jenkins 마스터에 저장한다.

Archiving artifacts는 nexus 같은 외부 artifacts 저장소를 대체하는 것이 아니고, 기본적인 report와 파일 보관에만 사용해야 한다.

### Test

자동화된 테스트를 실행하는 것은 CD의 중요한 요소이다. 젠킨스는 여러 플러그인이 제공하는 테스트 기록, report, visualization 기능을 많이 가지고 있다. 기본적으로 테스트가 실패하면, Jenkins가 이 테스트 실패에 대한 정보를 웹 UI에서 report와 visualization로 갖는 것이 유용하다. 아래 예시에서는 Junit plugin이 제공하는 Junit 단계를 이용한다.

자동화된 테스트를 실행하는 것은 성공적인 모든 연속적인 배송 과정의 중요한 요소다. 이와 같이 젠킨스는 다수의 플러그인이 제공하는 시험 기록, 보고 및 시각화 시설을 다수 보유하고 있다. 기본적인 수준에서, 테스트 실패가 있을 때, Jenkins가 웹 UI에서 보고와 시각화를 위한 실패를 기록하도록 하는 것이 유용하다. 아래 예에서는 JUnit 플러그인이 제공하는 Junit 단계를 사용한다.

아래 예시에서 테스트가 실패하면 웹 UI에서 노란 불로 표시된 것 처럼 파이프라인이 `unstable`로 표시된다. Jenkins는 여기서 기록된 Test report를 기반으로 시간 단위로 트렌드 분석과 시각화를 제공할 수 있다.

#### Jenkinsfile (Declarative Pipeline) - Test

```jenkinsfile
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                /* `make check` returns non-zero on test failures,
                * using `true` to allow the Pipeline to continue nonetheless
                */
                sh 'make check || true'
                junit '**/target/*.xml'
            }
        }
    }
}
```

#### Scripted pipeline(고급) 으로 전환 - Build

위 예시에서 `sh 'make check || true'` 부분에서는 테스트가 실패하더라도 항상 다음 스텝인 junit step이 진행되므로 테스트 보고서를 캡처하고 처리할 수 있다. 이렇게 하지 않기 위한 대안은 아래 Handling failure section에서 다룬다. junit step에서는 포함된 패턴(`*/target/*.xml`)과 일치하는 JUnit XML 파일을 캡처하고 연관짓는다.

### Deploy

배포는 프로젝트마다 용어에 대한 정의가 다를 수 있다.

여기 나오는 예제 파이프라인 단계에서는 deploy 단계는 이전 단계인 Build, Test단계가 성공적으로 완료되었다고 가정할 때만 실행될 것이고, 그 전에 실패했으면 파이프라인은 일찍 종료될 것이다.

#### Jenkinsfile (Declarative Pipeline) - Deploy

```jenkinsfile
pipeline {
    agent any

    stages {
        stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' 
              }
            }
            steps {
                sh 'make publish'
            }
        }
    }
}
```

#### Scripted pipeline(고급) 으로 전환 - Deploy

currentBuild.result 변수에 접근하면 파이프라인에서 테스트 실패 여부를 확인할 수 있다. 실패했을 경우에는 이 값은 사용할 수 없다. Jenkins 파이프라인에서 성공적으로 실행되었다고 가정하면, 각각의 파이프라인 실행은 관련된 build artifacts(테스트 결과, 전체 console output)를 저장한다.

Scripted pipeline은 위에 나타난 조건부 테스트, 루프, try/catch/finally 블럭과 다른 기능들을 포함할 수 있다. 다음 섹션에서 이 고급 스크립트로 작성된 파이프라인 syntax를 더 자세히 알아볼 것이다.

### 참고자료

- [Jenkinsfile docs](https://jenkins.io/doc/book/pipeline/jenkinsfile/)