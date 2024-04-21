---
layout: single
title:  모든 개발자를 위한 HTTP 웹 기본 지식 - 004
categories: 
  - learning http basic by younghan kim
tags: 
  - http
---

## HTTP 메서드

### HTTP API를 만들어보자

요구사항

- 회원 목록 조회
- 회원 조회
- 회원 등록
- 회원 수정
- 회원 삭제

API URI 설계?

- 회원 목록 조회 /read-member-list
- 회원 조회 /read-member-by-id
- 회원 등록 /create-member
- 회원 수정 /update-member
- 회원 삭제 /delete-member

URI 설계의 기준은 `리소스`

- 리소스의 의미는 무엇일까?
    - 회원을 등록/수정/조회하는 것은 리소스가 아님
    - 회원이라는 개념이 바로 리소스
- 리소스는 어떻게 식별하는게 좋을까?
    - 회원을 등록/수정/조회하는 것은 배제
    - 회원리소스를 UIR에 매핑

리소스가 식별된 API URI 설계

- 회원 목록 조회 /members
- 회원 조회 /members/{id}
- 회원 등록 /members/{id}
- 회원 수정 /members/{id}
- 회원 삭제 /members/{id}

계층 구조상 상위를 컬렉션으로 보고 복수단어 사용을 권장 (member -> members)

URI는 리소스만 식별

리소스와 해당 리소스를 대상으로 하는 행위를 분리

- 리소스 : 회원 -> 명사
- 행위 : 조회, 등록, 삭제, 변경 -> 동사

### HTTP 메서드 - GET, POST

HTTP 주요 메서드

- GET : 리소스 조회
- POST : 요청 데이터 처리, 주로 등록
- PUT : 리소스를 대체, 없으면 생성
- PATCH : 리소스 부분 변경
- DELETE : 리소스 삭제

기타 메서드

- HEAD : GET과 동일하지만 메시지 부분을 제외하고, 상태줄과 헤더만 반환
- OPTIONS : 대상 리소스에 대한 통신 가능 옵션(메서드)을 설명(주로 CORS에서 사용)
- CONNECT : 대상 자원으로 식별되는 서버에 대한 터널을 설정
- TRACE : 대상 리소스에 대한 경로를 따라 메시지 루프백 테스트를 수행

#### GET

리소스 조회

서버에 전달하고 싶은 데이터는 query를 통해서 전달

메시지 바디를 사용해서 데이터를 전달할 수 있지만, 지원하는 곳이 많아서 권장하지 않음

#### POST

요청 데이터 처리

메시지 바디를 통해 서버로 요청 데이터 전달

서버는 요청 데이터를 처리

메시지 바디를 통해 들어온 데이터를 처리하는 모든 기능을 수행

주로 전달되 데이터로 신규 리소스 등록, 프로세스 처리에 사용

`요청 데이터를 어떻게 처리한다는 뜻일까?`

SPEC : POST 메서드는 대상 리소스가 리소스의 고유한 의미 체계에 따라 요청에 포함된 표현을 처리하도록 요청

리소스 URI 에 POST 요청이 오면 요청 데이터를 어떻게 처리할지 리소스마다 정해야 함

1. 새 리소스 생성
    - 서버가 아직 식별하지 않은 새 리소스 생성
1. 요청 데이터 처리
    - 단순히 데이터를 생성하거나, 변경하는 것을 넘어서는 큰 프로세스를 처리해야 하는 경우
    - POST 의 결과로 새로운 리소스가 생성되지 않을 수도 있음   
    POST /orders/{orderId}/start-delivery (컨ㅌ롤 URI)
1. 다른 메서드로 처리하기 `애매한` 경우

### HTTP 메서드 - PUT, PATCH, DELETE

#### PUT

리소스를 `완전히` 대체
    - 리소스가 있으면 대체
    - 리소스가 없으면 생성
    - 쉽게 이야기하면 덮어버림
    - 리소스의 일부 필드만을 전달하면 기존 필드가 삭제됨

클라이언트가 리소스를 식별하여 URI 로 지정할 수 있음

```
PUT /members/100 HTTP/1.1
Content-Type: application/json

...
```

PUT은 리소스를 변경하는게 아니라 갈아치우는 것

#### PATCH

리소스 부분 변경

리소스의 일부 필드만을 전달하면 해당 필드만 변경됨

#### DELETE

리소스 제거

### HTTP 메서드의 속성

안전 (Safe Methods), 멱등 (Idenmpotent Methods), 캐시가능 (Cacheable Methods)

#### 안전

여러번 호출해도 리소스를 변경하지 않는다

GET, HEAD

#### 멱등

한 번 호출하든 여러번 호춣하든 결과가 똑같다

GET, PUT, DELETE

POST 는 멱등이 아니다. 중복 결제 사례 등

자동 복구 메커니즘

서버가 TIMEOUT 등으로 정상 응답을 못주었을 때, 클라이언트가 같은 요청을 해도 되는가의 판단 근거

#### 캐시가능

응답 결과 리소스를 캐시해서 사용해도 되는가?

GET, HEAD, POST, PATCH

POST, PATCH 는 서버상 구현이 쉽지 않음

실무적으로는 GET 정도만 사용됨

## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
