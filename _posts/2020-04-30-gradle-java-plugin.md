---
layout: single
title:  "Gradle - Java Plugin(작성중)"
date:   2020-04-30 18:50:59 +0900
classes: wide
categories: etc
tags: gradle plugin
toc: true
toc_sticky: true
---

Gradle 공식 문서의 Java Plugin 부분을 번역한 내용입니다. 제 개인의 의견 및 오역이 있을 수 있습니다. 잘못된 부분이나 다른 의견이 있으시면 지적과 관심 부탁드립니다.

## The Java Plugin

Java Plugin은 프로젝트에 테스트 및 여러 기능과 함께 자바 컴파일을 추가한다. JVM 언어의 Gradle plugin의 기반이 된다.

### Usage

Java Plugin 을 사용하기 위해서는 아래 빌드 스크립트 처럼 작성하면 된다.

build.gradle

```groovy
plugins {
    id 'java'
}
```

build.gradle.kts

```kotlin
plugins {
    java
}
```

### Tasks

자바 플러그인은 프로젝트에 다양한 아래 나와있는 task들을 추가할 수 있다.

#### compileJava — JavaCompile

Depends on: 컴파일 클래스패스의 모든 tasks(프로젝트 종속성을 통한 클래스패스에 있는 프로젝트의 jar task를 포함한 모든)

JDK 컴파일러를 이용하여 프로덕션 자바 소스파일을 컴파일 한다.

#### processResources — Copy

프로덕션 리소스를 프로덕션 리소스 디렉토리에 복사

#### classes

Depends on: compileJava, processResources

단순히 다른 태스크에 의존하는 총체적인 태스크다. 다른 플러그인은 추가 컴파일 태스크에 그것을 첨부할 수 있다.(?)

#### compileTestJava — JavaCompile

Depends on: classes, 테스트 컴파일 클래스 패스에 있는 모든 태스크

JDK 컴파일러를 이용하여 테스트 자바 소스파일을 컴파일 한다.

#### processTestResources — Copy

테스트 리소스를 테스트 리소스 디렉토리에 복사

#### testClasses

Depends on: compileTestJava, processTestResources

단순히 다른 태스크에 의존하는 총체적인 태스크다. 다른 플러그인은 추가 테스트 컴파일 태스크에 그것을 첨부할 수 있다.

#### jar — Jar

Depends on: classes

메인 소스셋에 있는 클래스와 리소스를 기반으로 프로덕션 JAR 파일을 모은다.

#### javadoc — Javadoc

Depends on: classes

Javadoc을 이용하여 프로덕션 자바소스로부터 API 문서를 생성한다.

#### test — Test

Depends on: testClasses, 테스트 런타임 클래스패스를 생성하는 모든 태스크

JUnit 이나 TestNG를 사용한 유닛 테스트를 실행한다.

#### ~~uploadArchives — Upload~~

Depends on: jar, 아카이브 설정에 첨부된 아티팩트를 생성하는 다른 태스크

Deprecated

#### clean — Delete

프로젝트 빌드 디렉토리를 지운다.

#### cleanTaskName — Delete

특정한 태스크로부터 만들어진 파일을 지운다. 예를 들어 cleanJar를 하면 jar task를 통해 만들어진 JAR 파일을 지운다. cleanTest를 하면 test task를 통해 만들어진 테스트 결과를 지운다.

#### SourceSet Task

프로젝트에 추가하는 각 소스 셋에 대해 자바 플러그인은 다음의 태스크들을 추가한다.

1. compileSourceSetJava — JavaCompile
   - Depends on: SourceSet의 컴파일 클래스패스에 기여하는 모든 태스크
   - JDK 컴파일러를 사용하여 주어진 SourceSet의 자바 소스파일을 컴파일한다.
2. processSourceSetResources — Copy
   - 주어진 SourceSet의 리소스를 리소스 디렉토리로 복사한다.
3. sourceSetClasses — Task
   - Depends on: compileSourceSetJava, processSourceSetResources
   - 주어진 SourceSet의 클래스와 리소스를 패키징과 실행을 위해 준비한다. 어떤 플러그인은 SourceSet을 위한 추가 컴파일 태스크가 추가될 수 있다.

#### Lifecycle Tasks

Java 플러그인은 Java 플러그인이 자동으로 적용하는 [*Base Plugin](https://smjeon.dev/etc/base-plugin)에서 정의한 라이프사이클 태스크에 해당 태스크의 일부를 첨부하고, 다음에 나오는 몇 가지 다른 라이프사이클 태스크도 추가한다.

1. assemble
   - Depends on: jar, 아카이브 설정에 첨부된 아티팩트를 만드는 모든 다른 태스크
   - 프로젝트 내의 모든 기록물을 모으는 총체적인 태스크다. 이 태스크는 Base Plugin에 의해 추가된다.
2. check
   - Depends on: test
   - verification tasks들이 수행하는 총체적인 태스크다.(테스트를 돌리는 것 같은..) 어떤 플러그인은 자체의 검증 태스크를 추가할 수도 있다. 전체 빌드에 커스텀 테스트 태스크를 실행하려면 이 태스크에 연결하면 된다. 이 태스크는 Base Plugin에 의해 추가된다.
3. build
   - Depends on: check, assemble
   - 프로젝트의 전체 빌드를 수행하는 총체적인 태스크다. Base Plugin에 의해 추가된다.
4. buildNeeded
   - Depends on: testRuntimeClasspath 설정의 디펜던시가 있는 모든 프로젝트의 build, buildNeeded task
   - 의존하고 있는 모든 프로젝트 빌드를 수행
5. buildDependents
   - Depends on: 모든 프로젝트 내의 testRuntimeClasspath 설정의 디펜던시에 의해 가지는 모든 프로젝트의 `build`와 `buildDependents`
   - 프로젝트와 그것에 의존하는 모든 프로젝트의 풀 빌드 수행.
   - 위에 buildNeeded랑 뭐가 다른건지 잘 모르겠음.
6. buildConfigName — task rule
   - Depends on: ConfigName으로 첨부된 아티팩트를 생성하는 모든 태스크
   - 특정 설정을 위한 아티팩트를 모은다. Base Plugin에 의해 추가된다.
7. uploadConfigName — task rule, type: Upload
   - Depends on: ConfigName으로 명명되어 첨부된 아티팩트를 생성하는 모든 태스크
   - 특정 설정을 위한 아티팩트를 모으고 업로드 한다. Base Plugin에 의해 추가된다.

### Project layout

The Java plugin assumes the project layout shown below. None of these directories need to exist or have anything in them. The Java plugin will compile whatever it finds, and handles anything which is missing.
