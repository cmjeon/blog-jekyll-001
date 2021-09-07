---
layout: single
title:  모든 개발자를 위한 HTTP 웹 기본 지식 - 007
categories: 
  - learning http basic by younghan kim
tags: 
  - http
---

## HTTP 헤더2 - 캐시와 조건부 요청

### 캐시 기본 동작

#### 캐시가 없을 때

- 데이터가 변경되지 않아도 계속 네트워크를 통해 데이터를 받아야 함
- 인터넷 네트워크는 느리고 비쌈
- 브라우저 로딩 속도가 느려짐 -> 느린 사용자 경험

#### 캐시 적용

1. 클라이언트 -> 서버 : 요청

    ```
    GET /star.jpg
    ```

1. 서버 -> 클라이언트 : 응답

    ```
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    caache-control: max-age=60
    Content-Length: 34012

    akdfjladfjlkdfjaslj....
    ```

1. 클라이언트는 응답 결과를 캐시에 저장
1. 클라이언트가 요청 시 캐시에서 제공

- 캐시 가능 시간동안 네트워크 필요없음
- 인터넷 네트워크를 줄일 수 있음
- 브라우저 로딩 속도 빠름 -> 빠른 사용자 경험

캐시 가능 시간이 끝나면 같은 데이터를 다시 받아야 할까?

### 검증 헤더와 조건부 요청1

캐시 유효 시간이 초과하여 서버에 다시 요청하면 두가지 케이스가 있을 수 있음

- 서버에서 캐시와 다른 데이터를 받음
- 서버에서 케시와 같은 데이터를 받음

#### 검증 헤더 추가

응답 헤더에 Last-Modified 추가

1. 서버 -> 클라이언트 : 최초요청에 대한 응답

    ```
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    cache-control: max-age=60
    Last-Modified: 2020년 11월 10일 10:00:00
    Content-Length: 34012

    ajldfkjadklfj.....
    ```

1. 클라이언트 -> 서버 : 재요청

    ```
    GET /star.jpg
    if-modified-since: 2020년 11월 10일 10:00:00
    ```

1. 서버 -> 클라이언트 : if-modified 에 대한 응답 (수정이 없는 경우는 헤더로만 응답)

    ```
    HTTP/1.1 304 Not Modified
    Content-Type: image/jpeg
    cache-control: max-age=60
    Last-Modified: 2020년 11월 10일 10:00:00
    Content-Length: 34012

    ```

1. 서버 -> 클라이언트 : if-modified 에 대한 응답 (수정이 있는 경우는 헤더+바디로 응답)

    ```
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    cache-control: max-age=60
    Last-Modified: 2020년 11월 10일 11:00:00
    Content-Length: 34012

    ajfdladsjfsalf...
    ```

1. 클라이언트는 응답 결과를 캐시에 저장(cache-control 갱신)

### 검증 헤더와 조건부 요청2

- 검증 헤더
    - 캐시 데이터와 서버 데이터가 같은지 검증하는 데이터
    - Last-Modified, ETag
- 조건부 요청 헤더
    - 검증 헤더로 조건에 따른 분기 If-Modified-Since: Last-Modified 사용 
    - If-None-Match: ETag 사용
    - 조건이 만족하면 200 OK
    - 조건이 만족하지 않으면 304 Not Modified

#### Last-Modified, If-Modified-Since 단점

- 1초 미만 단위로 캐시 조정이 불가능
- 날짜 기반의 로직
- 다른 날짜인데 같은 데이터인 경우
- 서버에서 별도의 캐시 로직을 관리하지 못함

#### Etag, If-None-Match

- ETag(Entity Tag)
- 캐시용 데이터에 임의의 고유한 버전 이름을 달아둠
- 데이터가 변경되면 이 이름을 바꾸어서 변경함
- ETag를 요청으로 보내서 응답이 같으면 유지, 다르면 다시 받기

#### Etag, If-None-Match 사용방법

응답 헤더에 ETag 추가

