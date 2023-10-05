---
layout: single
title: "jwt 로 인증하기"
categories:
  - "로그인/인증"
tags:
  - "jwt"
---

## jwt 로 인증하기

현재 쿠키안에 jwt 로 인증을 관리하고 있다.

테스트를 하는 과정에서 jwt 로 인증을 관리하는 방식에 문제가 좀 있다는 것을 알게 되었다.

무슨 문제냐면 jwt 는 최초 로그인할 때 서버쪽에서 유효기간을 써서 클라이언트에 발급을 해준다.

일단 발급된 토큰은 유효기간이 만료될 때까지 자유롭게 쓸 수 있다.

심지어 토큰이 탈취당하더라도 말이다.

따라서 유효기간을 길게 발급한 토큰을 중간에 탈취하면 탈취범은 사용자의 정보를 유효기간동안 자유롭게 볼 수 있게 된다.

그렇다고 토큰의 유효기간을 짧게하자니 사용자는 매번 토큰을 새로 발급받아야 하는 귀찮음이 생긴다.

로그인 방식을 다시 고민해야하는 시점이 되었다.

그래서 여러 블로그를 살펴보면서 고민하고 있다.

아래는 현재 살펴보고 있는 로그인 방법 관련 블로그들이다.

- [쉽게 알아보는 서버 인증 1편(세션/쿠키 , JWT)](https://tansfil.tistory.com/58?category=475681)
- [쉽게 알아보는 서버 인증 2편(Access Token + Refresh Token)](https://tansfil.tistory.com/59?category=475681)
- [인증 방식 : Cookie & Session vs JWT](https://tecoble.techcourse.co.kr/post/2021-05-22-cookie-session-jwt/)
- [[JWT/JSON Web Token] 로그인 / 인증에서 Token 사용하기](https://sanghaklee.tistory.com/47)

블로그 내용을 살펴보고 내린 결론은 jwt 로 인증을 구현하려면 Access Token + Refresh Token 을 발급해서 관리하는 방식으로 가야 한다는 생각을 하게 되었다.
