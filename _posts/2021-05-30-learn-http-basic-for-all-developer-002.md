---
layout: single
title:  모든 개발자를 위한 HTTP 웹 기본 지식 - 002
categories: 
  - learning http basic by younghan kim
tags: 
  - http
---

## URI 와 웹 브라우저 요청 흐름

### URI

URI(Uniform Resource Identifier)

- Uniform : 리소스를 식별하는 통일된 방식
- Resource : 자원, URI 로 식별할 수 있는 모든 것
- Identifier : 다른 항목과 구분하는데 필요한 정보

> URI는 로케이터(locator), 이름(name) 또는 둘 다 추가로 분류될 수 있다
> https://www.ietf.org/rfc/rfc3986.txt - 1.1.3. URI, URL, and URN

URI 가 URL(위치) 과 URN(이름) 이라는 개념을 포함함

#### URL

Uniform Resource Locator

```
foo://example.com:8042/over/there?name=ferret#nose
```

- foo : scheme
- example.com:8042 : authority
- /over/there : path
- name=ferret : query
- nose : fragment

#### URN

Uniform Resource Name

```
urn:example:animal:ferret:nose
```

- urn : scheme
- example:animal:ferret:nose : path

#### URL, URN

URL - Locator : 리소스가 있는 위치를 지정

URN - Name : 리소스에 이름을 부여

위치는 변할 수 있지만 이름은 변하지 않는다

URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화되지 않음

rufrnr URI = URL

#### URL 분석

```
https://www.google.com/search?q=hello&hl=ko
```

URL 구조

> scheme://[userinfo@]host[:port][/path][?query][#fragment]

`scheme`

- 주로 프로토콜 사용
- 프로토콜 : 어떤 방식으로 자원에 접근할 것인가를 정하는 약속 규칙   
    ex) http, https, ftp emd
- 주로 http 는 80, https 는 443 포트 사용, 포트는 생략가능   
    https 는 http 에 보안이 추가된 것 (HTTP Secure)

`userinfo@`

- URL 에 사용자정보를 포함하여 전송
- 웹브라우저에서는 거의 사용하지 않음

`host`

- 호스트명
- 도메인명 또는 IP 주소를 직접 사용가능

`PORT`

- 접속 포트
- 일반적으로 생략, http 는 80. https 는 443

`path`

- 리소스 경로, 계층적 구조

`query`

- key=value 형태
- ?로 시작, &로 추가 가능   
    ex) ?keyA=valueA&keyB=valueB
- query parameter, query string 등으로 불림

`fragment`

- html 내부 북마크 등에 사용
- 서버에 전송하는 정보 아님

### 웹 브라우저 요청 흐름

```
https://www.google.com/search?q=hello&hl=ko
```

1. DNS 에서 www.google.com 의 IP 확인
1. https scheme 으로 port 443 확인
1. 웹 브라우저가 HTTP 요청 메시지 생성

    ```
    GET /search?q=hello&hl=ko HTTP/1.1
    HOST: www.google.com
    ```

1. SOCKET 라이브러리를 통해 전달
    1. TCP/IP 연결(IP, PORT)
    1. 데이터 전달
1. TCP/IP 패킷 생성, HTTP 메시지 포함
1. 구글서버로 패킷 전송
1. 구글서버에서 프로세싱
1. 구글서버가 HTTP 응답 메시지 생성

    ```
    HTTP/1.1 200 OK
    Content-Type: text/html;charset=UTF-8
    Content-Length: 3423

    <html>
      <body>...</body>
    </html>
    ```

1. 클라이언트로 패킷 전송
1. 웹 브라우저가 수신한 패킷을 해석하여 표시

## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
- [https://www.ietf.org/rfc/rfc3986.txt](https://www.ietf.org/rfc/rfc3986.txt)
