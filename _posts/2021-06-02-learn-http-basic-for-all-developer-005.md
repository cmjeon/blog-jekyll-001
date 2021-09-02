---
layout: single
title:  모든 개발자를 위한 HTTP 웹 기본 지식 - 005
categories: 
  - learning http basic by younghan kim
tags: 
  - http
---

## HTTP 메서드 활용

### 클라이언트에서 서버로 데이터 전송

데이터 전달 방식은 크게 2가지

- 쿼리 파라미터를 통한 데이터 전송
    - GET
    - 주로 정렬 필터
- 메시지 바디를 통한 데이터 전송
    - POST, PUT, PATCH
    - 리소스 등록, 변경

4가지 상황

#### 정적 데이터 조회

    - GET
    - 이미지, 정적 텍스트 문서
    - 쿼리 파라미터 미사용

#### 동적 데이터 조회

    - GET
    - 조건, 필터
    - 쿼리 파라미터 사용
    - 쿼리 파라미터에 필터, 조건 등에 사용

#### HTML Form 을 통한 데이터 전송

폼에 데이터를 입력하여 전달하면 HTTP 메시지 생성

POST 전송

```
<form action="/save" method="post">
    ...
</form>
```

HTTP 메시지

```
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```

application/x-www/form-urlencoded 로 한글 전송 시 url encoding 처리 (ex abc김 -> abc%EA%B9$=%80)


GET 전송

```
<form action="/save" method="get">
    ...
</form>
```

HTTP 메시지

```
POST /save?username=kim&age=20 HTTP/1.1
Host: localhost:8080
```

multipart/form-data

```
<form action="/save" method="post" enctype="multipart/form-data">
    ...
</form>
```

HTTP 메시지

```
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: multipart/form-data; boundary=----XXX
Content-Lencth: 10457

----XXX
Content-Disposition: form-data; name="username"

kim
----XXX
Content-Disposition: form-data; name="age"

20
----XXX
Content-Disposition: form-data; name="file1"; filename="intro.png"
Content-Type: image/png

109238a9...
----XXX
```

form 의 기본 Content-Type 은 application/x-www-form-urlencoded

HTML Form 전송은 GET, POST 만 지원

#### HTTP API 를 통한 데이터 전송

HTTP 메시지

```
POST /members HTTP/1.1
Content-Type: application/json

{
    "username":"young",
    "age":20
}
```

- 서버 to 서버
    - 백엔드 시스템 통신
- 앱 클라이언트
    - 아이폰, 안드로이드
- 웹 클라이언트
    - HTML 에서 Form 전송 대신 javascript 를 통한 통신에 사용 (ajax)
- POST, PUT, PATCH: 메시지 바디를 통한 데이터 전송
- GET: 조회, 쿼리 파라미터로 데이터 전달
- Content-Type: application/json 을 주로 사용 (사실상 표준, de facto) 

### HTTP API 설계 예시





## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)