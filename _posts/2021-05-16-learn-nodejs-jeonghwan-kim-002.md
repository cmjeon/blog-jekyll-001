---
layout: single
title:  테스트주도개발(TDD)로 만드는 NodeJS API 서버 - 002
categories: 
  - learning nodejs by jeonghwan-kim
tags: 
  - nodejs
  - expressjs
  - rest-api
  - backend
---

## 익스프레스(ExpressJS) 기초

### 익스프레스(ExpressJS) 소개

ExpressJS 는 nodejs 로 만들어진 간결한 웹프레임워크

ExpressJS 기능

- 어플리케이션
- 미들웨어
- 라우팅
- 요청객체
- 응답객체

ExpressJS 설치

```bash
# node-api/
$ npm i express
```

### 어플리케이션

익스프레스 인스턴스를 어플리케이션이라 함

서버에 필요한 기능인 미들웨어를 어플리케이션에 추가함

라우팅 설정을 할 수 있음

서버를 요청 대기 상태로 만들 수 있음

index.js

```js
// node-api/index.js
const express = require('express');
const app = express(); // 어플리케이션
app.listen(3000, function(){
  console.log('Server is running');
});
```

### 미들웨어 만들기

미들웨어는 함수들의 연속이다

로깅 미들웨어를 만들어 보자

써드파티 미들웨어를 사용해 보자

일반 미들웨어 vs 에러 미들웨어

404, 500 에러 미들웨어를 만들어 보자

index.js

```js
// node-api/index.js
const express = require('express');
const app = express(); // 어플리케이션

function logger(req, res, next) {
  console.log('I am logger');
  next();
}

app.use(logger);

app.listen(3000, function(){
  console.log('Server is running');
});
```

실행

```bash
# node-api/
$ node index.js
Server is running
```

접속확인

```bash
$ curl -X GET 'localhost:3000'
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>Cannot GET /</pre>
</body>
</html>
```

서버로그

```bash
Server is running
I am logger
```

### 미들웨어 실행 순서

미들웨어 추가하기

index.js

```js
// node-api/index.js
...
function logger(req, res, next) {
  console.log('I am logger');
  next();
}

function logger2(req, res, next) {
  console.log('I am logger2');
  next();
}

app.use(logger);
app.use(logger2);
...
```

실행

```bash
# node-api/
$ node index.js
Server is running
```

접속확인

```bash
$ curl -X GET 'localhost:3000'
# 실행결과 출력
```

서버로그

```bash
Server is running
I am logger
I am logger2
```

미들웨어에서 next() 호출하는 것은 중요

만약 next() 를 호출하지 않으면 `I am logger` 에서 멈춤

### 다른 개발자가 만든 미들웨어 사용하기

expressJS 에서 외부 logger(morgan) 사용하기

```bash
# node-api/
$ npm insall morgan
```

```js
// node-api/index.js
const express = require('express');
const morgan = require('morgan');

const app = express();
...
app.use(logger2);
app.use(morgan('dev'));
...
```

실행

```bash
# node-api 폴더에서
$ node index.js
Server is running
```

접속확인

```bash
$ curl -X GET 'localhost:3000'
# 실행결과 출력
```

서버로그

```bash
Server is running
I am logger
I am logger2
GET / 404 2.096 ms - 139
```

### 에러 미들웨어

미들웨어는 일반 미들웨어 / 에러 미들웨어가 있음

일반 미들웨어는 param 을 3개 받음 : req, res, next

에러 미들웨어는 param 을 4개 받음 : err, req, res, next

```js
// node-api/index.js
const express = require('express');
const app = express();

function commonmw(req, res, next) {
  console.log('commonmw');
  next(new Error('error ouccered'));
}

function errormw(err, req, res, next) {
  console.log(err.message);
  // 에러를 처리하거나
  next();
}

app.use(commonmw);
app.use(errormw);
app.use(morgan('dev'));

app.listen(3000, fuction() {
  console.log('Server is running');
})
```

실행

```bash
# node-api 폴더에서
$ node index.js
Server is running
```

접속확인

```bash
$ curl -X GET 'localhost:3000'
```

서버로그

```bash
Server is running
commonmw
error ouccered
```

### 라우팅

요청 URL 에 대해 적절한 핸들러 함수를 연결해주는 기능을 라우팅이라고 부름

어플리케이션의 get(), post() 메소드로 구현할 수 있음

라우팅을 위한 전용 Router 클래스를 사용할 수 있음

