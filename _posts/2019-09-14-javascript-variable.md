---
layout: single
title:  "자바스크립트 변수와 값"
date:   2019-09-14 18:55:59 +0900
classes: wide
categories: etc
tag: javascript
toc: true
toc_sticky: true
---

## 특수한 값

값이 없음을 표현하기 위한 특수한 값에는 null과 undefined가 있다.

undefined는 다음 값들이 있다.

- 값을 아직 할당하지 않은 변수의 값
- 없는 객체의 프로퍼티를 읽으려고 시도했을 때의 값
- 없는 배열의 요소를 읽으려고 시도했을 때의 값
- 아무것도 반환하지 않는 함수가 반환하는 값
- 함수를 호출했을 때 전달받지 못한 인수의 값

null은 아무것도 없음을 값으로 표현한 리터럴

## 심벌

심벌은 Symbol()을 사용해서 생성한다. Symbol()은 호출할 때마다 새로운 값을 만든다.

```javascript
let sym1 = Symbol();
let sym2 = Symbol();
console.log(sym1 == sym2); // false
```

Symbol()에 인수를 전달하면 심버르이 설명을 덧붙일 수 있다.

```javascript
let HEART = Symbol("하트");
console.log(HEART.toString()); // -> Symbol(하트)
```

### 심벌과 문자열 연결하기

Symbol.for()를 활용하여 문자열과 연결된 심벌을 생성할 수 있다.

```javascript
let sym1 = Symbol.for("test");
let sym2 = Symbol.for("test");
console.log(sym1 == sym2); // true
```

심벌과 연결된 문자열은 `Symbol.keyFor()`로 구할 수 있다.

```javascript
let sym1 = Symbol.for("test");
let sym2 = Symbol("test");
console.log(Symbol.keyFor(sym1)); // -> test
console.log(Symbol.keyFor(sym2)); // -> undefined
```

## 템플릿 리터럴

템플릿 리터럴은 역따옴표(\`)로 묶은 문자열이다. 전에 자바스크립트로 게시글 템플릿(HTML 태그들..)을 만들 때 역 따옴표를 사용했었다. 예시를 보면 다음과 같다. 실제 사용했던 코드의 일부만 가져왔다.

```javascript
const article = `<div class="card widget-feed" data-object="article" data-article-id="{{id}}">
                    <div class="feed-header">
                        <ul class="list-unstyled list-info">
                            <li class="float-right mrg-right-40">
                                <div id="range-icon-{{id}}" class="ti-lock font-size-20"></div>
                            </li>
                        </ul>
                    </div>
                </div>`
```

템플릿 리터럴 안에는 플레이스 홀더를 넣을 수 있다. 플레이스 홀더는 ${...}로 표기한다. 자바스크립트 엔진은 플레이스 홀더 내부의 ...을 표현식으로 간주하여 evaluation 한다.

책의 예시를 살펴보면 다음과 같다.

```javascript
let a = 2;
let b = 3;
console.log(`${a} + ${b} = ${a+b}`); // 2 + 3 = 5
let now = new Date();
console.log(`오늘은 ${now.getMonth() + 1} 월 ${now.getDate()} 일 입니다.`); // 오늘은 9월 13일입니다.
```

플레이스 홀더는 사용자의 입력 값처럼 실행 시점에 외부에서 주어지는 값을 표현식에 반영하고자 할 때 그것이 들어갈 수 있도록 마련한 장소이다.

## 참고자료

- 모던 자바스크립트 입문 3장
