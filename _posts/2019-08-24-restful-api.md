---
layout: single
title:  "RESTful API"
date:   2019-08-24 15:00:59 +0900
classes: wide
categories: web
tags: restful web rmm
toc: true
toc_sticky: true
---

전에 취업준비 할 때도 자기소개서에도 썼었고, 많이 들어보기만 했던 개념이지만 확실하게 무엇인지는 모르고 지나쳤고 정리한 적도 없었던 개념이다. 구글링하며 얻은 정보들을 정리 해 둔다.

## REST

REST는 REpresentational State Transfer의 약자이다. 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식이다.

엄격한 의미의 REST는 네트워크 아키텍처 원리의 모음이다. `네트워크 아키텍처 원리`란 자원을 정의하고 자원에 대한 주소를 지정하는 방법 전반을 일컫는다.

간단한 의미로는, 웹 상의 자료를 HTTP위에서 SOAP([Simple Object Access Protocol](https://ko.wikipedia.org/wiki/SOAP))이나 쿠키를 통한 세션 트랙킹 같은 별도의 전송 계층 없이 전송하기 위한 아주 간단한 인터페이스를 말한다.

* 출처: [REST - wiki](https://ko.wikipedia.org/wiki/REST)

무슨 뜻인가 생각해보면..

* 자원을 이름으로 구분하여 해당 자원의 상태를 주고 받는 모든 것.
* HTTP 프로토콜을 그대로 활용하기 때문에 웹의 장점을 그대로 활용할 수 있다.
* HTTP URI를 통해 자원을 명시하고 HTTP Method를 통해 해당 자원의 행동을 나타낸다. 모든 자원에 고유한 ID를 부여한다.

그래서 RESTful API는 REST의 특징을 가진 API라고 할 수 있다. REST한 구조에 맞는 API 인 듯 하다.

## REST 구성 요소

1. 자원(Resource): URI
   1. 모든 자원에 고유한 ID가 존재.
   2. /groups/:group_id 형식
2. 행위(Verb): HTTP Method
   1. GET, POST, PUT, DELETE
3. 표현(Representation of Resource)
   1. Client가 자원의 상태에 대한 조작 요청을 하면 Server는 적절한 응답(Representation)을 보낸다.
   2. 하나의 자원은 JSON, XML, TEXT, RSS 등 여러 형태의 Representation으로 나타낸다.

## REST 특징

1. Server-Client
   1. Server: 자원이 있는쪽, API를 제공, 비즈니스 로직 처리
   2. Client: 자원 요청하는 쪽, 사용자 인증, 세션, 로그인 정보를 직접 관리
2. Stateless
   1. HTTP가 기본적으로 stateless protocol이다.
   2. Client의 context를 서버에 저장하지 않는다.
   3. 각각의 요청을 아무런 연관관계 없이 별개의 것으로 처리한다.
3. Cacheable
   1. HTTP의 캐싱 기능을 적용할 수 있다.
      1. 불필요한 데이터 전송을 줄인다.
      2. origin server에 대한 request를 줄여준다.
      3. 응답 시간이 빨라진다.
4. Layered System
   1. REST Server는 다중 계층으로 구성될 수 있다.
   2. Proxy, Gateway와 같은 네트워크 기반의 미들웨어를 사용할 수 있다.
5. Code-On-Demand(optional)
   1. Server로부터 스크립트를 받아서 Client에서 실행한다.
6. Uniform Interface(인터페이스 일관성)
   1. URI로 지정한 Resource에 대한 조작을 일관된 인터페이스로 수행한다.
   2. 특정 언어나 기술에 종속되지 않고 HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용가능하다.

## Richardson Maturity Model

RMM 은 REST의 제약 사항을 잘 따르고 있는지 평가하기 좋다. RMM은 4단계(0 - 3)로 구성된다.

![rest-step](/assets/img/rest/rest_step.png)

> Level 0: Swamp of POX
Level 0 uses its implementing protocol (normally HTTP, but it doesn't have to be) like a transport protocol. That is, it tunnels requests and responses through its protocol without using the protocol to indicate application state. It will use only one entry point (URI) and one kind of method (in HTTP, this normally is the POST method). Examples of these are SOAP and XML-RPC.

이 모델의 시작점은 웹 메커니즘은 전혀 사용하지 않고 HTTP = 원격 통신을 위한 전송 시스템 으로 사용하는 것이다.
Remote Procedure Invocation에 기반한 원격 통신 메커니즘을 위한 터널링 메커니즘으로 HTTP를 사용하는 것이다.

![level0](/assets/img/rest/level0.png)

그냥 level0 에서는 단순히 병원에 예약하는 것과 동일 하게 생각 할 수 있다.

나는 병원에 예약을 하기 위해서 병원에 예약 가능한 시간을 알아야 한다.

나 -> 병원: 예약 가능한 시간 요청
병원 -> 나: 예약 가능한 시간 응답 (예를 들어 14시-14시 50분, 16시-16시 50분)
나 -> 병원: 14시-14시 50분 예약 요청
병원 -> 나: 예약 성공 했다고 응답

이런 식으로 단순히 통신 하는 것이다. 이렇게 하는 통신에는 xml, json, yaml, key-value 등 어떤 형식이든 가능하다.

> Level 1: Resources
When your API can distinguish between different resources, it might be level 1. This level uses multiple URIs, where every URI is the entry point to a specific resource. Instead of going through `http://example.org/articles`, you actually distinguish between `http://example.org/article/1` and `http://example.org/article/2`. Still, this level uses only one single method like POST.

Level 1에서는 리소스를 도입한다. 여기서는 요청을 단일 서비스 엔드포인트로 보내는 것이 아니라, 개별 리소스와 통신한다.

![level1](/assets/img/rest/level1.png)

```xml
POST /doctors/martin HTTP/1.1
[various other headers]

<openSlotRequest date = "2010-01-04"/>
```

level 0 에서 예약 가능 시간은 level 1에서는 이제 리소스이다.

```xml
HTTP/1.1 200 OK
[various headers]

<openSlotList>
    <slot id = "1234" doctor = "martin" start = "1400" end = "1450"/>
    <slot id = "5678" doctor = "martin" start = "1600" end = "1650"/>
</openSlotList>
```

이런 응답이 온다고 하면 이제 다음 요청은 이 slot이라는 resource로 요청을 보내는 것이다.

```xml
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
    <patient id = "seongmo"/>
</appointmentRequest>
```

예약이 성공하면 전과 비슷한 응답을 받는다.

```xml
HTTP/1.1 200 OK
[various headers]

<appointment>
    <slot id = "1234" doctor = "martin" start = "1400" end = "1450"/>
    <patient id = "seongmo"/>
</appointment>
```

> Level 2: HTTP verbs
To be honest, I don't like this level. This is because this level suggests that in order to be truly RESTful, your API MUST use HTTP verbs. It doesn't. REST is completely protocol agnostic, so if you want to use a different protocol, your API can still be RESTful.
> This level indicates that your API should use the protocol properties in order to deal with scalability and failures. Don't use a single POST method for all, but make use of GET when you are requesting resources, and use the DELETE method when you want to delete a resources. Also, use the response codes of your application protocol. Don't use 200 (OK) code when something went wrong for instance. By doing this for the HTTP application protocol, or any other application protocol you like to use, you have reached level 2.

![level2](/assets/img/rest/level2.png)

Level 2에서는 HTTP 메소드를 잘 사용한다.

무엇인가를 얻기 위해서는 GET을 쓴다.

GET /doctors/martin/slots?date=20190824&status=open HTTP/1.1

HTTP는 GET을 상태를 크게 변화시키지 않는 안전한 오퍼레이션이라고 본다. 그래서 어떤 순서로 언제 호출 되어도 매번 같은 결과를 얻을 수 있어야 한다. 그렇기 때문에 이제 캐싱을 할 수 있다.

이제 예약을 하려면 POST나 PUT이 필요하다. POST를 쓴다면 다음과 같다.

```xml
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
    <patient id = "seongmo"/>
</appointmentRequest>
```

이에 대한 응답은 이제 201 response code와 같이 온다.

```xml
HTTP/1.1 201 Created
Location: slots/1234/appointment
[various headers]

<appointment>
    <slot id = "1234" doctor = "martin" start = "1400" end = "1450"/>
    <patient id = "seongmo"/>
</appointment>
```

201은 클라이언트(나)가 나중에 그 리소스의 현재 상태를 GET 할 수 있도록 URI를 갖는 location 속성을 포함한다.

이 단계에서는 이처럼 HTTP Response code, HTTP Method를 사용한다.

> Level 3: Hypermedia controls
Level 3, the highest level, uses HATEOAS to deal with discovering the possibilities of your API towards the clients. More information about HATEOAS can be found below.

마지막 레벨은 HATEOAS(Hypertext As The Engine Of Application State)라는 보기 흉한 약어로 종종 언급되는 것을 도입한다. 그것은 예약을 하기 위해 무엇을 해야 할지 알기 위해 어떻게 빈 시간대 목록을 가져올지에 대한 문제를 해결한다.

![level3](/assets/img/rest/level3.png)

레벨 2에서 보냈던 GET과 동일한 GET 요청으로 시작한다.

`GET /doctors/martin/slots?date=20100104&status=open HTTP/1.1`

하지만 응답은 새로운 요소를 가지고 있다.

```xml
HTTP/1.1 200 OK
[various headers]

<openSlotList>
<slot id = "1234" doctor = "martin" start = "1400" end = "1450">
    <link rel = "/linkrels/slot/book" uri = "/slots/1234"/>
 </slot>
 <slot id = "5678" doctor = "martin" start = "1600" end = "1650">
    <link rel = "/linkrels/slot/book" uri = "/slots/5678"/>
 </slot>
</openSlotList>
```

예약 하는 방법을 알려주는 URI를 포함한 link rel을 가진다.

하이퍼미디어 컨트롤의 요점은 그것들이 다음에 무엇을 할 수 있는지와 그것을 위한 리소스의 URI를 알려주는 것이다.

이전에는 우리가 알아서 예약을 어떻게 보내는지 알아야 했다면, 이 단계에서는 하이퍼미디어 컨트롤이 그것을 알려준다.

예약 요청은 레벨 2와 동일하다.

```xml
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
    <patient id = "seongmo"/>
</appointmentRequest>
```

이 다음 응답은 더 복잡해진다. 이제 예약 이후에 할 수 있는 일들에 대한 하이퍼미디어 컨트롤이 포함된다.

```xml
HTTP/1.1 201 Created
Location: http://royalhope.nhs.uk/slots/1234/appointment
[various headers]

<appointment>
    <slot id = "1234" doctor = "seongmo" start = "1400" end = "1450"/>
    <patient id = "seongmo"/>
    <link rel = "/linkrels/appointment/cancel" uri = "/slots/1234/appointment"/>
    <link rel = "/linkrels/appointment/addTest" uri = "/slots/1234/appointment/tests"/>
    <link rel = "self" uri = "/slots/1234/appointment"/>
    <link rel = "/linkrels/appointment/changeTime" uri = "/doctors/martin/slots?date=20100104@status=open"/>
    <link rel = "/linkrels/appointment/updateContactInfo" uri = "/patients/seongmo/contactInfo"/>
    <link rel = "/linkrels/help" uri = "/help/appointment"/>
</appointment>
```

장점은

1. 서버가 클라이언트에 문제를 일으키지 않고 URI scheme을 변경할 수 있다.
2. 클라이언트 개발자가 프로토콜을 탐색할 수 있도록 돕는다.
3. 새로운 기능의 추가를 알리기 쉽다. (링크를 추가하면 된다.)

RMM이 REST의 요소가 무엇인지 생각하는 좋은 방법이지만 REST 그 자체의 레벨 정의는 아니다.

## RESTful 이란

* RESTful은 REST라는 아키텍처를 구현하는 웹 서비스를 나타내기 위해 사용되는 용어이다.
* 이해하기 쉽고 사용하기 쉬운 REST API를 만들어야 한다.
* 리소스 중심으로 디자인 되며, 클라이언트에서 access할 수 있는 모든 종류의 개체, 데이터, 서비스가 리소스에 포함된다.
* 각 리소스마다 해당 리소스를 고유하게 식별하는 ID가 있다. (ex. /orders/1)
* 해당 리소스에 대한 행동은 HTTP method로 표현한다, (GET /posts, POST /posts)

**리소스 URI를 컬렉션/항목/컬렉션 보다 더 복잡하게 요구하지 않는 것이 좋다.**

### REST URI 형태

CRUD | HTTP Method | Path
--- | --- | ---
Resource의 리스트 Read | GET | /resource
Resource 하나 Read | GET | /resource/:id
Resource create | POST | /resource
Resource 하나 Update | PUT | /resource/:id
Resource 하나 Delete | DELETE | /resource/:id

---

## 참고 자료

* [REST COOKBOOK](http://restcookbook.com/Miscellaneous/richardsonmaturitymodel/)
* [RMM - Martin Fowler](https://martinfowler.com/articles/richardsonMaturityModel.html)
* [RMM - 한글화 버전](https://jinson.tistory.com/190)
* [MSDN](https://docs.microsoft.com/ko-kr/azure/architecture/best-practices/api-design#organize-the-api-around-resources)
