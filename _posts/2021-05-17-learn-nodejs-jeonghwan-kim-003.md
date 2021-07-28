---
layout: single
title:  테스트주도개발(TDD)로 만드는 NodeJS API 서버 - 003
categories: 
  - learning nodejs by jeonghwan-kim
tags: 
  - nodejs
  - expressjs
  - rest-api
  - backend
---

## 테스트 주도 개발 (TDD)

### 테스트 주도 개발이란?

Test Driven Development

개발시간은 많이 걸리지만 유지운영 시점에는 큰 도움이 됨

### 모카(mocha) 1

[https://mochajs.org/](https://mochajs.org/)

모카(mocha)는 테스트 코드를 돌려주는 테스트 러너

테스트 수트(suite) : 테스트 환경으로 모카는 describe() 로 구현

테스트 케이스 : 실제 테스트를 말하며 모카에서는 it() 으로 구현

### 모카(mocha) 2

utils.js

```js
// utils/js
function capitalize(str) {
  return str;
}
module.exports = {
  capitalize: capitalize
}
```

utils.spec.js

```js
// utils.spec.js
// spec 이라고 쓰여있는 파일은 주로 테스트 코드 파일
const utils = require('/.utils');

describe('utils.js 모듈의 capitalize() 함수는 ', () => {
  it('문자열의 첫번째 문자를 대문자로 변환한다', () => {
    // ...
  })
})
```

### 모카(mocha) 3

테스트 코드 작성

utils.spec.js

```js
// utils.spec.js
// spec 이라고 쓰여있는 파일은 주로 테스트 코드 파일
const utils = require('./utils');
const assert = require('assert');

describe('utils.js 모듈의 capitalize() 함수는 ', () => {
  it('문자열의 첫번째 문자를 대문자로 변환한다', () => {
    const result = utils.capitalize('hello')
    assert.equal(result, 'Hello');
  })
})
```

모카가 위의 테스트 코드를 실행해준다

mocha 설치

```bash
# node-api/
$ npm install mocha --save-dev
```

mocha 실행

```bash
$ node_modules/.bin/mocha utils.spec.js
# 테스트 실패
```

테스트를 통과하기 위해 utils 수정

utils.js

```js
// utils/js
function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}
module.exports = {
  capitalize: capitalize
}
```

mocha 실행

```bash
$ node_modules/.bin/mocha utils.spec.js
# 테스트 통과
```

### 슈드(should)

[https://github.com/tj/should.js/](https://github.com/tj/should.js/)

nodejs 의 권고 assert 말고 서드파티 라이브러리를 사용하라

슈드(should) 는 검증(assertion) 라이브러리

기독성 높은 테스트 코드를 만들 수 있음

should 설치

```bash
# node-api/
$ npm install should --save-dev
```


utils.spec.js

```js
// utils.spec.js
const utils = require('./utils');
const should = require('should');

describe('utils.js 모듈의 capitalize() 함수는 ', () => {
  it('문자열의 첫번째 문자를 대문자로 변환한다', () => {
    const result = utils.capitalize('hello')
    // assert.equal(result, 'Hello'); 삭제
    result.should.be.equal('Hello');
  })
})
```

mocha 실행

```bash
$ node_modules/.bin/mocha utils.spec.js
# 테스트 통과
```

### 슈퍼테스트(superTest) 1

단위 테스트 : 함수의 기능 테스트

통합 테스트 : API 의 기능 테스트

슈퍼테스트는 익스프레스 통합 테스트용 라이브러리

내부적으로 익스프레스 서버를 구동시켜 실제 요청을 보낸 뒤 결과를 검증

[https://github.com/visionmedia/supertest](https://github.com/visionmedia/supertest)

### 슈퍼테스트(superTest) 2

supertest 설치

```bash
$ npm install supertest --save-dev
```

테스트 케이스 생성

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');

describe('GET /users는', () => {
  it('...', (done) => { // done
    request(app)
      .get('/users')
      .end((err, res) => {
        console.log(res.body);
        done();
      })
  })
})
```

index.js

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

module.exports = app;
```

mocha 실행

```bash
$ node_modules/.bin/mocha index.spec.js
# 테스트 통과
```

## 참고
- [https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api)
- [http://expressjs.com/](http://expressjs.com/)