---
layout: single
title: jquery ajax의 이해
categories: 
  - jquery
tags: 
  - jquery
  - ajax
---

## jquery 란
- jquery는 javascript library임
- jquery는 다양한 (비표준) 브라우저에서 javascript가 올바르게 작동가능하도록 함
- 비동기식 HTTP Request를 수행하는 함수 ajax가 있음

## jquery ajax options 사용법
- ajax는 옵션들을 object에 넣어서 파라메터로 전달
```javascript
# ajax 예제
$.ajax({
  type : "POST",
  url : "your url",
  dataType : "json",
  data : "your Data",
  traditional : true,    // or false, your choice
  async : false,    // or true, your choice
  beforeSend : function(xhr, opts) {
    // when validation is false
    if (false) {
      xhr.abort();
    }
  },
  success : function() {
    // success code
  },
  error : function() {
    // error code
  }
});
```

## jquery ajax 주요 options
## url
- 기본옵션, Request가 보내져야 하는 URL
```javascript
$.ajax({
  url: "test.html",
})
...
```

### async
- type : boolean
- 기본값 true로 비동기식 호출
- false로 지정하면 동기식 호출을 수행함
- cross-domain이나 datatype이 jsonp인 경우는 동기식 호출 불가
```javascript
$.ajax({
  async:false,
})
```

### beforesend
- type : funciton 
- 호출전 수행하는 함수를 지정

### complete
- type : function
- success나 error 콜백함수가 실행되고 난 후 마지막에 실행되는 함수

### data
- type : String, Array
- 서버에 보내지는 데이터
- GET 방식같이 entity body가 없는 경우는 URL의 뒤에 붙음

### dataType
- type : String
- 기본값은 MIME Type을 통해 추론
- 서버로부터 받고자 기대하는 데이터의 타입

### error
- type : function
- 요청이 실패했을 경우 실행되는 함수

### headers
- type : object
- 요청 헤더에 추가할 헤더정보
- 기본으로 X-Requested-With: XMLHttpRequest이 포함
- beforeSend 함수에서 수정가능

### method
- type : String
- Request에 사용할 HTTP method

### success
- type : function
- 요청이 성공했을 경우 실행되는 콜백함수

### timeout
- type : number
- 요청의 제한시간
- 0은 제한시간이 없음

### type
- method의 유의어

## 참고
- https://api.jquery.com/jquery.ajax/
