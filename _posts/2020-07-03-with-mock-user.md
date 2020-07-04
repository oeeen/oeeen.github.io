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

일단 스프링 시큐리티 공식 문서에서 19. Testing 부분에서 내가 참고했던 부분을 번역해본다.

### @WithMockUser

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

### @WithAnonymousUser

> @WithAnonymousUser를 사용하는 것은 anonymous 사용자로 테스트를 돌리는 것이다. 이 방식은 대부분은 특정 유저로 테스트하고, 일부분의 테스트만 anonymous로 하려고 할 때 특히 유용하다. 예를 들어 아래 나오는 내용은 withMockUser1과 withMockUser2 메서드는 @WithMockUser로 돌리고, anonymous 메서드는 anonymous 사용자로 테스트를 돌린다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WithMockUser
public class WithUserClassLevelAuthenticationTests {

    @Test
    public void withMockUser1() {
    }

    @Test
    public void withMockUser2() {
    }

    @Test
    @WithAnonymousUser
    public void anonymous() throws Exception {
        // override default to run as anonymous user
    }
}
```

SecurityContext에 anonymous로 테스트하기 위해서는 @WithAnonymousUser를 사용하면 된다.

### @WithSecurityContext

우리는 @WithSecurityContext를 이용해서 우리가 원하는 SecurityContext를 만들 수 있는 커스텀 어노테이션을 만들 수 있다. 예를 들어 우리는 @WithMockCustomUser라는 어노테이션을 아래 나온 것처럼 만들 수 있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithMockCustomUserSecurityContextFactory.class)
public @interface WithMockCustomUser {

    String username() default "rob";

    String name() default "Rob Winch";
}
```

@WithMockCustomUser는 @WithSecurityContext 어노테이션과 함께 사용되었음을 볼 수 있다. 이는 Spring Security Test support에게 우리가 테스트에서 SecurityContext를 만들기 위해 신호를 주는 것이다. @WithSecurityContext 어노테이션은 @WithMockCustomUser에게 새로운 SecurityContext를 제공하기 위한 SecurityContextFactory를 특정하는 것이 필요하다. SecurityContextFactory를 아래처럼 구현하는 것을 볼 수 있다.

```java
public class WithMockCustomUserSecurityContextFactory
    implements WithSecurityContextFactory<WithMockCustomUser> {
    @Override
    public SecurityContext createSecurityContext(WithMockCustomUser customUser) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();

        CustomUserDetails principal =
            new CustomUserDetails(customUser.name(), customUser.username());
        Authentication auth =
            new UsernamePasswordAuthenticationToken(principal, "password", principal.getAuthorities());
        context.setAuthentication(auth);
        return context;
    }
}
```

이제 이 내용을 참고해서 내 @WithMockCustomUser 를 만들어보자.

## 커스텀 @WithMockUser

일단 @WithMockUser 가 사용하는 WithMockUserSecurityContextFactory를 살펴보자.

```java
final class WithMockUserSecurityContextFactory implements
        WithSecurityContextFactory<WithMockUser> {

    public SecurityContext createSecurityContext(WithMockUser withUser) {
        String username = StringUtils.hasLength(withUser.username()) ? withUser
                .username() : withUser.value();
        if (username == null) {
            throw new IllegalArgumentException(withUser
                    + " cannot have null username on both username and value properites");
        }

        List<GrantedAuthority> grantedAuthorities = new ArrayList<>();
        for (String authority : withUser.authorities()) {
            grantedAuthorities.add(new SimpleGrantedAuthority(authority));
        }

        if (grantedAuthorities.isEmpty()) {
            for (String role : withUser.roles()) {
                if (role.startsWith("ROLE_")) {
                    throw new IllegalArgumentException("roles cannot start with ROLE_ Got "
                            + role);
                }
                grantedAuthorities.add(new SimpleGrantedAuthority("ROLE_" + role));
            }
        } else if (!(withUser.roles().length == 1 && "USER".equals(withUser.roles()[0]))) {
            throw new IllegalStateException("You cannot define roles attribute "+ Arrays.asList(withUser.roles())+" with authorities attribute "+ Arrays.asList(withUser.authorities()));
        }

        User principal = new User(username, withUser.password(), true, true, true, true,
                grantedAuthorities);
        Authentication authentication = new UsernamePasswordAuthenticationToken(
                principal, principal.getPassword(), principal.getAuthorities());
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authentication);
        return context;
    }
}
```

