---
layout: single
title:  테스트주도개발(TDD)로 만드는 NodeJS API 서버 - 001
categories: 
  - learning nodejs by jeonghwan-kim
tags: 
  - nodejs
  - expressjs
  - rest-api
  - backend
---

## 오리엔테이션

### 강의소개

노드로 API 서버 만들기

테스트 주도 개발 익히기

### 개발 환경 구성

nodejs 설치 확인

```bash
$ node -v
$ npm -v
```

비쥬얼 스튜디오설치

## NodeJS 기초

### V8 엔진

브라우저 밖에서 자바스크립트 코드를 실행 할 수 있다

크롬에서 사용하는 V8 엔진을 사용한다

이벤트 기반의 비동기 I/O 프레임워크

CommonJS를 구현한 모듈 시스템

### 이벤트기반 비동기 I/O

1. 클라이언트가 nodejs 서버로 HTTP 요청을 보냄
1. nodejs 서버는 HTTP 요청을 이벤트로 만들어서 event queue 에 보관
1. event loop 가 이벤트큐에 있는 이벤트를 하나씩 꺼내서 job 으로 처리함 (`이벤트기반`)
1. 이 event loop 가 single thread 임
1. 바로 응답할 수 있는 job 은 event loop 가 처리하여 클라이언트로 리턴함
1. 바로 응답할 수 없는 job 은 Non-blocking Worker 에게 위임하여 비동기로 처리함 (`비동기 I/O`)
1. Non-blocking Worker 는 작업을 완료하면 callback 으로 event loop 에 돌려줌

### 모듈시스템1

브라우저에서는 모듈을 만들 때 윈도우 객체를 사용

```javascript
//  브라우저의 개발자 모드
> window.module1 = function() { return 'myModule1'; }
> module1();
"myModule1";
> windows.module1();
"myModule1";
```

혹은 RequireJS 같은 의존성 로더를 사용

nodejs 는 파일형태로 모듈을 관리할 수 있는 CommonJS 로 구현

### 모듈시스템2

모듈사용 사례

node-api 폴더에 index.js 생성

```javascript
// node-api/index.js

// http 라는 모듈을 가져와서 http 라는 변수에 할당
const http = require('http');

// http 모듈의 createServer() 함수 실행
http.createServer();
```

모듈 만들기

math.js 생성

```javascript
// node-api/math.js
function sum(a, b) {
  return a + b;
}

module.exports = {
  sum: sum
}
```

index.js 에서 만든 모듈 사용하기

```javascript
// node-api/index.js
const math = require('./math.js');
const result = math.sum(1, 2);
console.log(result);
```

실행

```bash
# node-api 폴더에서
$ node index.js
3
```

### 비동기 세계 1 - readFileSync

노드는 기본적으로 비동기로 동작함

- readFile() vs readFileSync() 의 이해

data.txt 생성

```text
// node-api/data.txt
This is data file
```

index.js

```js
// node-api/index.js
const fs = require('fs');
const data = fs.readFileSync('./data.txt', 'utf8');
console.log(data);
console.log('end');
```

실행

```bash
# node-api 폴더에서
$ node index.js
This is data file
end
```

### 비공기 세계 2 - readFile

index.js

```js
// node-api/index.js
const fs = require('fs');
const data = fs.readFile('./data.txt', 'utf8', function(err, data){
  console.log(data);
});
console.log('end');
```

실행

```bash
# node-api 폴더에서
$ node index.js
end
This is data file
```

## 노드로 만나는 Hello World

### Hello World 노드버전

index.js

```js
// node-api/index.js
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

실행

```bash
# node-api 폴더에서
$ node index.js
Server runnin at http://localhost:3000/
```

접속확인

```bash
$ curl -X GET 'localhost:3000'
```

### 헬로월드 코드읽기

index.js

```js
// node-api/index.js
const http = require('http');  // http 모듈을 가져옴

const hostname = '127.0.0.1'; // hostname 선언, 할당
const port = 3000; // port 선언, 할당

const server = http.createServer((req, res) => { // http 의 createServer 함수를 실행
  // 파라메터로 (req, res) ~ res.end 까지 전달
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!\n');
});

server.listen(port, hostname, () => { // server 의 listen 을 실행해서 클라이언트 요청 대기 상태로 만듬
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

### 라우팅 추가하기

현재 index.js 는 어떤 경로를 호출하더라도 'Hello, World!' 라고 응답합

호출하는 경로에 따라 다른 응답을 하도록 개선

index.js

```js
// node-api/index.js
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  if(req.url === '/') {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello, World!\n');
  } else if(req.url === '/users') {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('User list');
  } else {
    res.statusCode = 404;
    res.end('Not Found');
  }
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```



## 참고
- [https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api)