---
layout: single
title:  모든 개발자를 위한 HTTP 웹 기본 지식 - 007
categories: 
  - learning http basic by younghan kim
tags: 
  - http
---

## HTTP 헤더1 - 일반 헤더

### HTTP 헤더 개요

#### HTTP 헤더

- header-filed = field-name:OWSfield-valueOWS(ows:띄어쓰기 허용)
- field-name 은 대소문자 구분 없음

#### HTTP 헤더 분류

- Request 헤더: 요청 정보   
예) User-Agent: Mozilla/5.0 (Macintosh; ..)
- General 헤더: 메시지 전체에 적용되는 정보   
예) Connection: close
- Response 헤더: 응답 정보   
예) Server: Apache
- Entity 헤더: 엔티티 바디 정보   
예) Content-Type: text/html, Content-Length: 3423

#### RFC2616 HTTP BODY

- 메시지 본문은 엔티티 본문을 전달하는데 사용
- 엔티티 본문은 요청/응답에 사용할 데이터
- 엔티티 헤더는 엔티티 본문을 해석할 수 있는 정보 제공

#### RFC723x 변화

- 엔티티(Entity) -> 표현(Representation)
- 표현 = 표현 메타데이터 + 표현 데이터

#### RFC723x HTTP BODY

- 메시지 본문(payload)을 통해 표현 데이터 전달
- 표현은 요청/응답에 사용할 데이터
- 표현 헤더는 표현 데이터를 해석할 수 있는 정보 제공

### 표현

표현헤더는 전송/응답 둘다에서 사용

- Content-Type: 표현 데이터의 형식
- Content-Encoding: 표현 데이터의 압축 방식
- Content-Language: 표현 데이터의 자연 언어
- Content-Length: 표현 데이터의 길이

#### Content-Type

미디어 타입, 문자 인코딩

- text/html; charset=utf-8, application/json, image/png

#### Content-Encoding

표현 데이터를 압축하기 위해 사용

데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가 -> 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제

#### Content-Language

표현 데이터의 자연 언어를 표현

- ko, en, en-US

#### Content-Length

바이트 단위

Transfer-Encoding 을 사용할 때 Content-Length 를 사용하면 안됨

### 콘텐츠 협상(콘텐츠 네고시에이션)

클라이언트가 선호하는 표현 요청, 요청시에 사용

- Accept : 클라이언트가 선호하는 미디어 타입 전달
- Accept-Charset: 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
- Accept-Language: 클라이언트가 선호하는 자연 언어

#### Accept-Language 적용 전

1. 클라이언트 -> 서버 : 요청

    ```
    GET /event
    ```

1. 서버 -> 클라이언트 : 응답

    ```
    Content-Language: en

    hello
    ```

#### Accept-Language 적용 후

1. 클라이언트 -> 서버 : 요청

    ```
    GET /event
    Accept-Language: ko
    ```

1. 서버 -> 클라이언트 : 응답

    ```
    Content-Language: ko

    안녕하세요
    ```

#### 협상과 우선순위1

- quality values(q) 값 사용
- 0~1, 클수록 높은 우선순위
- 생략 시 1

#### Accept-Language 적용 후

기본언어 독일어, 영어 지원되는 서버에서는 영어로 응답함

1. 클라이언트 -> 서버 : 요청

    ```
    GET /event
    Accept-Language: ko-KR,ko;q=0.9,en-US;1=0.8,en;9=0.7
    ```

1. 서버 -> 클라이언트 : 응답

    ```
    Content-Language: en

    Hello
    ```

#### 협상과 우선순위2

클라이언트가 선호하는 미디어 타입은 구체적인 것이 우선

    - Accept: text/*, text/plain, text/plain; format=flowed, */*

1. `text/plain;format=flowed`
1. `text/plain`
1. `text/*`
1. `*/*`

#### 협상과 우선순위3

구체적인 것을 기준으로 미디어 타입을 맞춤

    - Accept: text/*;q=0.3, text/html;q=0.7, text/html;level=1, text/html;level=2;q=0.4, */*;q=0.5

1. `text/html;level=1`
1. `text/html`
1. `text/html;level=3`
1. `image/jpeg`
1. `text/html;level=2`
1. `text/plain`

### 전송 방식

#### 전송방식

    - 단순 전송
    - 압축 전송
    - 분할 전송
    - 범위 전송

#### 단순 전송

서버 -> 클라이언트 : 응답

  ```
  HTTP/1.1 200OK
  Content-Type: text/html;charset=UTF-8
  Content-Length:3424

  <html>
    <body>...</body>
  </html>
  ```

#### 압축 전송

서버 -> 클라이언트 : 응답

  ```
  HTTP/1.1 200OK
  Content-Type: text/html;charset=UTF-8
  Content-Encoding: gzip
  Content-Length:521

  dkqn21k23j1l32132jdkf...
  ```

#### 분할 전송

서버 -> 클라이언트 : 응답

  ```
  HTTP/1.1 200OK
  Content-Type: text/plain
  Transfer-Encoding: chunked

  5
  Hello
  5World
  0
  \r\n
  ```

#### 범위 전송

서버 -> 클라이언트 : 응답

  ```
  HTTP/1.1 200OK
  Content-Type: text/plain
  Content-Range: bytes 1001-2000 / 2000

  eriwrwn123sfjpdfopwer
  ```

