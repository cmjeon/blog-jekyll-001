---
layout: single
title:  테스트주도개발(TDD)로 만드는 NodeJS API 서버 - 004
categories: 
  - learning nodejs by jeonghwan-kim
tags: 
  - nodejs
  - expressjs
  - rest-api
  - backend
---

## TDD 로 하는 API 서버 개발

### 사용자 목록 조회 API 테스트 코드 만들기 1

성공 테스트

> 유저 객체를 담은 배열로 응답한다    

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  describe('성공시', () => {
    it('유저 객체를 담은 배열로 응답한다', (done) => {
      request(app)
        .get('/users')
        .end((err, res) => {
          res.body.should.be.instanceOf(Array);
          done();
        });
    });
  })
})
```

### NPM 테스트 스크립트

package.json 에 테스트 스크립트 등록

```js
{
  ...
  "scripts": {
    "test": "mocha index.spec.js",
    "start" : "node index.js"
  }
  ...
}
```

테스트 실행

```bash
$ npm test
# 테스트 통과
```

### 사용자 목록 조회 API 테스트 코드 만들기 2

성공

> 최대 limit 갯수만큼 응답한다

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  describe('성공시', () => {
    ...
    it('최대 limit 갯수만큼 응답한다', (done) => {
      request(app)
        .get('/users?limit=2')
        .end((err, res) => {
          res.body.should.be.lengthOf(2);
          done();
        });
    });
  })
})
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.get('/users', function(req, res) {
  const limit = req.query.limit;
  res.json(users.slice(0, limit));
});
...
```

실패

