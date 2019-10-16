---
layout: single
title:  "1 일 1 질문"
date:   2019-10-13 23:00:59 +0900
classes: wide
categories: etc
tags: web
---

틀린 내용이나 본문의 내용과 다른 의견이 있으시면 댓글로 남겨주세요!

## 의식적인 노력을 통해 하루에 질문을 한개씩

하루에 질문을 한개씩 할 수 있도록 강제하기 위해 쓰는 글입니다.

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
  - 라우터와 동일한 개념으로 이해할 수 있다.
    - 라우터는 패킷을 다른 네트워크로 보내주는 역할을 하면서 최적의 네트워크 경로를 찾아주는 역할을 한다.
  - 로컬 네트워크에는 게이트웨이가 필요 없지만, 인터넷과 같은 다른 네트워크에 접근하려면 게이트웨이가 필요하다.

### 10/13

- [상속과 컴포지션](https://smjeon.dev/etc/composite-extends/)