1. 서버 -> 클라이언트 : 최초요청에 대한 응답

    ```
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    cache-control: max-age=60
    ETag: "aaaa"
    Content-Length: 34012

    ajldfkjadklfj.....
    ```

1. 클라이언트 -> 서버 : 재요청

    ```
    GET /star.jpg
    If-None-Match: "aaaa"
    ```

1. 서버 -> 클라이언트 : If-None-Match 에 대한 응답 (수정이 없는 경우는 헤더로만 응답)

    ```
    HTTP/1.1 304 Not Modified
    Content-Type: image/jpeg
    cache-control: max-age=60
    ETag: "aaaa"
    Content-Length: 34012

    ```

1. 서버 -> 클라이언트 : If-None-Match 에 대한 응답 (수정이 있는 경우는 헤더+바디로 응답)

    ```
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    cache-control: max-age=60
    ETag: "bbbb"
    Content-Length: 34012

    sdvvczxcvz1111...
    ```

1. 클라이언트는 응답 결과를 캐시에 저장(cache-control 갱신)

- 캐시 제어 로직을 서버에서 완전히 관리, 클라이언트는 캐시 메커니즘을 모름

### 캐시와 조건부 요청 헤더

    - Cache-Control: 캐시 제어
    - Pragma: 캐시 제어(하위 호환)
    - Expires: 캐시 유효 기간(하위 호환)

#### Cache-Control

캐시 지시어(directives)

- Cache-Control: max-age - 캐시 유효 시간, 초 단위
- Cache-Control: no-cache - 데이터는 캐시해도 되지만, 항상 Origin 서버에 검증하고 사용
- Cache-Control: no-store - 데이터에 민감한 정보가 있으므로 저장하면 안됨

#### Pragma

캐시 제어(하위 호환)

- Pragma: no-cache
- HTTP 1.0 하위 호환

#### Expires

캐시 만료일 지정(하위 호환)

- HTTP 1.0 부터 사용
- Cache-Control: max-age 가 더 유연함
- Expires 가 Cache-Control: max-age 와 함께 사용되면 Expires 는 무시

#### 검증 헤더와 조건부 요청 헤더

- 검증 헤더
    - ETag, Last-Modified
- 조건부 요청 해더
    - If-Match, In-None-Match: ETag
    - If-Modified-Since, If-Unmodified-Since: Last-Modified

### 프록시 캐시와 관련된 캐시 지시어

- Cache-Control: public - 응답이 public 캐시에 저장되어도 됨
- Cache-Control: private - 응답이 해당 사용자만을 위한 것임, private 캐시에 저장해야 함(기본값)
- Cache-Control: s-maxage - 프록시 캐시에만 적용되는 max-age 
- Age: 60 (HTTP 헤더) - 오리진 서버에서 응답 후 프록시 캐시 내에 머문 시간(초)

### 캐시 무효화

    - Cache-Control: no-cache, no-store, must-revalidate
    - Pragma: no-cache

#### 캐시 무효화할 때 캐시 지시어

- Cache-Control: no-cache - 데이터는 캐시해도 되지만, 항상 원 서버에 검증하고 사용(이름에 주의!)
- Cache-Control: no-store - 데이터에 민감한 정보가 있으므로 저장하면 안됨  (메모리에서 사용하고 최대한 빨리 삭제)
- Cache-Control: must-revalidate
    - 캐시 만료후 최초 조회시 원 서버에 검증해야함
    - 원 서버 접근 실패시 반드시 오류가 발생해야함 - 504(Gateway Timeout)
    - must-revalidate는 캐시 유효 시간이라면 캐시를 사용함
- Pragma: no-cache - HTTP 1.0 하위 호환

#### no-cache vs must-revalidate

프록시가 Origin 서버에 접근할 수 없어서 검증이 불가능한 상황에서

- no-cache 의 경우 - 오래된 데이터라도 보여주도록 조정할 수 있음
- no-revalidate - 반드시 오류페이지가 발생

## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
- [https://restfulapi.net/resource-naming](https://restfulapi.net/resource-naming)