---
layout: single
title:  모든 개발자를 위한 HTTP 웹 기본 지식 - 006
categories: 
  - learning http basic by younghan kim
tags: 
  - http
---

## HTTP 상태코드

### HTTP 상태코드 소개

클라이언트가 보낸 요청의 처리 상태를 응답에서 알려주는 기능

    - 1xx (Informational) : 처리중
    - 2xx (Successful) : 정상처리
    - 3xx (Redirection) : 요청을 완료하려면 추가 행동 필요
    - 4xx (Client Error) : 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 처리할 수 없음
    - 5xx (Server Error) : 서버 오류, 서버가 정상 요청을 처리하지 못함

만약 모르는 상태 코드가 나타나면?

- 클라이언트가 인식할 수 없는 상태코드를 서버가 반환 시 클라이언트는 상위 상태코드로 해석하여 처리   
ex) 299 ??? -> 2xx (Successful), 451 ??? -> 4xx (Client Error)


### 2xx - 성공

    - 200 OK
    - 201 Created
    - 202 Accepted
    - 204 No Content

#### 200 OK

요청 성공

#### 201 Created

요청 성공해서 새로운 리소스가 생성됨

#### 202 Accepted

요청이 접수되었으나 처리가 완료되지 않았음   

ex) 배치처리 : 요청 접수 후 일정시간 뒤 배치 프로세스가 요청을 처리함

#### 204 No Content

요청이 성공적으로 수행되었지만, 응답 페이로드 본문에 보낼 데이터가 없음

ex) 웹 문서 편집기의 save 버튼, 눌러도 아무 내용이 없고, 같은 화면

### 3xx - 리다이렉션1

    - 300 Multiple Choices
    - 301 Moved Permanently
    - 302 Found
    - 303 See Other
    - 304 Not Modified
    - 307 Temporary Redirect
    - 308 Permanent Redirect

#### 리다이렉션 이해

웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(redirect)

#### 리다이렉션의 일반적인 흐름

1. Client -> Server : 요청

    ```
    GET /event HTTP/1.1
    Host: localhost:8080 
    ```

1. Server -> Clinet : Location 헤더가 있는 응답

    ```
    HTTP/1.1 301 Moved Permanently
    Locatioin: /new-event
    ```

1.  Client -> Server : 새 URI 로 요청

    ```
    GET /new-event HTTP/1.1
    Host: localhost:8080
    ```

1. Server -> Client : 응답

    ```
    HTTP/1.1 200OK
    ...
    ```

#### 종류

    - 영구 리다이렉션 - 특정 리소스의 URI가 영구적으로 이동
        - /members -> /users
        - /event -> /new-event
    - 일시 리다이렉션 - 일시적인 변경
        - 주문 완료 후 주문 내역 화면으로 이동
        - PRG : Post/Redirect/Get
    - 특수 리다이렉션 : 결과 대신 캐시를 사용

#### 영구 리다이렉션

- 301 Moved Permanently
    - 리다이렉트 시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음(MAY)
- 308 Permanent Redirect
    - 301과 기능은 같음
    - 리다이렉트시 요청 메서도아 분문 유지(처음 Post 를 보내면 리다이렉트도 Post 유지)

#### 영구 리다이렉션 301 호름

리다이렉션 호출 시 GET 으로 메서드 변경, Form 데이터가 사라짐

1. Client -> Server : 요청

    ```
    POST /event HTTP/1.1
    Host: localhost:8080 

    name=hello&age=20
    ```

1. Server -> Clinet : Location 헤더가 있는 응답

    ```
    HTTP/1.1 301 Moved Permanently
    Locatioin: /new-event
    ```

1.  Client -> Server : 새 URI 로 요청

    ```
    GET /new-event HTTP/1.1
    Host: localhost:8080
    ```

1. Server -> Client : 응답

    ```
    HTTP/1.1 200OK
    ...
    ```

#### 영구 리다이렉션 308 흐름

리다이렉션 호출 시 POST 메서드 유지, Form 데이터 유지

1. Client -> Server : 요청

    ```
    POST /event HTTP/1.1
    Host: localhost:8080 

    name=hello&age=20
    ```

1. Server -> Clinet : Location 헤더가 있는 응답

    ```
    HTTP/1.1 301 Moved Permanently
    Locatioin: /new-event
    ```

1.  Client -> Server : 새 URI 로 요청

    ```
    POST /new-event HTTP/1.1
    Host: localhost:8080

    name=hello&age=20
    ```

1. Server -> Client : 응답

    ```
    HTTP/1.1 200OK
    ...
    ```

### 3xx - 리다이렉션2

리소스의 URI 가 일시적으로 변경, 따라서 검색 엔진 등에서 URL 을 변경하면 안됨

    - 302 Found
    - 307 Temporary Redirect
    - 303 See Other

#### 302 Found

리다이렉트시 요청 메서드가 GET 으로 변하고, 본문이 제거될 수 있음(MAY)

#### 307 Temporary Redirect

302와 기능은 같음

리다이렉트시 요청 메서드와 본문 유지(요청 메서드를 변경하면 안됨, MUST NOT)


#### 303 See other

302 와 기능은 같은

리다이렉트시 요청 메서드가 GET 으로 변경, 본문이 무조건 제거됨

#### Post/Redirect/Get

Post 로 주문데이터 처리를 요청한 후에 웹 브라우저를 새로고침하면?