### 일반 정보

  ```
  - From: 유저 에이전트의 이메일 정보
  - Referer: 이전 웹 페이지 주소
  - User-Agent: 유저 에이전트 애플리케이션 정보
  - Server: 요청을 처리하는 오리진 서버의 소프트웨어 정보
  - Date: 메시지가 생성된 날짜
  ```

#### From

- 일반적으로 잘 사용되지 않음
- 검색 엔진 같은 곳에서 주로 사용, 요청에서 사용

#### Referer

- 현재 요청된 페이지의 이전 웹 페이지 주소
- A -> B로 이동하는 경우 B를 요청할 때 Referer: A 를 포함해서 요청
- Referer를 사용해서 유입 경로 분석 가능
- 요청에서 사용
- 참고: referer는 단어 referrer의 오타

#### User-Agent

- user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/ 537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36
- 클리이언트의 애플리케이션 정보(웹 브라우저 정보, 등등)
- 통계 정보
- 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
- 요청에서 사용

#### Server

요청을 처리하는 Origin 서버의 소프트웨어 정보

- Server: Apache/2.2.22 (Debian) server: nginx
- 응답에서 사용

#### Date

- Date: Tue, 15 Nov 1994 08:12:31 GMT
- 응답에서 사용

### 특별한 정보

  ```
  - Host : 요청한 호스트 정보(도메인)
  - Location : 페이지 리다이렉션
  - Allow : 허용 가능한 HTTP 메서드
  - Retry-After : 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
  ```

#### Host

- 요청에서 사용
- 필수
- 하나의 서버가 여러 도메인을 처리해야 할 때
- 하나의 IP 주소에 여러 도메인이 적용되어 있을 때

#### Location

- 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동 (리다이렉트)
- 201 (Created): Location 값은 요청에 의해 생성된 리소스 URI
- 3xx (Redirection): Location 값은 요청을 자동으로 리디렉션하기 위한 대상 리소스를 가리킴

#### Allow

- 405 (Method Not Allowed) 에서 응답에 포함해야함
- Allow: GET, HEAD, PUT
- 응답에서 사용

#### Retry-After

- 503 (Service Unavailable): 서비스가 언제까지 불능인지 알려줄 수 있음
- Retry-After: Fri, 31 Dec 1999 23:59:59 GMT (날짜 표기)
- Retry-After: 120 (초단위 표기)

### 인증

  ```
  - Authorization: 클라이언트 인증 정보를 서버에 전달
  - WWW-Authenticate: 리소스 접근시 필요한 인증 방법 정의
  ```

#### Authorization

클라이언트 인증정보를 서버에 전달

#### WWW-Authenticate

리소르 접근 시 필요한 인증 방법 정의

- 리소스 접근시 필요한 인증 방법 정의
- 401 Unauthorized 응답과 함께 사용

### 쿠키

  ```
  - Set-Cookie: 서버에서 클라이언트로 쿠키 전달(응답)
  - Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달
  ```

#### 쿠키 미사용

1. 클라이언트 -> 서버 : 요청

    ```
    GET /welcome HTTP/1.1
    ```

1. 서버 -> 클라이언트 : 응답

    ```
    HTTP/1.1 200 OK

    안녕하세요. 손님
    ```

1. 클라이언트 -> 서버 : 요청

    ```
    POST /login HTTP/1.1
    user=홍길동
    ```

1. 서버 -> 요청 : 응답

    ```
    GET /welcome HTTP/1.1
    ```

1. 서버 -> 클라이언트 : 응답

    ```
    HTTP/1.1 200 OK

    안녕하세요. 손님
    ```

어째서 로그인 여부를 확인하지 못하는가?

#### Stateless 

- HTTP는 무상태(Stateless) 프로토콜이다.
- 클라이언트와 서버가 요청과 응답을 주고 받으면 연결이 끊어진다.
- 클라이언트가 다시 요청하면 서버는 이전 요청을 기억하지 못한다.
- 클라이언트와 서버는 서로 상태를 유지하지 않는다.

#### 쿠키

1. 로그인 후 로그인 정보를 쿠키저장소에 보관
1. 페이지에 접근할 때마다 쿠키 저장소에서 조회
1. 모든 요청에 쿠키 정보를 자동 

- 사용처
  - 사용자 로그인 세션 관리
  - 광고 정보 트래킹
- 쿠키 정보는 항상 서버에 전송됨
  - 네트워크 트래픽 추가 유발
  - 최소한의 정보
  - 보안에 민감한 데이터는 저장금지

#### 쿠키 - 생명주기

- Set-Cookie: expires=Sat, 26-Dec-2020 04:39:21 GMT
  - 만료일이 되면 쿠키 삭제
- Set-Cookie: max-age=3600 (3600초)
  - 0이나 음수를 지정하면 쿠키 삭제
- 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지
- 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지

#### 쿠키 - 도메인

- 명시: 명시한 문서 기준 도메인 + 서브 도메인 포함
  - domain=example.org를 지정해서 쿠키 생성
  - example.org는 물론이고, dev.example.org도 쿠키 접근
- 생략: 현재 문서 기준 도메인만 적용
  - example.org 에서 쿠키를 생성하고 domain 지정을 생략
  - example.org 에서만 쿠키 접근, dev.example.org는 쿠키 미접근

#### 쿠키 - 경로

path 에 선언한 경로를 포함한 하위 경로 페이지에서만 쿠키 접근

- path=/home 지정
- /home -> 가능
- /home/level1 -> 가능
- /hello -> 불가능

#### 쿠키 - 보안



## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
- [https://restfulapi.net/resource-naming](https://restfulapi.net/resource-naming)