일단 들어온 username이 비어있는지 확인한다. Role에 `ROLE_` 이라는 prefix가 있는지 확인한다.(있으면 exception) 그 다음 Authority를 하나만 지정했으면서 그 권한을 User로 설정했다면 exception을 던진다.

그 다음으로 본격적인 과정이 있다. annotation 설정 값으로부터 들어온 값을 이용하여 User 객체(import org.springframework.security.core.userdetails.User;
)를 만든다. 그리고 UsernamePasswordAuthenticationToken(Authentication 객체)을 만들어 SecurityContext에 넣어준 후 그 SecurityContext를 return 한다.

그러면 이와 비슷한 과정으로 Custom annotation과 Factory를 만들어보면 아래와 같다. (import 부분도 포함했다.)

```java
// WithMockCustomUser.java
import org.springframework.security.test.context.support.WithSecurityContext;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithMockCustomUserSecurityContextFactory.class)
public @interface WithMockCustomUser {

    String username() default "martin";

    UserRole role() default UserRole.BUYER;
}

// WithMockCustomUserSecurityContextFactory.java
import dev.smjeon.commerce.security.UserContext;
import dev.smjeon.commerce.security.token.PostAuthorizationToken;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.test.context.support.WithSecurityContextFactory;

public class WithMockCustomUserSecurityContextFactory implements WithSecurityContextFactory<WithMockCustomUser> {

    @Override
    public SecurityContext createSecurityContext(WithMockCustomUser customUser) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();

        UserContext userContext = new UserContext(1L, "aabb", customUser.username(), customUser.role());
        PostAuthorizationToken token = new PostAuthorizationToken(userContext);

        context.setAuthentication(token);

        return context;
    }
}
```

WithMockUserSecurityContextFactory에서 본 Exception 처리 부분은 제외하고 SecurityContext를 채우는 부분만 구현해서 만들었다. 프로젝트에서 로그인 했을 때 PostAuthorizationToken을 Authentication의 principal로 사용한다. 그래서 WithMockCustomUser annotation으로부터 가져올 수 있는 유저 이름과 권한을 이용해서 PostAuthorizationToken을 만들어서 SecurityContext에 채우고 그 SecurityContext를 return 하는 방식으로 구현했다.

이제 이 `@WithMockCustomUser`를 이용해서 테스트를 작성해보면 다음과 같다.

```java
@Test
@WithAnonymousUser
@DisplayName("권한 없이 모든 상품을 조회할 수 있습니다.")
void findAll() {
    List<Product> products = Collections.singletonList(product);
    given(productRepository.findAll()).willReturn(products);

    productService.findAll();

    verify(productRepository).findAll();
}

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

`@WithAnonymousUser`는 기본 어노테이션으로 아무 권한이 없더라도 접근할 수 있는 로직에 대한 테스트이다. `@WithMockCustomUser(role = UserRole.ADMIN)`는 ADMIN 권한이 있어야만 접근할 수 있는 로직에 대한 테스트이다. 지금 구현은 ServiceTest지만 User를 Mocking해서 쓰는게 아니라 ADMIN권한으로 실제로 만들어서 테스트를 해서 원래 `@WithMockUser`의 원래 사용 목적과는 다를 수 있다. 하지만 `@WithMockCustomUser` 가 없으면 SecurityContext 내부 객체가 비어있어서 테스트가 실패할 것이다.

ProductService의 delete 메서드를 살펴보면 사실 User의 권한에 따라 product의 상태가 다르게 바뀌게 구현되어있다.(ADMIN이면 BLOCKED, 본인이 삭제하면 REMOVED)

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

지금 테스트 코드에서 `@WithMockCustomUser`에서 UserRole을 변경해도 상관없이 돌아간다.(SecurityContext의 Authentication만 들어있다면..) 이는 실제 `@WithMockUser`의 사용 목적과는 다른 것 같기 때문에, 변경이 필요할 것 같다.

## 참고자료

- [Spring Security Docs - Testing](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#test)