> limit 이 숫자형이 아니면 400을 응답한다   

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
  describe('실패시', () => {
    it('limit 이 숫자형이 아니면 400을 응답한다', (done) => {
      request(app)
        .get('/users?limit=two')
        .expect(400)
        .end(done);
    });
  });
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.get('/users', function(req, res) {
  const limit = parseInt(req.query.limit || 10);
  if(Number.isNaN(limit)) {
    return res.status(400).end();
  }
  res.json(users.slice(0, limit));
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

`유저 객체를 담은 배열로 응답한다` 테스트가 실패됨

index.js

```js
// node-api/index.js
...
app.get('/users', function(req, res) {
  req.query.limit = req.query.limit || 10;
  const limit = parseInt(req.query.limit || 10);
  if(Number.isNaN(limit)) {
    return res.status(400).end();
  }
  res.json(users.slice(0, limit));
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

### 사용자 조회 API 성공시

성공 

> id 가 1인 유저 객체를 반환한다

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
});
describe('GET /users/:id는', () => {
  describe('성공시', () => {
    it('id 가 1인 유저 객체를 반환한다', (done) => {
      request(app)
        .get('/users/1')
        .end((err, res) => {
          res.body.should.be.have.property('id', 1)
          done();
        });
    });
  });
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.get('/users/:id', function(req, res) {
  const id = parseInt(req.params.id, 10);
  const user = users.filter((user) => { 
    return user.id == id 
  })[0];
  res.json(user);
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

### 사용자 조회 API 실패시

실패

> id 가 숫자가 아닐 경우 400으로 응답한다   
> id 로 유저를 찾을 수 없을 경우 404로 응답한다

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
});
describe('GET /users/:id는', () => {
  ...
  describe('성공시', () => {
    ...
  });
  describe('실패시', () => {
    it('id가 숫자가 아닐 경우 400으로 응답한다', (done) => {
      request(app)
        .get('/users/one')
        .expect(400)
        .end(done);
    });
    it('id로 유저를 찾을 수 없을 경우 404으로 응답한다', (done) => {
      request(app)
        .get('/users/999')
        .expect(404)
        .end(done);
    });
  })
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.get('/users/:id', function(req, res) {
  const id = parseInt(req.params.id, 10);
  if(Number.isNaN(id)) {
    return res.status(400).end();
  }
  const user = users.filter((user) => { user.id = id })[0];
  if(!user) {
    return res.status(404).end();
  }
  res.json(user);
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

### 사용자 삭제 API 성공시

성공 

> 204를 응답한다

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
});
describe('GET /users/:id는', () => {
  ...
});
describe('DELETE /users/:id는', () => {
  describe('성공시', () => {
    it('204를 응답한다', (done) => {
      request(app)
        .delete('/users/1')
        .expect(204)
        .end(done);
    })
  })
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.delete('/users/:id', function(req, res) {
  const id = parseInt(req.params.id, 10);
  users = users.filter((user) => { user.id !== id });
  res.status(204).end();
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

### 사용자 삭제 API 실패시

실패

> id 가 숫자가 아닐 경우 400으로 응답한다   

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
});
describe('GET /users/:id는', () => {
  ...
});
describe('DELETE /users/:id는', () => {
  ...
  describe('실패시', () => {
    it('id가 숫자가 아닐경우 400으로 응답한다', (done) => {
      request(app)
        .delete('/users/one')
        .expect(400)
        .end(done);
    });
  });
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.delete('/users/:id', function(req, res) {
  const id = parseInt(req.params.id, 10);
  if(Number.isNaN(id)) return res.status(400).end();
  const user = users.find((user) => { user == id });
  if(!user) return res.status(404).end();
  users = users.filter((user) => { user.id !== id });
  res.status(204).end();
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

### 사용자 추가 API 성공시

성공 

> 201 상태코드를 반환한다
> 생성된 유저 객체를 반환한다   
> 입력한 name을 반환한다

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
});
describe('GET /users/:id는', () => {
  ...
});
describe('DELETE /users/:id는', () => {
  ...
});
describe('POST /users는', () => {
  describe('성공시', () => {
    let name = 'danial';
    let body;

    before((done) => {
      request(app)
        .post('/users')
        .send({name: 'daniel'})
        .expect(201)
        .end((err, res) => {
          body = res.body;
          done();
        });
    });
    it('생성된 유저 객체를 반환한다', () => {
      body.should.have.property('id');
    });
    it('입력한 name을 반환한다', () => {
      body.should.have.property('name', name);
    });
  });
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

### bodyParser 모듈

express 에서 body 를 다루기 위해서는 body-parser 가 필요함

body-parser 설치

```bash
$ npm install body-parser --save
```

index.js

```js
// node-api/index.js
...
var morgan = require('morgan');
var bodyParser = require('body-parser');
...
app.use(bodyParser.json()); // for parsing application/json
app.use(bodyParser.urlencoded({extended: true})); // for parsing application/x-www-form-urlencoded
...
app.post('/users', function(req, res) {
  const name = req.body.name;
  const id = Date.now();
  const user = { id, name };
  users.push(user);
  res.status(201).json(user);
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

### 사용자 추가 API 실패시

실패

> name 파라메터 누락 시 400을 반환한다   
> name 이 중복일 경우 409을 반환한다

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
});
describe('GET /users/:id는', () => {
  ...
});
describe('DELETE /users/:id는', () => {
  ...
});
describe('POST /users는', () => {
  describe('실패시', () => {
    it('name 파라메터 누락시 400을 반환한다', (done) => {
      request(app)
        .post('/users')
        .send({})
        .expect(400)
        .end(done);
    });
    it('name 이 중복일 경우 409를 반환한다', (done) => {
      request(app)
        .post('/users')
        .send({name:'daniel'})
        .expect(409)
        .end(done);
    });
  });
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.post('/users', function(req, res) {
  const name = req.body.name;
  if(!name) return res.status(400).end();
  if(users.filter((user) => { return user.name === name}).length) return res.status(409).end();
  const id = Date.now();
  const user = { id, name };
  users.push(user);
  res.status(201).json(user);
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

### 사용자 수정 API 성공시

성공 

> 변경된 name 을 응답한다

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
});
describe('GET /users/:id는', () => {
  ...
});
describe('DELETE /users/:id는', () => {
  ...
});
describe('POST /users는', () => {
  ...
});
describe('PUT /users/:id는', () => {
  describe('성공시', () => {
    it('변경된 name 을 응답한다', (done) => {
      const name = 'charlie ';
      request(app)
        .put('/users/3')
        .send({
          name: name
        })
        .end((err, res) => {
          res.body.should.have.property('name', name);
          done();
        });
    });
  })
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.put('/users/:id', (req, res) => {
  const id =parseInt(req.params.id, 10);
  const name = req.body.name;

  users.filter((user) => user.id === id)[0];
  user.name = name;

  res.json(user);  
});
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

### 사용자 수정 API 실패시

실패

> 정수가 아닌 id일 경우 400 응답
> name 이 없을 경우 400 응답   
> 없는 유저일 경우 404 응답
> 이름이 중복일 경우 409 응답

index.spec.js

```js
// node-api/index.spec.js
const app = require('./index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users는', () => {
  ...
});
describe('GET /users/:id는', () => {
  ...
});
describe('DELETE /users/:id는', () => {
  ...
});
describe('POST /users는', () => {
  ...
});
describe('PUT /users/:id는', () => {
  ...
  describe('실패시', () => {
    it('정수가 아닌 id 일 경우 400 응답', (done) => {
      request(app)
        .put('/users/one')
        .send({})
        .expect(400)
        .end(done);
    });
    it('name 이 없을 경우 400 응답', (done) => {
      request(app)
        .put('/users/1')
        .send({})
        .expect(400)
        .end(done);
    });
    it('없는 유저일 경우 404 응답', (done) => {
      request(app)
        .put('/users/999')
        .send({name:'foo'})
        .expect(404)
        .end(done);
    });
    it('이름이 중복일 경우 409 응답', (done) => {
      request(app)
        .put('/users/3')
        .send({name: 'beck'})
        .expect(409)
        .end(done);
    });
  });
});
```

테스트 실행

```bash
$ npm test
# 테스트 실패
```

index.js

```js
// node-api/index.js
...
app.put('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  if(Number.isNaN(id)) return res.status(400).end();
  
  const name = req.body.name;
  if(!name) return res.status(400).end();
  if(users.filter((users) => {user.name === name}).length) return res.status(409).end();

  const uesr = users.filter((user) => { user.id === id})[0];
  if(!user) return res.status(404).end();
  
  user.name = name;
  res.json(user);
});
...
})
...
```

테스트 실행

```bash
$ npm test
# 테스트 성공
```

## 참고
- [https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api)
- [http://expressjs.com/](http://expressjs.com/)