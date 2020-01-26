---
layout: post
title:  "HTTP Header"
date:   2019-08-25 20:00:59 +0900
classes: wide
categories: web
tags: http
toc: true
toc_sticky: true
---

## HTTP Header

HTTP 프로토콜의 리퀘스트와 리스폰스에는 반드시 메시지 헤더가 포함되어 있다. 메시지 헤더에는 클라이언트나 서버가 리퀘스트나 리스폰스를 처리하기 위한 정보가 들어 있다.

리퀘스트의 HTTP 메시지는 다음과 같다.

![Request Message](/assets/img/http/header.png)

HTTP header 필드는

`헤더 필드명: 필드 값` 으로 이루어져 있다.

** HTTP 헤더 필드가 중복 된 경우에는? 브라우저마다 행동이 달라진다..

## HTTP/1.1 헤더 필드

- 일반 헤더 필드

헤더 필드 명 | 설명
--- | ---
Cache-Control | 캐싱 동작 지정
Connection | Hop-by-hop 헤더, 커넥션 관리
Data | 메시지 생성 날짜
Pragma | 메시지 제어
Trailer | 메시지의 끝에 있는 헤더의 일람
Transfer-Encoding | 메시지 바디의 전송 코딩 형식 지정
Upgrade | 다른 프로토콜에 업그레이드
Via | 프록시 서버에 관한 정보
Warning | 에러 통지

- 리퀘스트 헤더 필드

헤더 필드 명 | 설명
--- | ---
Accept | 유저 에이전트가 처리 가능한 미디어 타입
Aceept-Charset | Charset 우선 순위
Accept-Encoding | 컨텐츠 인코딩 우선 순위
Accept-Language | 언어 우선 순위
Authorization | 웹 인증을 위한 정보
Expect | 서버에 대한 특정 동작의 기대
From | 유저의 메일 주소
Host | 요구된 리소스의 호스트
If-Match | 엔티티 태그의 비교
If-Modified-Since | 리소스의 갱신 시간 비교
If-None-Match | 엔티티 태그의 비교(If-Match의 반대)
If-Range | 리소스가 갱신되지 않은 경우에 엔티티의 바이트 범위의 요구를 송신
If-Unmodified-Since | 리소스의 갱신 시간 비교(If-Modified-Since의 반대)
Proxy-Authorization | 프록시 서버의 클라이언트 인증을 위한 정보
Range | 엔티티 바이트 범위 요구
Referer | 리퀘스트중의 URI를 취득하는 곳
TE | 전송 인코딩의 우선 순위
User-Agent | HTTP 클라이언트의 정보

- 리스폰스 헤더 필드

헤더 필드 명 | 설명
--- | ---
Accept-Ranges | 바이트 단위의 요구를 수신할 수 있는지 없는지 여부
Age | 리소스의 지정 경과 시간
Etag | 리소스 특정하기 위한 정보
Location | 클라이언트를 지정한 URI에 리다이렉트
Proxy-Authenticate | 프록시 서버의 클라이언트 인증을 위한 정보
Retry-After | 리퀘스트 재시행의 타이밍 요구
Server | HTTP 서버 정보
Vary | 프록시 서버에 대한 캐시 관리 정보
WWW-Authenticate | 서버의 클라이언트 인증을 위한 정보

- 엔티티 헤더 필드

헤더 필드 명 | 설명
--- | ---
Allow | 리소스가 제공하는 HTTP 메소드
Content-Encdoing | 엔티티 바디에 적용되는 컨텐츠 인코딩
Content-Language | 엔티티 언어
Content-Length | 엔티티 바디 사이즈
Content-Location | 리소스에 대응하는 대체 URI
Content-MD5 | 엔티티 바디의 메시지 다이제스트
Content-Range | 엔티티 바디의 범위 위치
Content-Type | 엔티티 바디 미디어 타입
Expires | 엔티티 바디 유효기간 날짜
Last-Modified | 리소스의 최종 갱신 날짜

