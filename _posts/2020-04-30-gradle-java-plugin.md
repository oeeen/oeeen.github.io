---
layout: single
title:  "Gradle - Java Plugin"
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

컴파일 classpath에 연관된 모든 태스크들에 의존한다.
Depends on: All tasks which contribute to the compilation classpath, including jar tasks from projects that are on the classpath via project dependencies

Compiles production Java source files using the JDK compiler.

#### processResources — Copy

Copies production resources into the production resources directory.

#### classes

Depends on: compileJava, processResources

This is an aggregate task that just depends on other tasks. Other plugins may attach additional compilation tasks to it.

#### compileTestJava — JavaCompile

Depends on: classes, and all tasks that contribute to the test compilation classpath

Compiles test Java source files using the JDK compiler.

#### processTestResources — Copy

Copies test resources into the test resources directory.

#### testClasses

Depends on: compileTestJava, processTestResources

This is an aggregate task that just depends on other tasks. Other plugins may attach additional test compilation tasks to it.

#### jar — Jar

Depends on: classes

Assembles the production JAR file, based on the classes and resources attached to the main source set.

#### javadoc — Javadoc

Depends on: classes

Generates API documentation for the production Java source using Javadoc.

#### test — Test

Depends on: testClasses, and all tasks which produce the test runtime classpath

Runs the unit tests using JUnit or TestNG.

#### uploadArchives — Upload

Depends on: jar, and any other task that produces an artifact attached to the archives configuration

Uploads artifacts in the archives configuration — including the production JAR file — to the configured repositories. This task is deprecated, you should use one of the Ivy or Maven publishing plugins instead.

#### clean — Delete

Deletes the project build directory.

#### cleanTaskName — Delete

Deletes files created by the specified task. For example, cleanJar will delete the JAR file created by the jar task and cleanTest will delete the test results created by the test task

### SourceSet Task

For each source set you add to the project, the Java plugin adds the following tasks:

#### compileSourceSetJava — JavaCompile

Depends on: All tasks which contribute to the source set’s compilation classpath

Compiles the given source set’s Java source files using the JDK compiler.

#### processSourceSetResources — Copy

Copies the given source set’s resources into the resources directory.

#### sourceSetClasses — Task

Depends on: compileSourceSetJava, processSourceSetResources

Prepares the given source set’s classes and resources for packaging and execution. Some plugins may add additional compilation tasks for the source set.

### Lifecycle Tasks

The Java plugin attaches some of its tasks to the lifecycle tasks defined by the Base Plugin — which the Java Plugin applies automatically — and it also adds a few other lifecycle tasks:

#### assemble

Depends on: jar, and all other tasks that create artifacts attached to the archives configuration

Aggregate task that assembles all the archives in the project. This task is added by the Base Plugin.

#### check

Depends on: test

Aggregate task that performs verification tasks, such as running the tests. Some plugins add their own verification tasks to check. You should also attach any custom Test tasks to this lifecycle task if you want them to execute for a full build. This task is added by the Base Plugin.

#### build

Depends on: check, assemble

Aggregate tasks that performs a full build of the project. This task is added by the Base Plugin.

#### buildNeeded

Depends on: build, and buildNeeded tasks in all projects that are dependencies in the testRuntimeClasspath configuration.

Performs a full build of the project and all projects it depends on.

#### buildDependents

Depends on: build, and buildDependents tasks in all projects that have this project as a dependency in their testRuntimeClasspath configurations

Performs a full build of the project and all projects which depend upon it.

#### buildConfigName — task rule

Depends on: all tasks that generate the artifacts attached to the named — ConfigName — configuration

Assembles the artifacts for the specified configuration. This rule is added by the Base Plugin.

#### uploadConfigName — task rule, type: Upload

Depends on: all tasks that generate the artifacts attached to the named — ConfigName — configuration

Assembles and uploads the artifacts in the specified configuration. This rule is added by the Base Plugin.

### Project layout

The Java plugin assumes the project layout shown below. None of these directories need to exist or have anything in them. The Java plugin will compile whatever it finds, and handles anything which is missing.