1. Client -> Server : 주문데이터 처리 요청

    ```
    POST /order HTTP/1.1
    Host: localhost:8080 

    itemId=mouse&count=1
    ```

1. Server -> Clinet : 주문이 처리되고, 응답

    ```
    HTTP/1.1 200 OK
    <html>주문완료</html>
    ```

1.  Client -> Server : 결과화면에서 새로고침, 주문데이터 처리 요청

    ```
    POST /order HTTP/1.1
    Host: localhost:8080

    itemId=mouse&count=1
    ```

1. Server -> Client : 주문이 중복 처리되고, 응답

    ```
    HTTP/1.1 200 OK
    <html>주문완료</html>
    ```

일시적인 리다이렉션으로 해당 문제를 해결할 수 있음

1. Client -> Server : 주문데이터 처리 요청

    ```
    POST /order HTTP/1.1
    Host: localhost:8080 

    itemId=mouse&count=1
    ```

1. Server -> Clinet : 주문이 추가되고, 리다이렉트 응답

    ```
    HTTP/1.1 302 FOUND
    Location: /order-result/19
    ```

1.  Client -> Server : 리다이렉트된 주문 결과화면 요청

    ```
    GET /order-result/19 HTTP/1.1
    Host: localhost:8080
    ```

1. Server -> Client : 응답

    ```
    HTTP/1.1 200 OK
    <html>주문결과</html>
    ```

1.  Client -> Server : 결과화면에서 새로고침, 주문 결과화면 요청

    ```
    GET /order-result/19 HTTP/1.1
    Host: localhost:8080
    ```

1. Server -> Client : 응답

    ```
    HTTP/1.1 200 OK
    <html>주문결과</html>
    ```

#### 302, 307, 303

- 잠깐 정리
    - 302 Found -> GET 으로 변할 수 있음
    - 307 Temporary Redirect -> 메서드가 변하면 안됨
    - 303 See Other -> 메서드가 GET 으로 변경
- 역사
    - 처음 302 스펙의 의도는 HTTP 메서드를 유지하는 것
    - 그런데 웹 브라우저들이 대부분 GET 으로 바꾸어버림(일부는 다르게 동작)
    - 그래서 모호한 302 를 대신하는 명확한 307, 303 이 등장(301 대응으로 308도 등장)
- 현실
    - 307, 303 을 권장하지만 현실적으로 이미 많은 어플리케이션 라이브러리들이 302를 기본값으로 사용
    - 자동 리다이렉션시에 GET 으로 변해도 되면 그냥 302 를 사용해도 큰 문제 없음

#### 기타 리다이렉션

- 300 Multiple Choices : 안씀
- 304 Not Modified
    - 캐시를 목적으로 사용
    - 클라이언트에게 리소스가 수정되지 않았음을 알려줌. 따라서 클라이언트는 로컬 PC에 저장된 캐시를 재사용함 (캐시로 리다이렉트)
    - 304 응답은 응답에 메시지 바디를 포함하면 안됨 (로컬 캐시를 사용해야 하므로)
    - 조건부 GET, HEAD 요청 시 사용

### 4xx - 클라이언트 오류, 5xx -서버 오류

#### 4xx - 클라이언트 오류

- 클라이언트의 요청에 잘못된 문법등으로 서버가 요청을 수행할 수 없음
- 오류의 원인이 클라이언트에 있음
- 클라이언트가 이미 잘못된 요청, 데이터를 보내고 있기 때문에, 똑같은 재시도가 계속 실패

#### 400 Bad Request

클라이언트가 잘못된 요청을 해서 서버가 요청을 처리할 수 없음

- 요청 구문, 메시지 등등 오류
- 클라이언트는 요청 내용을 다시 검토하고 보내야 함

#### 401 Unauthorized

클라이언트가 해당 리소스에 대한 인증이 필요함, Authentication 문제(이름과 다른 뜻)

- 인증(Authentication) 되지 않음
- 401 오류 발생시 응답에 WWW-Authenticate 헤더와 함께 인증 방법을 설명 

    - 인증(Authentication): 본인이 누구인지 확인   
    ex) 로그인
    - 인가(Authorization): 권한부여   
    ex) 특정 리소스에 접근할 수 있는 권한
    - 인증이 있어야 인가가 있음

#### 403 Forbidden

서버가 요청을 이해했지만 승인을 거부함, Authorization 문제

- 주로 인증 자격 증명은 있지만, 접근 권한이 불충분한 경우
- 예) 어드민 등급이 아닌 사용자가 로그인은 했지만, 어드민 등급의 리소스에 접근하는 경우

#### 404 Not Found

요청 리소스를 찾을 수 없음

- 요청 리소스가 서버에 없음
- 또는 클라이언트가 권한이 부족한 리소스에 접근할 때 해당 리소스를 숨기고 싶을 때

#### 5xx - 서버 오류

- 서버 문제로 오류 발생
- 서버에 문제가 있기 때문에 재시도하면 성공할 수도 있음

#### 500 Internal Server Error

서버 문제로 오류 발생, 애매하면 500 오류

- 서버 내부 문제로 오류 발생
- 애매하면 500 오류

#### 503 Service Unavailable

서비스 이용 불가

- 서버가 일시적인 과부하 또는 예정된 작업으로 잠시 요청을 처리할 수 없음
- Retry-After 헤더 필드로 얼마뒤에 복구되는지 보낼 수도 있음

## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
- [https://restfulapi.net/resource-naming](https://restfulapi.net/resource-naming)