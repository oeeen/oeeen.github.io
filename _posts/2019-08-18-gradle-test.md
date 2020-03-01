---
layout: post
title:  "Gradle로 빌드 및 배포 시 Test"
date:   2019-08-18 01:00:59 +0900
classes: wide
categories: etc
tags: gradle
toc: true
toc_sticky: true
---

## 빌드 시에 Test를 모두 돈 후에 빌드가 진행 되도록 해보자

jenkins를 활용해서 우리의 웹 어플리케이션을 배포 해봤다.

근데 지금 상태에서는 단순히 빌드만 돌고 우리가 열심히 짰던 테스트 코드들이 돌지 않았다..

build.gradle에 간단하게 추가해줘서 테스트를 돌게 했다.

```groovy
test {
    useJUnitPlatform()
}

testImplementation 'org.junit.jupiter:junit-jupiter-api'
testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine'
```

테스트 코드에서 `@Tag("no")` 이런 식으로 태그를 줄 수 있고, 이 태그에 따라 빌드 시에 테스트를 할 지 안할 지도 설정할 수 있는 것 같다.

```java
// Test Code
public class CalculatorJUnit5Test {
    @Tag("slow")
    @Test
    public void testAddMaxInteger() {
        assertEquals(2147483646, Integer.sum(2147183646, 300000));
    }
  
    @Tag("fast")
    @Test
    public void testDivide() {
        assertThrows(ArithmeticException.class, () -> {
            Integer.divideUnsigned(42, 0);
        });
    }
}

// build.gradle
test {
    useJUnitPlatform {
        includeTags 'fast'
        excludeTags 'slow'
    }
}
```

이런 식으로 하면 slow는 안돌고 fast만 테스트를 수행할 수 있다.

gradle에 대해 조금 더 공부해보면 더 재밌는 일들을 많이 할 수 있을 것 같다!

참고 링크: [baeldung(junit-5-gradle)](https://www.baeldung.com/junit-5-gradle)