비표준 헤더 필드는 [RFC4229 - HTTP Header Field Registrations](https://tools.ietf.org/html/rfc4229#section-2.1)에 정리되어 있다.

## 일반 헤더 필드

Request, Response 양쪽에서 사용되는 헤더다.

1. Cache-control
   1. 디렉티브로 불리는 명령을 사용하여 캐싱 동작을 지정한다.
2. Connection
   1. 프록시에 더 이상 전송하지 않는 헤더 필드를 지정
   2. 지속적 접속 관리
3. Date
   1. HTTP 메시지를 생성한 날짜를 나타낸다.
   2. RFC1123에 나와 있는 날짜 포맷은 Date: Sun, 25 Aug 2019 21:05:05 GMT
4. Pragma
   1. HTTP/1.1 보다 오래된 버전의 흔적으로 HTTP/1.0 후방 호환성만을 위해서 정의되어 있는 헤더 필드이다.
5. Trailer
   1. 메시지 바디의 뒤에 기술되어 있는 헤더 필드를 미리 전달할 수 있다. 이 헤더 필드는 HTTP/1.1에 구현되어 있는 청크 전송 인코딩을 사용하고 있는 경우에 사용 가능하다.
6. Transfer-Encoding
   1. 메시지 바디의 전송 코딩 형식을 지정하는 경우에 사용된다.
7. Upgrade
   1. HTTP 및 다른 프로토콜의 새로운 버전이 통신에 이용되는 경우에 사용된다.
8. Via
   1. 클라이언트와 서버 간의 Request or Response 메시지의 경로를 알기 위해서 사용된다.
   2. 프록시 혹은 게이트 웨이는 자신의 서버 정보를 Via 헤더 필드에 추가한 뒤에 메시지를 전송한다.
   3. 각각의 프록시 서버가 자기 자신의 정보를 Via 헤더에 추가한다.
9. Warning
   1. 캐시에 관한 문제의 경고를 유저에 전달한다.
   2. `Warning: [경고 코드][경고한 호스트:포트 번호]"[경고문]" ([날짜])`

## Upgrade Field

### Request

```request
GET /index HTTP/1.1
Upgrade: TLS/1.0
Connection: Upgrade
```

### Response

```response
HTTP/1.1 101 Switching Protocols
Upgrade: TLS/1.0, HTTP/1.1
Connection: Upgrade
```

## Request Header

1. Accept
   1. 처리할 수 있는 미디어 타입과 미디어 타입의 상대적인 우선 순위를 전달한다.
2. Accept-Charset
   1. 처리할 수 있는 문자셋을 전달한다.
   2. 한번에 여러개 지정할 수 있다.
3. Accept-Encoding
   1. 처리할 수 있는 컨텐츠 인코딩 타입을 전달한다.
   2. gzip, compress, deflate, identity
4. Accept-Language
   1. 처리할 수 있는 자연어 세트와 우선 순위를 전달한다.
5. Authorization
   1. 유저의 인증 정보를 전달한다.
6. Expect
   1. 클라이언트가 서버에 특정 동작 요구를 전달한다.
7. From
   1. 사용하고 있는 유저의 메일 주소를 전달한다.
8. Host (HTTP/1.1 필수 헤더)
   1. Request한 리소스의 인터넷 호스트와 포트 번호를 전달한다.
9. If-Match
   1. 조건부 리퀘스트
   2. Etag의 값과 일치하면 리퀘스트를 받을 수 있다.
   3. 일치하지 않으면 412 Precondition Failed를 반환한다.
10. If-Modified-Since
    1. 리소스가 갱신된 날짜에 따라 리퀘스트를 받는다.
    2. 실패하면 304 Not Modified를 반환한다.
11. If-None-Match
    1. If-Match와 반대로 동작
12. If-Range
    1. If-Match처럼 Etag와 비교하고 일치하면 Range Request를 한다.
    2. 일치하지 않으면 전체 Resource를 반환한다.
13. If-Unmodified-Since
    1. If-Modified-Since와 반대로 동작
14. Max-Forwards
    1. 최대로 전송할 수 있는 서버의 수를 나타낸다.
15. Proxy-Authorization
    1. 프록시 서버에서의 인증 요구를 받아들인 때에 인증이 필요한 클라이언트의 정보를 전달한다.
16. Range
    1. 리소스의 일부분만 요청할 수 있다.
    2. 206 Partial Content를 반환한다.
17. Referer (영어 철자는 Referrer가 맞다.)
    1. 리퀘스트가 발생한 본래 리소스의 URI를 전달한다.
18. TE
    1. 리스폰스로 받을 수 있는 전송 코딩의 형식과 상대적인 우선순위를 전달한다.
    2. Accept-Encdoing과 비슷하지만, 전송 코딩에 적용된다.
19. User-Agent
    1. 리퀘스트를 생성한 브라우저와 유저의 이름 등을 전달한다.

## Response Header

1. Accept-Ranges
   1. Range Request를 받을 수 있는지 여부를 전달
2. Age
   1. 얼마나 오래전에 오리진 서버에서 리스폰스가 생성되었는지 전달
3. ETag
   1. 리소스를 특정하기 위한 문자열을 전달
   2. 약한 ETag는 앞에 W/가 붙는다.
4. Location
   1. Request URI 외에 다른 리소스 엑세스를 유도하는 경우에 사용된다.
   2. 보통 Redirection의 URI 처리
5. Proxy-Authenticate
   1. 프록시 서버에서의 인증 요구를 클라이언트에 전달한다.
6. Retry-After
   1. 일정 시간 후에 리퀘스트를 다시 해야하는 것을 전달
7. Server
   1. 서버에 설치되어 잇는 HTTP 서버의 소프트웨어를 전달한다.
8. Vary
   1. 캐시 컨트롤을 위해 사용
   2. 오리진 서버가 프록시 서버에 로컬 캐시를 사용하는 방법에 대한 지시를 전달한다.
9. WWW-Authenticate
   1. HTTP 액세스 인증에 사용된다.

## Entity Header

1. Allow
   1. 리소스가 제공하는 메소드들
2. Content-Encoding
   1. 서버가 엔티티 바디에 대해서 인코딩한 컨텐츠 인코딩 방식을 전달
3. Content-Language
   1. 엔티티 바디에 적용된 언어를 전달
4. Content-Length
   1. 리소스의 크기를 전달
5. Content-Location
   1. 메시지 바디의 URI를 전달
6. Content-MD5
   1. 메시지 바디가 변경되지 않고 도착 했는지 확인하기 위해 MD5 알고리즘에 의해 생성된 값을 전달한다.
   2. 악의적인 변조는 감지할 수 없다. Content-MD5도 변조해서 보내면 되기 때문
7. Content-Range
   1. Range Request에 대해 응답할 때 사용된다.
8. Content-Type
   1. 엔티티 바디에 포함되는 오브젝트의 미디어 타입을 전달
9. Expires
   1. 리소스의 유효 기한 날짜를 전달
10. Last-Modified
    1. 리소스가 마지막으로 갱신 되었던 날짜를 전달

## 쿠키를 위한 헤더 필드

가장 널리 사용되고 있는 쿠키 사양은 넷스케이프사에 의한 사양을 근간으로 확장한 것이다.

- Set-Cookie: 상태 관리 개시를 위한 쿠키 정보 (Response)
- Cookie: 서버에서 수신한 쿠키 정보 (Request)

### Set-Cookie

필드 속성 | 설명
--- | ---
NAME=VALUE | 쿠키에 부여된 이름과 값(필수)
Expires=DATE | 쿠키 유효 기한
Path=PATH | 쿠키 적용 대상이 되는 서버 상의 디렉토리
Domain=도메인 명 | 쿠키 적용 대상이 되는 도메인 명
Secure | HTTPS로 통신하는 경우에만 쿠키를 송신
HttpOnly | 쿠키를 자바스크립트에서 액세스 하지 못하도록 제한, 크로스 사이트 스크립팅으로부터 쿠키의 도청을 막는 것을 목적으로 함

### Cookie

클라이언트가 HTTP의 상태 관리 지원을 원할 때 서버로부터 수신한 쿠키를 이후의 리퀘스트에 포함해서 전달한다.

참고자료: 그림으로 배우는 HTTP & Network Basic, [RFC2616](https://tools.ietf.org/html/rfc2616)
