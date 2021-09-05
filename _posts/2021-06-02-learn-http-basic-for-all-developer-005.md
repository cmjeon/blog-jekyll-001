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

#### HTTP API - 컬렉션

POST 기반 등록

    - 회원 목록 /members -> GET
    - 회원 등록 /members -> POST
    - 회원 조회 /members/{id} -> GET
    - 회원 수정 /members/{id} -> PATCH, PUT, POST
        - 기존 데이터를 완전히 대치할 수 없으면 PATCH
        - 기존 데이터를 완전히 대치할 수 있으면 PUT (ex 게시판 글 수정)
        - 기존 데이터 대치여부가 애매하면 POST
    - 회원 삭제 /members/{id} -> DELETE

- 클라이언트는 등록될 리소스의 URI를 모름
- 서버가 새로 등록될 리소스 URI 를 생성해줌
- 컬렉션(Collection)
    - 서버가 관리하는 리소스 디렉토리
    - 서버가 리소스의 URI 를 생성하고 관리

#### HTTP API - 스토어

PUT 기반 등록

    - 파일 목록 /files -> GET
    - 파일 조회 /files/{filename} -> GET
    - 파일 등록 /files/{filename} -> PUT
    - 파일 삭제 /files/{filename} -> DELETE
    - 파일 대량 등록 /files -> POST

- 클라이언트가 리소스 URI를 알고 있어야 함
- 클라이언트가 직접 리소스의 URI를 지정
- 스토어(Store)
    - 클라이언트가 관리하는 리소스 저장소
    - 클라이언트가 리소스의 URI 를 알고 관리

#### HTML FORM 사용

HTML FORM 은 GET, POST 만 지원

ajax 같은 기술을 사용하여 해결 가능 -> 회원 API 참고

    - 회원 목록 /members -> GET
    - 회원 등록폼 /members/new -> GET
    - 회원 등록 /members/new, /members -> POST
    - 회원 조회 /members/{id} -> GET
    - 회원 수정폼 /members/{id}/edit -> GET
    - 회원 수정 /members/{id}/edit, /members/{id} -> POST
    - 회원 삭제 /members/{id}/delete -> POST

GET, POST 만 지원되는 경우는 컨트롤 URI 를 이용하여 처리

POST 메소드로 /new, /edit, /delete 등 컨트롤 URI 를 처리

#### 주요 URI 설계 개념

- 문서(document)
    - 단일 개념(파일 하나, 객체 인스턴스, 데이터베이스 row)   
    /members/100, /files/star.jpg
- 컬렉션(collection)
    - 서버가 관리하는 리소스 디렉터리
    - 서버가 리소스의 URI를 생성하고 관리   
      예) /members
- 스토어(store)
    - 클라이언트가 관리하는 자원 저장소
    - 클라이언트가 리소스의 URI를 알고 관리   
      예) /files
- 컨트롤러(controller), 컨트롤 URI
    - 문서, 컬렉션, 스토어로 해결하기 어려운 추가 프로세스 실행
    - 동사를 직접 사용   
      예) /members/{id}/delete

## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
- [https://restfulapi.net/resource-naming](https://restfulapi.net/resource-naming)