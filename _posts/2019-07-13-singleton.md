---
layout: single
title:  "Multithread 환경에서 Singleton"
date:   2019-07-13 15:01:59 +0900
categories: etc
classes: wide
tag: java
---

## LazyHolder

```java
public class Singleton {
    private Singleton() {

    }

    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }

    private static class LazyHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
}
```

Singleton 클래스에서는 LazyHolder 클래스 변수가 없기 때문에, Singleton 클래스 로드 시에는 LazyHolder는 로드되지 않는다.  

그리고 getInstance() 실행 시에 LazyHolder.INSTANCE 를 참조하면 그때 LazyHolder Class가 로드 되어 초기화가 진행 된다.  

Class 로드 하고 초기화 하는 시점은 thread-safe를 보장하기 때문에 synchronized 키워드를 쓰지 않아도 된다.
