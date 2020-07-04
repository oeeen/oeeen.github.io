---
layout: single
title:  "Spring Security Test 하기"
date:   2020-07-03 22:30:59 +0900
classes: wide
categories: etc
tags: spring test
toc: true
toc_sticky: true
---

Spring Security를 사용하면 권한을 이용한 테스트를 할 때가 있다. @WithMockUser 어노테이션을 사용하고 싶은데 커스텀 Authentication 객체를 사용할 때, 이를 그대로 사용할 수는 없다. 이걸 어떻게 쓰려고 했는지 경험을 기록하려고 한다.

## Spring Security Docs

일단 스프링 시큐리티 공식 문서에서 19. Testing 부분을 번역해본다.

> 문제는 "특정 사용자로서 테스트를 어떻게 할까?" 이다. 답은 `@WithMockUser`를 사용하는 것이다. 아래 테스트는 username "user"와 password "password"인 "ROLE_USER" 권한으로 테스트를 돌린다.

```java
@Test
@WithMockUser
public void getMessageWithMockUser() {
String message = messageService.getMessage();
...
}
```

다음과 같은 내용이 뒷받침된다.

1. "user"이름을 가진 유저는 없어도 된다.(mocking하기 때문에)
2. UsernamePasswordAuthenticationToken 타입의 Authentication 객체가 SecurityContext 내에 생긴다.
3. Authentication 객체 내의 principal은 Spring Security의 User 객체이다.
4. User는 username은 "user", password는 "password", 권한은 "ROLE_USER"이다.

이 문서를 읽어보면 @WithMockUser는 Spring Security의 User 객체를 사용하고, 기본 Authentication 객체를 이 User 객체로 채운다. 그리고 SecurityContext내에 Authentication Setting한다.

## 실제 사용한 코드

```java
@Test
@WithMockCustomUser(role = UserRole.ADMIN)
@DisplayName("관리자가 삭제요청을 하면 Blocked 상태로 변경됩니다.")
void deleteWithAdmin() {
    given(productRepository.findById(anyLong())).willReturn(Optional.ofNullable(product));
    given(userService.findById(anyLong())).willReturn(admin);

    productService.delete(1L);

    verify(productRepository).findById(1L);
    assertEquals(product.getStatus(), ProductStatus.BLOCKED);
}
```

ProductService 내에 테스트 코드이다. 위에서 만들었던 @WithMockCustomUser 어노테이션에 ADMIN 권한으로 메서드를 실행한다. ProductService의 delete 메서드도 살펴보자.

```java
@Transactional
public void delete(Long productId) {
    Product product = productRepository.findById(productId).orElseThrow(() -> new NotFoundProductException(productId));
    User owner = getUserFromAuthentication();

    if (owner.isAdmin()) {
        product.block();
        return;
    }

    product.remove(owner);
}

private User getUserFromAuthentication() {
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    UserContext user = (UserContext) authentication.getPrincipal();
    return userService.findById(user.getId());
}
```

그리고 @WithMockCustomUser를 SELLER 권한이나 BUYER 권한으로 실행한 결과를 살펴보면 아래와 같다.

## 참고자료

- [Spring Security Docs - Testing](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#test)
