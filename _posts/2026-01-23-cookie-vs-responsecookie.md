---
title: "[스프링] Cookie 와 ResponseCookie 의 차이점이 뭔가요?"
date: 2026-01-23 04:10:04 +0900
categories: [스프링]
tags: []
image:
  path: /assets/img/posts/cookie-vs-responsecookie/cover.png
  alt: Cookie 와 ResponseCookie 의 차이점이 뭔가요?
---
## 들어가며

프로젝트를 진행하며 쿠키를 클라이언트에 전달하는 로직을 구현했다.
하지만 테스트 과정에서 쿠키가 정상적으로 저장되지 않는 문제를 겪게 되었고, 이를 계기로 쿠키 동작 방식에 대해 다시 살펴보게 되었다.

문제의 원인은 단순했다.
HTTP와 HTTPS 간에는 쿠키가 전달되지 않는다는 브라우저의 보안 정책 때문이었다.
다만 이 과정을 통해 그동안 막연하게 사용해왔던 쿠키에 대해 제대로 이해할 필요성을 느꼈다.

이 글에서는 해당 경험을 바탕으로 Cookie 와 Spring 에서 제공하는 ResponseCookie 에 대해서 정리해보려고 한다.

## Cookie 와 ResponseCookie

### Cookie 가 뭐에요?

Cookie 는 웹 서버가 웹 클라이언트에게 보내는 데이터 종류 중 하나로, 브라우저는 받은 쿠키를 내부에 저장하고 웹 서버에 요청을 보낼 때마다 쿠키를 요청에 담아서 전달한다.

### Cookie<span style="color: grey">(servlet.http.Cookie)</span> 의 역할

- 요청(Request)에 포함된 쿠키를 읽을 때
- 전통적인 방식으로 쿠키를 응답에 추가할 때

### Cookie 설정 방법

Setter 방식으로 설정을 할 수 있다.

``` java
Cookie cookie = new Cookie("refreshToken", refreshToken);
cookie.setHttpOnly(true);
cookie.setSecure(true);
cookie.setPath("/jwt/refresh");
cookie.setMaxAge(7 * 24 * 60 * 60);  // 7일

response.addCookie(cookie);  // 응답에 쿠키 추가
```

### ResponseCookie 의 역할

- 쿠키를 응답에 추가할 때
- 보안 설정이 중요한 경우

Cookie 의 경우 보안 설정에 한계가 있다. 하지만 `ResponseCookie` 의 경우 더 풍부한 보안 설정 메서드를 지원하기 때문에 응답으로 사용하기 적합하다.

### ResponseCookie 설정 방법

ResponseCookie 의 경우 Builder 패턴으로 설정이 가능하다.

``` java
ResponseCookie cookie = ResponseCookie.from("refreshToken", refreshToken)
                .path("/")
                .sameSite("Lax")
                .httpOnly(true)
                .secure(true)
                .maxAge(7 * 24 * 60 * 60)
                .build();
                
response.addHeader(HttpHeaders.SET_COOKIE, cookie.toString());
```

### Cookie 와 ResponseCookie 차이점

| <center>Cookie</center> | ResponseCookie |
| :- | - |
| Servlet 표준 | Spring 전용 |
| Setter 패턴 | Builder 패턴 |
| 제한적인 보안 설정 | 보안 설정 완전 지원 |

<br />

### Cookie 와 Response Cookie 는 각각 언제 사용하지?

Cookie 는 요청 쿠키를 조회할 때 필수적으로 사용된다.
ResponseCookie 는 더 많은 보안 옵션 설정을 할 수 있어서 응답에 추가할 때 사용된다.

Cookie 역시 응답이 가능하지만, ResponseCookie 사용이 권장된다.


### 옵션 정리

#### - sameSite()
>`SameSite()` 는 **_"이 쿠키를 다른 사이트 요청에도 함께 보낼 것인가?"_** 를 설정한다.

위에서 언급했지만, 두 방식은 보안 설정 범위의 차이점이 존재한다.
가장 대표적으로 `sameSite()` 의 유무이다.

sameSite() 에는 몇 가지 옵션이 존재한다.
 
 - Strict <span style="color:grey">(가장 강함)</span>
   완전히 동일한 사이트 요청에서만 쿠키 전송
 - Lax <span style="color:grey">(기본값)</span>
   다른 사이트에서 발생하는 fetch, axios 요청에는 쿠키가 담기지 않음.
   CSRF 공격 차단 가능
 - None <span style="color:grey">(완전 개방)</span>
   이 경우에는 secure 옵션을 무조건 true 로 해야한다. 하지만 그럼에도 CSRF 공격에 취약하다.

<br />

먼저 공격 시나리오에 대해서 생각해보자

1. 사용자가 bank.com 에 로그인<span style="color:grey">(쿠키 저장됨)</span>
2. 사용자가 악성 사이트 evil.com 에 접속
3. evil.com 이 bank.com/transfer 로 요청 전송
4. 쿠키가 자동 포함됨  -> CSRF 공격 발생

SameSite 는 3~4번 단계를 차단해준다.

<br />

**여기서 같은 사이트란 무엇일까?**

> 프론트: https://myVelog.com
백엔드: https://api.myVelog.com

스프링에서는 이런 경우에도 클라이언트 도메인을 cors 정책에서 허용해주어야 한다.
그러면 이 두 도메인은 `'다른 사이트'` 인가?

정답은 **_"다른 사이트가 아니다."_**

✅ `https://myVelog.com` 과 `https://api.myVelog.com` 은 서로 다른 origin 이지만 같은 Site 이다.

#### - domain()

> 쿠키가 적용될 도메인 범위를 지정한다.

``` java
.domain(".example.com")
```

.example.com 입력 시에는
- example.com
- api.example.com
- www.example.com

모두에서 쿠키 공유

⚠️ 생략 시에는 쿠키를 발급한 도메인에서만 사용 가능하다.


#### - maxAge()

> 쿠키의 유효 시간을 설정한다.

``` java
.maxAge(Duration.ofDays(7))
// or
.maxAge(7 * 24 * 60 * 60)
```

초 단위로 설정이 가능하다.

#### - httpOnly()
> JS 의 쿠키 접근을 차단한다.

``` java
.httpOnly(true)
```

XSS 공격을 방어할 수 있다.


#### - secure()
> HTTPS 에서만 쿠키를 전송한다.

``` java
.secure(true)
```

⚠️ sameSite("None") 사용 시 필수이다.


#### - path()
> 쿠키 전송 경로를 제한한다.

``` java
.path("/admin")
```

이 경우에 `/admin` 과 그 하위 경로 외에는 쿠키가 전송되지 않는다.