```js
// node-api/index.js
const http = require('http');
const hostname = '127.0.0.1';
const port = 3000;

const server =http.createServer((req, res) => {
  if(req.url === '/') {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plan');
    res.end('Hello World\n');
  } else if(req.url === '/users') {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plan');
    res.end('User list');
  } else {
    res.statusCode = 404;
    res.end('Not Found');
  }
});

server.listen(port, hostname, () => {
  console.log(`Server running as http://${hostname}:${port}/`);
});
```

어떤 요청이 왔을 때 그 요청에 해당하는 응답을 연결시켜 주는 것을 라우팅이라고 함

http 모듈에서는 라우팅이 위처럼 다소 장황하게 작성됨

### 요청객체와 응답객체

#### 요청객체

클라이언트 요청 정보를 담은 객체를 요청(Request) 객체라고 함

http 모듈의 request 객체를 래핑한 것

req.params(), req.query(), req.body() 메소드를 주로 사용함

#### 응답객체

클라이언트 응답 정보를 담은 객체를 응답(Response) 객체라고 함

http 모듈의 response 객체를 래핑한 것

res.send(), res.status(), res.json() 메소드를 주로 사용함

### Hello world 익스프레스 버전

index.js

```js
// node-api/index.js
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```

## npm 에 대해 좀 더 알아보기

### npm1

```bash
$ npm install express
```

위 명령어로 express 를 설치했는데 어디에 설치되었을까?

node_modules 폴더에 설치됨

node_modules 폴더에 가면 express 말고 다른 모듈들이 있음

express 가 사용하는 다른 모듈들이 같이 설치된 것

git 에 소스를 올릴 때는 node_modules 폴더는 올리지 않음, 용량문제

그렇게 되면 git 에 index.js 를 올리더라도 다른 사용자는 실행해 볼 수 없음

`npm init` 을 통해 package.json 파일을 만들어 올릴 수 있음

```bash
$ npm init
```

package.json 안에 dependencies 에 express 를 추가해줌

만약 node_modules 가 없는 폴더에서 `npm init` 을 하게 되면 dependencies 는 비어있음

`npm install express --save` 를 통해 설치와 동시에 package.json 의 dependencies 에 추가할 수 있음

```bash
$ npm install express --save
```

이 후 package.json 의 dependencies 에 모듈이 있는 상황에서는 `npm install` 만으로 모듈들을 설치할 수 있음

```bash
$ npm install
```

### npm2

"script" 에 "start" 추가

package.json

```js
{
  "name": "node-api",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "dependencies": {
    "express": "^4.15.2",
    "morgan": "^1.8.1",
  },
  "devDependencies": {},
  "script": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "author": "",
  "license": "ISC"
}
```

실행

```bash
# node-api 폴더에서
$ npm start

Example app listening on port 3000!
```

## REST API란

### 요청 형식

HTTP 메서드

자원에 대한 행동을 나타냄

- GET : 자원을 조회
- POST : 자원을 생성
- PUT : 자원을 갱신
- DELETE : 자원을 삭제

이는 익스프레스 어플리케이션의 메소드로 구현되어 있음

REST API 예시

```bash
# 유저목록을 조회할 때
$ curl -X GET 'localhost:3000/users'
# 유저목록 중 1번회원을 조회할 때
$ curl -X GET 'localhost:3000/users/1'
```

index.js

```js
// node-api/index.js
...
// get 방식 요청에 대한 처리
app.get('/users', function(req, res) {
  res.send('users list');
});
// post 방식 요청에 대한 처리
app.post('/users', function(req, res) {
  // create user...
  res.send(user);
})
...
```

### 응답 형식

HTTP 상태코드

- 1xx : 아직 처리중
- 2xx : 자, 여기있어
    - 200 : 성공(success), GET, PUT
    - 201 : 작성됨(created), POST
    - 204 : 내용 없음(No Content), DELETE
- 3xx : 잘 가~
- 4xx : 니가 문제임
    - 400 : 잘못된 요청(Bad Request)
    - 401 : 권한 없음(Unauthorized)
    - 404 : 찾을 수 없음(Not found)
    - 409 : 충돌(Conflict)
- 5xx : 내가 문제임
    - 500 : 서버 에러(Internal server error)

### 첫번째 API 만들기: 사용자 목록 조회 API

GET /users

사용자 목록을 조회하는 기능



```js
// node-api/index.js
var express = require('express');
var morgan = require('morgan');
var app = express();

var users = [
  {id: 1, name: 'alice'},
  {id: 2, name: 'beck'},
  {id: 3, name: 'chris'},
];

app.use(morgan('dev'));

app.get('/users', function(req, res) {
  res.json(users);
});

app.listen(3000, function() {
  console.log('Example app listening on port 3000!');
})
```

실행

```bash
# node-api 폴더에서
$ npm start
Example app listening on port 3000!
```

요청

```bash
$ curl -X GET 'localhost:3000/users' -v
```

서버로그

```bash
Example app listening on port 3000!
GET /users 200 2.815 ms - 71
```

요청 결과

```bash
Note: Unnecessary use of -X or --request, GET is already inferred.
* Trying 127.0.0.1...
* TCP_NODELAY set
* connected to 127.0.0.1 (127.0.0.1) port 3000 (#0)
# > 은 요청값
> GET /usrs HTTP/1.1
> Host: 127.0.0.1:3000
> User-Agent: curl/7.51.0
> Accept: */*
>
# < 은 응답값
< HTTP/1.1 200 OK
< X-Powered-By: Express
< Content-Type: application/json; charset=utf-8
< Content-Length: 71
...
[{"id":1,"name":"alice"},{"id":2,"name":"beck"},{"id":3,"name":"chris"}]
```

## 참고
- [https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api)
- [http://expressjs.com/](http://expressjs.com/)