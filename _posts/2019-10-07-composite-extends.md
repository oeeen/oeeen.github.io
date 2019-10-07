---
layout: single
title:  "상속보다는 컴포지션을 사용하라"
date:   2019-10-07 22:00:00 +0900
classes: wide
categories: etc
tags: java
---

누군가에게 `상속과 composite`의 개념에 대해 듣고 정리를 하기 위해 여러 블로그들을 참조하고, `Effective Java 3/E Item 18. 상속보다는 컴포지션을 사용하라.` 의 내용 정리입니다.

생각을 해보면, 상속과 컴포지션은 정확하게 분리해서 사용한다면 아예 다른 의미로 사용할 수 있는 것 같습니다만. 일단은 정리를 해둡니다.

## 상속

메서드 호출과 달리 상속은 캡슐화를 깨뜨린다. - 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.

상속은 상위 클래스와 하위 클래스가 순수한 IS-A 관계일 때만 써야 한다. - 그러나 순수한 IS-A 관계여도 안심해서는 안된다.(캡슐화를 깨뜨리기 때문)

만약 하위 클래스의 패키지가 상위 클래스와 다르고 상위 클래스가 확장을 고려해서 설계되지 않았다면, 상속을 다시 고려해보아야 한다.

자바 기본 라이브러리에 있는 Stack이나, Properties도 IS-A 관계를 위반한 클래스이다. Stack IS A Vector 관계가 성립하지 않는다. 하지만 문제를 바로잡기에는 너무 늦어버려서 바꿀 수 없게 되었다.

![Stack](/assets/img/composition/stack.png)

***상위 클래스가 확장을 고려했고, 문서화도 잘 된 클래스**라면 안전하다. 상속의 취약점을 피하려면 상속 대신 Composition과 Forwarding을 사용하자.

### **상속을 고려했고, 문서화도 잘 된 클래스**

1. 메서드를 재정의하면 어떤 일이 일어나는지 정확히 정리하여 문서로 남겨야한다.
   - 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야한다.
   - 클래스 내부에서 스스로의 메서드를 어떻게 사용하는지 문서로 남겨야한다.(어떤 순서로 호출하는지, 각각의 호출 결과는 어떻게 되는지)
   - 일단 문서화 한 것은 클래스가 쓰이는 한 반드시 지켜야한다. (그렇지 않으면 하위 클래스의 오동작을 만들 수 있다.)
2. 상속용으로 설계한 클래스는 하위 클래스를 만들어서 검증해야 한다.

## 컴포지션

아래는 상속관계와 컴포지션 관계를 한번에 이해할 수 있는 코드이다.

```java
class Engine {} // The Engine class.

class Automobile {} // Automobile class which is parent to Car class.

class Car extends Automobile { // Car is an Automobile, so Car class extends Automobile class.
  private Engine engine; // Car has an Engine so, Car class has an instance of Engine class as its member.
}
```

`Car IS-A Automobile`이기 때문에 상속 관계, `Automobile HAS-A Engine`이기 때문에 컴포지션(구성?) 관계라고 할 수 있다.

## 상속과 컴포지션

기존에 프로젝트 진행하면서 있던 코드를 가져와보면 아래와 같다.

```java
public class Article extends BaseEntity implements Comparable<Article> {

    @Embedded
    private ArticleFeature articleFeature;

    @OnDelete(action = OnDeleteAction.CASCADE)
    @ManyToOne
    private User author;

    @Enumerated(EnumType.STRING)
    private OpenRange openRange;

    public Article(ArticleFeature articleFeature, User author, OpenRange openRange) {
        this.articleFeature = articleFeature;
        this.author = author;
        this.openRange = openRange;
    }

    public void update(ArticleFeature updatedArticleFeature) {
        articleFeature = updatedArticleFeature;
    }

    public boolean isSameUser(User user) {
        return this.author.equals(user);
    }

    @Override
    public int compareTo(Article article) {
        return this.getUpdatedTime().compareTo(article.getUpdatedTime());
    }
}
```

Article이 BaseEntity를 상속하는 관계이다. `Article IS-A BaseEntity`

그리고 `Article HAS-A ArticleFeature`, `Article HAS-A Author`, `Article HAS-A OpenRange`

Article과 ArticleFeature 간에 상속 관계가 없기 때문에, 상위 클래스의 변화 때문에 하위 클래스가 강제로 변하게 되는 상황 자체가 생기지 않는다.

만약 ArticleFeature 클래스가 변경되어 이상해지더라도(;;) Article 객체에서는 ArticleFeature 객체를 사용하지 않는 형태로 변경하면 된다.

극단적인 예시로 누군가가 설명해준 것이 있는데, 만약에 아래와 같은 코드가 있다면 재밌는 상황이 된다. (예시용 코드에 이펙티브 자바의 예시를 합쳤다.)

```java
public class 스마트폰 extends 전화기 {
    private int callCount;
    @Override
    public boolean call() {
        callCount++;
        super.call();
    }
    @Override
    public boolean callAll(Collection<? extends E> c) {
        callCount += c.size();
        ...super.callAll();
    }
}

public class 전화기 {
    public boolean call() {
        ...
    }
    public boolean callAll(Collection<? extends E> c) {
        ...call(); // iterate c
    }
    public void usePhone() {
        손으로 사용한다. -> 발로 사용해야 한다.
    }
}
```

이런 상황에서 전화기 클래스의 usePhone이 발로 전화를 해야 한다고 변경된다면, 그를 상속하는 스마트폰 클래스도 갑자기 발로 사용하게 되는 것이다.

또한, callAll() 내부에서 call() 메서드를 사용하고 있기 때문에, 스마트폰 클래스에서 callAll 메서드를 사용할 경우 callCount가 두배로 늘어나게 될 것이다. 하위 클래스에서 상위 클래스의 구현(자기 스스로의 메서드 사용)에 따라 하위 클래스의 구현에 영향을 주는 것이다. (이펙티브 자바의 예시)

## 그래서 상속을 사용할 때는

만약 컴포지션 대신 상속을 사용하기로 결정한다면, 다음과 같은 질문을 해봐야 한다.

1. 확장하려는 클래스의 API에 아무런 결함이 없는가?
2. 결함이 있다면, 이 결함이 여러분 클래스의 API까지 전파돼도 괜찮은가?

컴포지션으로는 이런 결함을 숨기는 새로운 API를 설계할 수 있지만, 상속은 상위 클래스의 결함을 그대로 상속 받는다.

## 참고자료

- Effective JAVA 3/E Item 18
- [https://stackoverflow.com/questions/2399544/difference-between-inheritance-and-composition](https://stackoverflow.com/questions/2399544/difference-between-inheritance-and-composition)
