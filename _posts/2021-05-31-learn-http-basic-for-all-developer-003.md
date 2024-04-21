---
layout: single
title:  모든 개발자를 위한 HTTP 웹 기본 지식 - 003
categories: 
  - learning http basic by younghan kim
tags: 
  - http
---

## HTTP 기본

### 모든 것이 HTTP

HyperText Transfer Protocol

HTTP 메시지에 모든 것을 전송할 수 있음

- HTML, TEXT
- image, 음성, 영상, 파일
- json, xml(API)
- 거의 모든 형태의 데이터 전송 가능
- 서버간의 데이터를 주고 받을 때도 대부분 HTTP 사용
- 지금은 HTTP 시대!
- HTTP/1.1 이 가장 많이 사용됨

기반 프로토콜

- TCP: HTTP/1.1, HTTP/2
- UDP: HTPP/3
- 현재 HTTP/1.1 을 사용하지만 HTTP/2, HTTP/3 도 점점 증가

웹 브라우저의 개발자모드 > 네트워크 탭에서 protocol 필드를 통해 사용하는 프로토콜 확인가능

#### HTTP 특징

- 클라이언트 서버 구조
- 무상태 프로토콜(statusless), 비연결성
- HTTP 메시지
- 단순, 확장성

### 클라이언트 서버 구조

Request Response 구조

클라이언트는 서버에 요청을 보내고, 응답을 대기

서버가 요청에 대한 결과를 만들어서 응답

클라이언트와 서버를 분리하는 개념이 중요함

클라이언트는 UI와 사용성에 집중

서버는 데이터와 비즈니스로직에 집중

이러한 구조를 통해 클라이언트와 서버가 독립적으로 진화될 수 있었음

### Statful, Stateless

Stateless 란 서버가 클라이언트의 상태를 보존하지 않음을 뜻함

- 장점 : 서버 확장성 높음(스케일 아웃)
- 단점 : 클라이언트가 추가 데이터 전송

#### Stateful 예시

```
고객 : 이 노트북 얼마인가요?
점원 : 100만원입니다. (노트북 상태 유지)

고객 : 2개 구매하겠습니다.
점원 : 200만원입니다( 노트북, 2개 상태 유지), 신용카드, 현금 중 어떤 걸로 구매하시겠어요?

고객 : 신용카드로 하겠습니다.
점원 : 200만원 결제 완료되었습니다. (노트북, 2개, 신용카드 상태 유지)
```

#### Stateless 예시

```
고객 : 이 노트북 얼마인가요?
점원A : 100만원입니다.

고객 : 노트북 2개 구매하겠습니다.
점원B : 노트북 2개는 200만원입니다, 신용카드, 현금 중 어떤 걸로 구매하시겠어요?

고객 : 노트북 2개를 신용카드로 하겠습니다.
점원C : 200만원 결제 완료되었습니다.
```

Stateless 에서는 점원(서버)이 중간이 바뀌더라도 상관없음

#### Stateful, Stateless 의 차이

- Stateful : 중간에 다른 점원으로 바뀌면 안됨 or 고객의 상태정보를 다른 점원들이 미리 알고 있어야 함
- Stateless : 중간에 다른 점원으로 바뀌어도 됨
    - 갑자기 고객이 증가해도 점원을 대거 투입할 수 있음
    - 갑자기 클라이언트 요청이 증가해도 서버를 대거 투입할 수 있음
- 무상태는 응답 서버를 쉽게 증설(Scale out)이 가능

#### 실무적 한계

- 무상태
    - 단순 서비스 소개 화면
    - 데이터를 너무 많이 전송함
- 상태유지
    - 로그인 상태를 서버에 유지
    - 브라우저 쿠키와 서버 세션을 사용하여 상태 유지
    - 상태유지는 최소한으로만 사용

### 비연결성(Connectionless)

연결을 유지하는 모델 : 서버는 연결을 계속 유지해야 하고, 서버 자원이 소모됨

연결을 유지하지 않는 모델 : 서버는 연결 유지X, 최소한의 자원 사용

#### HTTP 비연결성 장점

HTTP 는 기본적으로 연결을 유지하지 않는 모델

일반적으로 초 단위 이하의 빠른 속도로 응답

서버 자원을 매우 효율적으로 사용할 수 있음

#### HTTP 비연결성 단점

TCP/IP 연결을 새로 맺어야 함 : 3 way handshake 시간 추가

웹 브라우저로 사이트를 요청하면 HTML, javascript, css, image 등 수많은 자원이 함께 다운로드

지금은 HTTP 지속 연결(Persistent Connection)로 문제 해결

HTTP/2, HTTP/3 에서 더 많은 최적화

#### Stateless 를 기억하자

- 같은 시간에 딱 맞추어 발생하는 대용량 트래픽에 대한 대응

### HTTP 메시지

HTTP 메시지로 모든 것을 전송

HTTP 는 start line, header, empty line, message body 로 구성

HTTP 공식 스펙

```
start-line
*( header-field CRLF )
CRLF
[ message-body ]
```

HTTP 요청 메시지 예시

```
GET /search?q=hello&hl=ko HTTP/1.1 --> start-line
Host: www.google.com --> header
--> empty-line
```

HTTP 응답 메시지 예시

```
HTTP/1.1 200 OK --> start-line
Content-Type: text/html;charset=UTF-8 --> header
Content-Length: 3423 --> header
--> empty-line
<html> --> message body
  <body>...</body> --> message-body
</html> --> message-body
```

#### 요청메시지 start-line

start-line = request-line / status-line

request-line = method SP(공백) request-target SP HTTP-version CRLF(엔터)

`method`

- 종류 : GET, POST, PUT, DELETE
- 서버가 수행해야 할 동작 지정

`request-target`

- absolute-apth[?query]
- 절대경로="/"로 시작하는 경로

#### 응답메시지 start-line

start-line = request-line / status-line

status-line = HTTP-version SP status-code SP reason-phrase CRLF

`status-code`

- 요청 성공, 실패를 나타냄
    - 200 : 성공
    - 400 : 클라이언트 요청 오류
    - 500 : 서버 내부 오류

`reason-phrase`

- 사람이 이해할 수 있는 짧은 상태 코드 설명

#### header-field

header-field = field-name":" OWS field-value OWS
(OWS: 띄어쓰기 허용)

HTTP 헤더 용도

- HTTP 전송에 필요한 모든 부가정보
- 표준 헤더 너무 많음
- 필요시 임의의 헤더 추가 가능

#### message-body

- 실제 전송할 데이터
- HTML 문서, 이미지, 영상, JSON 등 byte 로 표현할 수 있는 모든 데이터 전송 가능

#### 단순함, 확장성

- HTTP는 단순
- 크게 성공하는 기술은 단순하지만 확장가능한 기술

## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
