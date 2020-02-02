---
layout: single
title:  "쿠키, 세션"
date:   2019-09-28 18:00:00 +0900
classes: wide
categories: etc
tags: cookie session
toc: true
toc_sticky: true
---

## 쿠키

쿠키는 서버에서 클라이언트에게 전송하는 데이터다. 브라우저는 이 데이터를 저장해놓고 다음 요청에 담아서 함께 전송한다. 보통 동일한 사용자에게서 들어온 요청인지 확인 할 때 사용한다.

쿠키는 주로 아래 세 가지 목적을 위해 사용한다.

1. 세션 관리(Session management)
   - 서버에 저장해야 할 로그인, 장바구니, 게임 스코어 등의 정보 관리
2. 개인화(Personalization)
   - 사용자 선호, 테마 등의 세팅
3. 트래킹(Tracking)
   - 사용자 행동을 기록하고 분석하는 용도

### 쿠키 만들기

HTTP Request를 수신할 때, 서버는 Response와 함께 Set-Cookie 헤더를 전송할 수 있다. Set-Cookie로 만들어진 쿠키는 브라우저에 저장되며, 그 이후로 만료될 때 까지 Request의 헤더에 포함되어 전송 된다. 추가적으로, 특정 도메인 혹은 경로 제한을 설정할 수 있다.

```http
Set-Cookie: Expires=<date>
Set-Cookie: Max-Age=<non-zero-digit>
Set-Cookie: Domain=<domain-value>
Set-Cookie: Path=<path-value>
Set-Cookie: Secure
Set-Cookie: HttpOnly
```

`Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly` 이렇게 해도 된다.

그러면 이 다음 요청부터 브라우저는 Cookie 헤더에 쿠키를 담아서 보낸다.

### 쿠키 옵션

- Expires=\<data\>
  - 쿠키의 최대 수명 (`Date: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT` 의 형식으로 나타남)
  - 이 옵션이 없으면, 세션 쿠키로 취급되며 클라이언트가 종료될 때 파기된다.
  - 이 값은 클라이언트에 상대적인 값으로 취급된다.
- Max-Age=\<number\>
  - 쿠키가 만료될 때 까지의 시간(초), 0이나 음수면 즉시 만료된다. (Max-Age vs. Expires는 Max-Age가 이김)
- Domain, Path 는 쿠키의 스코프를 정의한다.
- Domain=\<domain-value\>
  - 쿠키가 적용되어야 하는 호스트 지정, 없으면 현재 문서의 URI만 적용 (서브 도메인 미포함)
  - ex) `Domain=mozilla.org` 하면 `developer.mozilla.org` 같은 서브도메인에도 들어간다.
- Path=\<path-value\>
  - Path는 URL에 있는 경로다. ex) `Path=/articles`를 설정하면, `/articles/10`, `/articles/test` 에 모두 매칭 된다.
- Secure
  - HTTPS 프로토콜을 사용할 때만 전송된다.
- HttpOnly
  - Javascript에서 Document.cookie로 접근하지 못하게 한다.
- SameSite 쿠키는 Cross Site Request Forgery에 대해 보호 방법을 제공한다. (쿠키가 cross-site 요청과 함께 전송되지 않았음을 밝히게 한다.)

보안을 위해 아래와 같은 기본 동작은 해야한다.

1. 입력 필터링
2. 보안에 민감한 동작을 위해 확인하는 절차는 항상 수행되도록 한다.
3. 민감한 동작에 사용되는 쿠키는 짧은 수명

## 세션

일반적으로 세션은 사용자(클라이언트)를 구분하는 용도로 사용된다. 세션은 클라이언트가 로그인 한 시점부터 브라우저를 종료한 시점까지 들어오는 요청을 하나의 상태로 보고 관리하는 것이다. **클라이언트가 접속해 있는 상태를 하나의 세션이라고 할 수 있다.**
세션은 서버 측에서 관리를 한다. 보통 서버에서는 클라이언트를 구분하기 위해서 Session ID를 쿠키에 넣어 구분한다.

## 세션의 동작 순서

1. 클라이언트가 서버에 처음으로 Request를 보냄 (첫 요청이기 때문에 session id가 존재하지 않음)
2. 서버에서는 session id 쿠키 값이 없는 것을 확인하고 새로 발급해서 응답
3. 이후 클라이언트는 전달받은 session id 값을 매 요청마다 헤더 쿠키에 넣어서 요청
4. 서버는 session id를 확인하여 사용자를 식별
5. 클라이언트가 로그인을 요청하면 서버는 session을 로그인한 사용자 정보로 갱신하고 새로운 session id를 발급하여 응답
6. 이후 클라이언트는 로그인 사용자의 session id 쿠키를 요청과 함께 전달하고 서버에서도 로그인된 사용자로 식별 가능
7. 클라이언트 종료 (브라우저 종료) 시 session id 제거, 서버에서도 세션 제거

## 로그인 상태를 검사하는 세션 (feat. naver)

세션은 클라이언트가 접속해 있는 상태를 말하는 것이므로 생성 시점이라고 하기엔 어색하지만, 보편적으로 사용하는 tomcat의 JSESSIONID의 생성 시점을 알아보았다.

네이버에서 실험을 하기 위해 `https://www.naver.com`를 정확하게 쳐주면 200 OK가 응답으로 온다. 그냥 `www.naver.com` 이나 `naver.com`은 다른 응답이 올 것이다.

`http://www.naver.com` 으로 보내면 `302 Moved Temporarily`이 응답으로 온다.

`naver.com` 만 쳐보면 `301 Moved Permanently`응답으로 `http://www.naver.com`으로 보내고, `http://www.naver.com`에서는 `307 Internal Redirect`로 `https://www.naver.com`으로 보낸다.

![response](/assets/img/session/naver_response.png)

크롬의 개발자모드에서 Application tab을 보면 쿠키를 볼 수 있는데, 초기 상태(로그아웃)에서는 이런 쿠키들이 들어있었다.

![before login](/assets/img/session/before_login.png)

어떤 의미인지 정확하게는 알 수 없지만, NNB는 시크릿모드 기준으로 브라우저를 껐다가 켜면 바뀌었다.

그리고 로그인을 해보면

![after login](/assets/img/session/after_login.png)

이처럼 NID_AUT, NID_JKL, NID_SES 와 같은 쿠키들이 생기는데, 이것으로 세션(로그인 상태)을 관리하는 것으로 추정된다.

NID_JKL은 어떤 역할인지 모르겠지만, 로그인과 함께 생성되고 지워도 로그인 상태와는 관련이 없는 것으로 보인다.

NID_AUT, NID_SES 중 하나만 쿠키를 지우더라도 로그아웃 상태가 되고 나머지 하나도 지워진다. 이 쿠키들을 이용해서 사용자의 로그인 세션을 유지하는 것으로 추정 된다.

그리고, NID_AUT는 HttpOnly옵션이 달려있고, NID_SES는 아닌데 이건 어떤 의미로 이렇게 만들었을까? 궁금증이 생겼다. (이 궁금증을 해결할 방법이 있을까)

## 참고자료

- [https://cjh5414.github.io/cookie-and-session/](https://cjh5414.github.io/cookie-and-session/)
- [https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies](https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies)
- [https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Set-Cookie](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Set-Cookie)
- [https://www.naver.com](https://www.naver.com)
