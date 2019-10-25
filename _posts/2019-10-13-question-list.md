---
layout: single
title:  "1 일 1 질문"
date:   2019-10-13 23:00:59 +0900
classes: wide
categories: etc
tags: web
---

**틀린 내용이나 본문의 내용과 다른 의견이 있으시면 댓글로 남겨주세요!**

## 의식적인 노력을 통해 하루에 질문을 한개씩

하루에 질문을 한개씩 할 수 있도록 강제하기 위해 쓰는 글입니다.

### 10/25

- JVM에서 GC가 일어날 때 한 트랜잭션이 끝났는지 어떻게 확인하고 Stop-The-World 하는가?
  - GC가 돌 때 나머지 Thread들의 상태를 저장하고, GC가 돈다.
  - 하지만 Transaction 관점에서 보면, 만약 DB에 insert하는 transaction 중간에 GC가 끼게된다면, timeout이 발생할 가능성은 생긴다.
  - GC의 정책이 변경된다고 했을 때 예상치 못한 에러가 발생할 가능성이 있다.

### 10/22

- JPA FetchType과 프록시 객체
  - 프록시는 원본 객체를 상속받아서 만들어진다.
  - 영속성 컨텍스트는 영속 엔티티의 동일성을 보장해야한다.
  - 처음 조회할 때 프록시 객체였다면, 다음 조회할 때도 프록시 객체가 나와야 한다.(동일성 보장)

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@Getter
@Entity
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

이 상황에서 ArticleService.findById(Long id) 호출 시 Article이 영속성 컨텍스트에 등록되면서, Article 내부의 author도 영속성 컨텍스트에 들어간다.

현재 코드에서는 ManyToOne이라서 FetchType.EAGER로 동작해서 프록시 객체로 생성되지 않지만, 테스트를 위해 아래와 같이 LAZY로 수정 해본다.

```java
@OnDelete(action = OnDeleteAction.CASCADE)
@ManyToOne(fetch = FetchType.LAZY)
private User author;
```

그럴 경우 Article 조회 시 User author는 프록시 객체로 1차 캐시에 들어간다. 그 다음에 해당하는 User를 조회해오면 영속성 컨텍스트는 동일성을 보장해야하기 때문에 똑같이 프록시 객체로 조회가 된다.(프록시 타겟의 식별자가 같은) 따라서 이 경우에는 equals를 구현 할 때 IDE에서 자동 생성해주는 equals를 사용하는 것이 아니라 instanceof를 사용해서 비교해야 한다.(프록시는 원본 엔티티의 자식 타입이므로)

- 초기화된 프록시를 비교할 때는 instanceof를 사용, 프록시에 user.id 말고 user.getId()을 사용하자. 프록시의 데이터를 조회할 때는 getter를 쓰자.

### 10/19 (혼자 알아본 것)

REST는 긴 시간을 거쳐 진화하는 웹 애플리케이션을 위한 것이다.

- REST - How do I improve HTTP without breaking the Web
  - 분산 하이퍼미디어 시스템(예: 웹)을 위한 아키텍쳐 스타일(제약조건의 집합)
  - Client-Server / Stateless / Cache / Uniform interface / Layered System / Code-on-demand
  - Uniform interface - 서버와 클라이언트가 각각 독립적으로 진화하기 위해서.
  - 우리의 REST API들을 아래 내용들을 가장 많이 지키지 못하고 있다.
    - Self-Descriptive Messages
      - 메세지의 내용만으로 메세지가 뭘 하는지 알수 있어야한다.
    - HATEOAS
      - 애플리케이션의 상태는 Hyperlink를 통해서 전이되어야 한다.
- REST API는?
  - REST 아키텍쳐 스타일을 따르는 API

```html
GET /todos HTTP/1.1
Host: smjeon.dev

HTTP/1.1 200 OK
Content-Type: text/html

<html>
<body>
<a href="https://todos/1">REST 정리하기</a>
<a href="https://todos/2">REST API 정리하기</a>
</body>
</html>
```

위와 같은 HTML을 보면 self-descriptive, HATEOAS 하다는 것을 알수 있다.

