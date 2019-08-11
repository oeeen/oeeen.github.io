---
layout: single
title:  "데메테르의 법칙(Law of Demeter)"
date:   2019-07-03 15:01:59 +0900
categories: etc
classes: wide
tag: java
---

## 데메테르의 법칙

-   메서드에서 생성한 객체의 메서드만 호출
-   파라미터로 받은 객체의 메서드만 호출
-   필드로 참조하는 객체의 메서드만 호출

### 예시

#### 신문 배달부가 고객에게 요금을 받아가는 상황

고객.getWallet.getMoney 이런식으로 구현 가능 --> 데메테르의 법칙 위반 --> 데이터 중심으로 생각하지 말자.

이 상황을 현실반영해서 생각해보면 신문 배달부는..

1.  고객의 지갑을 꺼냄
2.  그 지갑에 돈을 신문 가격보다 많이 가지고 있는지 확인
3.  돈이 충분하면 돈을 가져감.  
    이러고 있는 것이다.

말도 안되는 짓이기 때문에.. 현실 반영해서 코드를 생각해보면..

1.  신문 배달부는 돈 주세요! 한다.
2.  고객은 자신이 가진 돈이 신문 가격에 충분한지 확인한다.
3.  고객은 돈을 지불한다. (자기가 가진 돈에서 빼면서)  
    이렇게 하면 될 것 같다.

### 데메테르 법칙 위반의 전형적인 증상

데메테르의 법칙을 위반하는 전형적인 증상이 두가지가 있다.

-   연속된 get method 호출
-   임시 변수의 get 호출이 많음

`value = someObject.getA().getB().getValue();` 이런식이다

```
A a = someObject.getA();
B b = a.getB();
value = b.getValue();
```

두 코드는 결국 동일하다. 이 두 가지 증상을 보인다면 데메테르의 법칙을 어기고 있을 가능성이 높다.  
이는 캡슐화를 약화시켜서 코드 변경을 어렵게 만드는 원인이 될 수 있다.

우아한테크코스 1단계 중, 사다리게임 코드 리뷰에서 이런 식으로 코드를 짰다.

```
public RewardPersonConnector(Ladder ladder, LadderGameData ladderGameData) {
        for (int i = 0; i < ladderGameData.getPerson().getCountOfPerson(); i++) {
            nameToResult.put(ladderGameData.getPerson().getName(i), 
            ladderGameData.getLadderRewards().getResult(ladder.moveStartToEnd(i + 1) - 1));
        }
}
```

이런식으로 디미터 원칙을 위반 했었다..

해결 방법을 생각해보자. Rewards와 Person간의 Map을 연결 시켜주는 클래스 인 것 같다.

그래서 여기에는 Person의 이름과 LadderRewards 내부의 Result가 필요하다.  
전체 구조 자체를 바꿔야 이 문제를 해결 할 수 있을 것 같다. 아니면 LadderGameData 내부에서  
getPerson().getName() + getLadderRewards().getResult() 가 아니라  
LadderGameData의 클래스 명을 바꾸고 해당 클래스 내부에서

```
for (int i = 0; i < person.getCountOfPerson(); i++) {
    nameToResult.put(person.getName(i), ladderGameRewards.getResult(ladder.moveStartToEnd(i + 1) - 1));
}
```

이런 식으로 할 수도 있지 않을까?  
이렇게 바꾼다고 해도 별로 마음에 드는 코드는 아니다. 구조 자체를 다시 생각해보아야 겠다.

참고: 개발자가 반드시 정복해야 할 객체 지향과 디자인 패턴 - 최범균 저