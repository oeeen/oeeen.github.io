---
layout: single
title:  "Entity와 VO"
date:   2019-08-13 23:00:59 +0900
classes: wide
categories: etc
tags: entity vo
toc: true
toc_sticky: true
---

이 글은 항상 논쟁거리가 되는 Entity, VO가 각각 무엇을 의미하는지 토론이 벌어질 때 마다 추가 되는 글입니다.

틀리다고 생각하시는 부분이 있으면 지적 부탁드립니다.

## Entity

Entity는 어떤 속성으로 같음을 판단하는 객체?

예를 들어 User라는 Entity가 id(PK), name, email, password를 갖는다고 치면 데이터베이스에서는 id로 user를 구분한다.

```java
@Entity
@NoArgsConstructor
@Getter
@EqualsAndHashCode(of = "id")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Convert(converter = EmailConverter.class)
    private Email email;

    @Convert(converter = PasswordConverter.class)
    private Password password;

    @Convert(converter = NameConverter.class)
    private Name name;
}
```

이 User 객체에서 Id를 제외하고 다른 필드들이 변경 되더라도 Id가 동일하다면 동일한 엔티티다. 엔티티는 속성이 바뀔 수 있기 때문에 가변 객체다.

## Value Object

값 자체로 같음을 판단한다. 불변 객체다.

위 코드에서 이어 간다면,

```java
@EqualsAndHashCode(of = "email")
public class Email {
    private static final String EMAIL_REGEX = "^[_a-zA-Z0-9-.]+@[.a-zA-Z0-9-]+\\.[a-zA-Z]+$";

    private final String email;

    private Email(final String email) {
        this.email = validateEmail(email);
    }

    public static Email of(final String email) {
        return new Email(email);
    }
}
```

이런 VO가 있을 수 있겠다. 여기서는 email이 같으면 같은 객체이다.

또 다른 예시를 들어보면 만약 레이싱 게임 코드를 작성하는데, MovedCar라는 VO가 있고 그 VO는 이름과 현재 위치를 갖는다. 그 이름과 현재 위치가 같다면 두 객체는 같은 객체이다.

```java
public class MovedCar {
    private final String name;
    private final int currentPosition;

    public MovedCar(final String name, final int currentPosition) {
        this.name = name;
        this.currentPosition = currentPosition;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        MovedCar movedCar = (MovedCar) o;
        return currentPosition == movedCar.currentPosition &&
                Objects.equals(name, movedCar.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, currentPosition);
    }
}
```

`MovedCar car1 = new MovedCar("MyCar", 10);`

`MovedCar car2 = new MovedCar("MyCar", 10);` 이 경우 car1 과 car2는 동일한 객체다.

테스트 코드로 다음을 확인해볼 수 있다.

```java
public class MovedCarTest {
    @Test
    @DisplayName("동등성 확인")
    void equal() {
        MovedCar car1 = new MovedCar("MyCar", 10);
        MovedCar car2 = new MovedCar("MyCar", 10);

        assertEquals(car1, car2);
//        assertSame(car1, car2);
    }
}
```

실제 객체 레퍼런스 자체는 다르지만(동일성), 동등한 객체(동등성)임을 알 수 있다.(위 코드에서 assertSame 부분의 주석을 풀면 테스트 실패가 된다.)

`assertEqauls(expected, actual)` 부분을 본다면, 아래와 같다.

```java
// Assertions.java
public static void assertEquals(Object expected, Object actual) {
    AssertEquals.assertEquals(expected, actual);
}

// AssertEquals.java
static void assertEquals(Object expected, Object actual) {
    assertEquals(expected, actual, (String) null);
}

// AssertionUtils.java
static boolean objectsAreEqual(Object obj1, Object obj2) {
    if (obj1 == null) {
        return (obj2 == null);
    }
    return obj1.equals(obj2);
}
```

위와 같이 equals로 비교하고 있다는 것을 알 수 있다. 반면 `assertSame(expected, actual)`을 따라가보면 아래와 같다.

```java
// Assertions.java
public static void assertSame(Object expected, Object actual) {
    AssertSame.assertSame(expected, actual);
}

// AssertSame.java
static void assertSame(Object expected, Object actual) {
    assertSame(expected, actual, (String) null);
}
static void assertSame(Object expected, Object actual, String message) {
    if (expected != actual) {
        failNotSame(expected, actual, message);
    }
}
```

위 처럼 `!=`로 비교함을 알 수 있다.

## 왜 VO와 Entity의 구분이 중요할까

같은 속성을 가진 두 개의 Entity가 있다고 해보자. 두 객체가 다른 id를 가지고 있다면 둘은 다른 객체이다.
그러나 같은 속성을 가진 두 개의 VO는 같은 객체이다.

그리고 Entity는 존재하는 동안 다른 속성들이 계속해서 바뀔 수 있다. 하지만 계속 같은 엔티티다.

그러나 VO는 값이 바뀐다면 다른 객체가 된다. 그러니까 돈이라는 객체가 있다고 한다면, 그 돈을 사용해서 얼마를 지불하고 **거스름돈**을 받을 것이다. 그러면 그 거스름돈은 처음에 내가 지불했던 돈과는 다른 값 객체가 되어서 돌아오는 것이다.

참고 자료: [What is the difference between Entities and Value Objects?](https://culttt.com/2014/04/30/difference-entities-value-objects/)