```json
GET /todos HTTP/1.1
Host: smjeon.dev

HTTP/1.1 200 OK
Content-Type: application/json

[
    {"id": 1, "title": "REST 정리하기"},
    {"id": 2, "title": "REST API 정리하기"}
]
```

그러나 우리가 구현하는 API의 경우에는 위와 같은 식으로 나오지만, 여기서 json 문서에서 id는 뭐고 title이 무엇인지 확인할 수 없다. HATEOAS 측면에서는 링크가 없어서.. HATEOAS를 만족시킬 수 없다.

- Self-descriptive - 확장가능한 커뮤니케이션
- HATEOAS - 애플리케이션 상태 전이의 late binding, 링크는 동적으로 변경될 수 있다.

```json
GET /todos HTTP/1.1
Host: smjeon.dev

HTTP/1.1 200 OK
Content-Type: application/json
Link: <https://example.org/docs/todos>; rel="profile"

[
    {"id": 1, "title": "REST 정리하기"},
    {"id": 2, "title": "REST API 정리하기"}
]
```

이런 식으로 Self-descriptive 를 만족 시킬 수도 있다.

- 참고 자료: [그런 REST API로 괜찮은가](https://www.youtube.com/watch?v=RP_f5dMoHFc&list=WL&index=7&t=1835s)

### 10/18

- 회사에서 업무 진행하면서 기존 레거시 코드에 대한 리팩토링 시간을 따로 가질 수 있을까?
  - 팀 바이 팀이겠지만, 따로 리팩토링을 할 시간을 주지는 않는다. 반드시 고치고 싶은 코드들이 있다면 내가 더 노력해서 바꿔보자

- Web server에서 static file을 serving할 때 WAS의 요청과 어떻게 구분할 수 있을까?
  - web server 단에서 설정하기 나름이다. (내 생각이지만, 현재는 web server와 was를 모두 내가 구현하기 때문에 일종의 규약 같은게 필요 없지만 팀 단위로 진행되는 프로젝트에서는 일종의 컨벤션처럼 정적파일 서빙에 대한 처리를 규약으로 가지고 있어야 할 것 같다.)
  - 그러면 /resource/*에서 찾는 정적파일과 내가 /resourse/test 와 같은 api call을 만들었다면 이 둘을 어떻게 구분할 수 있을까?

### 10/17

- Java의 Heap은 왜 Heap일까?
  - [Stack overflow](https://stackoverflow.com/questions/1699057/why-are-two-different-concepts-both-called-heap)
  - Heap 자료구조와는 관련이 없다.
  - 위키피디아에서는 Lisp이 Memory store를 구현하기 위해 heap 자료구조를 사용했기 때문에.. 그렇게 됐다.(어떻게 했다고는 나와있지 않다.)
  - 메모리 힙은 세탁물 바구니의 heap of clothes와 같은 느낌으로 부른다. 그러니까 메모리가 할당되고 해제되는 messy한 공간을 가리키기 위해 사용된다.

### 10/16

- Java에서 ArrayList
  - ArrayList
    - default capacity: 10이다. MAX_ARRAY_SIZE: MAX_INT - 8 이다.
    - ArrayList에 add 할 때 공간이 부족하면 현재 있는 값들을 복사하고 add한다.
    - 공간이 부족하면 기존 사이즈의 1.5배를 해서 새로운 Array를 만들고 값을 복사한다.
    - add에서 동작을 보면 thread-safe 하지 않다.

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

ArrayList에서 사이즈가 바뀔 때 Object[]이 바뀌는지 확인하기 위한 테스트로 아래와 같은 코드를 수행한다.

```java
@Test
void arrayTest() {
    List<Integer> testArrayList = new ArrayList<>();
    for (int i = 0; i < 15; i++) {
        testArrayList.add(i);
    }
}
```

그리고 ArrayList의 grow 메서드에 break point를 찍고 확인해보면, 볼 수 있다. (사이즈를 지정하지 않으면 default size는 10이다.) 사이즈가 10보다 커질 때 grow method가 실행되는데, grow method가 실행된 후 size는 기존 사이즈의 1.5배만큼으로 늘어난다. (`int newCapacity = oldCapacity + (oldCapacity >> 1);`) 그러면서 Object array를 늘어난 크기로 새로 만들고 기존 데이터들을 복사한다.

Default Size 10까지는 변화 없이 하나씩 요소들을 추가 한다.

![Size 10](/assets/img/question_list/size10.png)

10보다 커지는 순간 grow method가 실행 되면서 새로운 Object array를 만들고 여기에 기존에 값들을 복사한다.

![Size 15](/assets/img/question_list/size15.png)

### 10/15

- Java Stream
  - 스트림은 중간 처리 단계(map, filter 같은)와 최종 단계(foreach, reduce 같은)로 구성된다.
  - 중간 처리 단계(intermediate) 를 거치면 새로운 스트림이 나온다.(항상 Lazy) 파이프라인의 최종 단계(terminal)가 실행 될 때까지 파이프라인 소스는 시작되지 않는다.
  - 최종 단계를 거치면 스트림 파이프라인은 소모되었다고 생각되어서 더이상 쓸 수 없다.
  - 동일한 데이터 소스를 다시 통과해야 하면, 데이터 소스에서 새 스트림을 얻어야 한다.
  - 최종 단계는 대부분의 경우에 eager다. (iterator(), spliterator()만 아니다)
  - Parallelism
    - for-loop 처리는 본질적으로 serial 이다.
    - stream은 각 개별 요소에 대해 필수 작업 말고, aggregate operation의 파이프라인으로서 계산을 reframing해서 병렬 실행을 용이하게 한다?
    - 모든 스트림작업은 직렬 또는 병렬로 실행할 수 있다.
    - Collection.stream(): 직렬 스트림, Collection.parallelStream(): 병렬 스트림
    - 직렬, 병렬 스트림의 차이는 terminal operation이 시작되면 스트림 파이프라인은 직렬 또는 병렬로 수행된다.
    - findAny() 같은 nondeterministic 한 것이 아니면, 직렬인지 병렬인지의 차이는 계산 결과에 영향이 없어야 한다.
  - 박싱 주의 (기본형의 경우 IntStream, LongStream, DoubleStream 사용)
  - Stream source에 따라 병렬화에 좋을 수도 있고, 좋지 않을 수도 있다.(예를 들어 LinkedList는 분해에 좋지 않다.)

```java
ArrayList<String> results = new ArrayList<>();
stream.filter(s -> pattern.matcher(s).matches())
      .forEach(s -> results.add(s));
```

위 코드에서 ArrayList는 thread-safe하지 않기 때문에 원하는 대로 결과가 나오지 않을 수 있다.(무슨 의미인지 눈으로 확인해본 적이 없어서 잘 모르겠다.) 대신 아래와 같은 코드로 구현하면 안전하다.

```java
List<String>results =
         stream.filter(s -> pattern.matcher(s).matches())
               .collect(Collectors.toList());
```

### 10/14

- Gateway
  - 서로 다른 통신망, 프로토콜을 사용하는 네트워크 간의 통신을 가능하게 하는 컴퓨터나, 소프트웨어.
  - 다른 네트워크로 들어가는 입구 역할을 하는 네트워크 포인트
  - 서로 다른 네트워크 상의 통신 프로토콜을 적절히 변환해주는 역할을 한다.
  - 하나 이상의 프로토콜을 사용하여 통신한다는 면에서 라우터, 스위치와는 다르다.
  - 비유하자면 외국인과 원활한 의사소통을 위해 통역사를 두는 것, 통역사(게이트웨이)는 외국인(다른 네트워크)과 소통을 위해 반드시 거쳐야하는 사람. 이라고 할 수 있다.
  - ~~라우터와 동일한 개념으로 이해할 수 있다.~~
    - ~~라우터는 패킷을 다른 네트워크로 보내주는 역할을 하면서 최적의 네트워크 경로를 찾아주는 역할을 한다.~~
  - ~~로컬 네트워크에는 게이트웨이가 필요 없지만,~~ 인터넷과 같은 다른 네트워크에 접근하려면 게이트웨이가 필요하다.

### 10/13

- [상속과 컴포지션](https://smjeon.dev/etc/composite-extends/)
