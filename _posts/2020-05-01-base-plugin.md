---
layout: single
title:  "Gradle - Base Plugin"
date:   2020-05-01 18:50:59 +0900
classes: wide
categories: etc
tags: gradle plugin
toc: true
toc_sticky: true
---

Gradle 공식 문서의 Base Plugin 부분을 번역한 내용입니다. 제 개인의 의견 및 오역이 있을 수 있습니다. 잘못된 부분이나 다른 의견이 있으시면 지적과 관심 부탁드립니다.

## The Base Plugin

Base Plugin은 대부분의 빌드에 공통의 몇 가지 태스크와 컨벤션을 제공하고, 빌드 실행 방식의 일관성을 증진하는 구조를 빌드에 추가한다. 가장 중요한 점은 다른 플러그인과 빌드 작성자가 제공하는 more specific한 작업에 대한 포괄적인 역할을 하는 라이프 사이클 태스크의 집합이다.

### Usage

build.gradle

```groovy
plugins {
    id 'base'
}
```

build.gradle.kts

```kotiln
plugins {
    base
}
```

### Tasks

#### clean — Delete

빌드 디렉토리와 그 안에 있는 모든 것을 지운다. 즉, `Project.getBuildDir()`에 의해 특정되는 경로 안에 있는 모든 것을 지운다.

#### check — lifecycle task

플러그인과 빌드 작성자는 여기에 verification tasks(running test같은)를 넣어야한다. (`check.dependsOn(task)`를 이용해서)

#### assemble — lifecycle task

플러그인과 빌드 작성자는 이 라이프사이클 태스크에 배포와 다른 소모성 아티팩트를 생성하는 태스크를 넣어야한다. 예를 들어 jar는 Java 라이브러리를 위한 소모성 아티팩트를 생성한다. 이 라이프사이클 태스크를 assemble.dependsOn(task)를 이용해서 넣어야한다.

#### build — lifecycle task

Depends on: check, assemble

모든 테스트 실행, 생산 아티팩트 제작, 문서 생성 등 모든 것을 구축하기 위한 것이다. 조립과 점검이 일반적으로 더 적절하기 때문에 콘크리트 작업을 직접 부착하는 경우는 거의 없을 것이다

모든 테스트 실행, 프로덕션 아티팩트 생성, 문서 생성 등 모든 것을 빌드하기 위한 것이다. assemble과 check가 일반적으로 더 적절하다.

#### buildConfiguration — task rule

Configuration에 들어가는 이름의 아티팩트를 생성하기 위해 필요한 모든 태스크를 실행한다. 예를 들어 buildArchive는 arhive 설정에 첨부된 아티팩트를 생성하기 위해 필요한 모든 태스크를 실행한다.

#### uploadConfiguration — task rule

buildConfiguration이랑 하는 일이 똑같다. 대신 주어진 설정에 첨부된 모든 아티팩트를 업로드한다.

#### cleanTask — task rule

특정 태스크의 결과물을 제거한다. 예를 들어 cleanJar는 jar 태스크에 의해 생성된 JAR 파일을 제거한다.

### Dependency Management

Base Plugin은 종속성을 위해 아무런 설정도 추가하지 않지만, 아티팩트를 위해 다음의 설정들을 추가한다.

#### default

사용자의 프로젝트에 사용되는 예비 구성. 예를 들어 프로젝트 A에 의존하는 프로젝트 B를 가지고 있다고 해보자. Gradle은 프로젝트 A의 아티팩트와 의존성 중 어떤 것이 B의 specified configuration에 추가되는 지를 결정하기 위해 내부의 어떤 로직을 사용한다. 다른 요소가 적용되지 않는다면, Gradle은 프로젝트 A의 `default` 구성의 모든 것을 사용한다.

새로운 빌드와 플러그인은 `default` 설정을 사용하지 마십시오. 이 설정은 과거와의 호환성을 위해 남아있는 것입니다.

#### archives

프로젝트의 프로덕션 아티팩트를 위한 표준 설정이다.

`assemble` 태스크는 `archives` 설정에 첨부된 모든 아티팩트를 생성한다.

**artifacts**는 다음과 같은 식으로 사용하는 어떤 것..

```groovy
task myTask(type:  MyTaskType) {
    destFile = file("$buildDir/somefile.txt")
}

artifacts {
    archives(myTask.destFile) {
        name 'my-artifact'
        type 'text'
        builtBy myTask
    }
}
```

### Conventions

Base Plugin은 ZIP, TAR, JAR와 같은 아카이브 작성과 관련된 컨벤션만 추가한다. 특히 다음과 같이 설정할 수 있는 프로젝트 속성을 제공한다.

#### archivesBaseName — default: $project.name

아카이브 태스크에 대한 default `AbstractArchiveTask.getArchiveBaseName()`를 제공한다.

#### distsDirName — default: distributions

Distribution archive(JAR가 아닌)를 위한 디렉토리의 default name
Default name of the directory in which distribution archives, i.e. non-JARs, are created.

#### libsDirName — default: libs

라이브러리 archive(JAR)를 위한 디렉토리의 default name

**플러그인은 AbstractArchiveTask를 상속받는 모든 태스크에서 다음 특성의 기본 값을 제공한다.**

#### destinationDirectory

Non-JAR archive를 위한 `$buildDir/$distsDirName` 와 JAR archive를 위한 `$buildDir/$libsDirName`

#### archiveVersion

기본적으로 `$project.version`이고 프로젝트가 버전이 없다면, `'unspecified'`

#### archiveBaseName

`$archivesBaseName`의 기본값

### 원본 출처

- [gradle 문서 - base plugin](https://docs.gradle.org/current/userguide/base_plugin.html)
