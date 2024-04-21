---
layout: single
title:  테스트주도개발(TDD)로 만드는 NodeJS API 서버 - 005
categories: 
  - learning nodejs by jeonghwan-kim
tags: 
  - nodejs
  - expressjs
  - rest-api
  - backend
---

## 코드 리펙토링

### 라우터 클래스

역할에 따라 파일로 분리하자

- api/user/index.js // 라우팅 로직
- api/user/user.ctrl.js // API의 로직
- api/user/user.spec.js // 테스트 코드

api/user 폴더 및 파일 생성

```bash
# /node-api
$ mkdir api/user
$ cd api/user
# /node-api/api/user
$ touch index.js
$ touch user.ctrl.js
$ touch user.spec.js
```

node-api/index.js 에서 라우팅 부분만 node-api/api/user/index.js 로 복사

app.get ~ app.put 까지가 라우팅 부분

node-api/api/user/index.js

```js
// node-api/api/user/index.js
const express = require('express');
const router = express.Router();
const ctrl = require('./user.ctrl');

router.get('/', ctrl.index);
router.get('/:id', ctrl.show);
router.delete('/:id', ctrl.destroy);
router.post('/', ctrl.create);
router.put('/:id', ctrl.update);

module.exports = router;
```

node-api/index.js

```js
// node-api/index.js
var express = require('express');
var app = express();
var morgan = require('morgan');
var user = require('./api/user');

app.use(morgan('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.use('/users', user);

app.get('/', function (req, res) {
  return res.json('Hello');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});

module.exports = app;
```

### 컨트롤러 함수로 분리

node-api/api/user/user.ctrl.js

```js
// node-api/api/user/user.ctrl.js
var users = [
  { id: 1, name: 'alice' },
  { id: 2, name: 'beck' },
  { id: 3, name: 'chris' },
];

let count = users.length;

const index = (req, res) => {
  req.query.limit = req.query.limit || 10;
  const limit = parseInt(req.query.limit, 10);
  if (Number.isNaN(limit)) {
    return res.status(400).end();
  }
  res.json(users.slice(0, limit));
}

const show = (req, res) => {
  const id = parseInt(req.params.id, 10);
  if (Number.isNaN(id)) {
    return res.status(400).end();
  }
  const user = users.filter((user) => {
    return user.id == id;
  })[0];
  if (!user) {
    return res.status(400).end();
  }
  return res.json(user);
}

const destroy = (req, res) => {
  const id = parseInt(req.params.id, 10);
  if (Number.isNaN(id)) {
    return res.status(400).end();
  }
  users = users.filter((user) => {
    return user.id !== id;
  });
  res.status(204).end();
}

const create = (req, res) => {
  const name = req.body.name;
  if (!name) {
    return res.status(400).end();
  }

  const checkName = users.filter((user) => {
    return user.name == name;
  })[0];
  if (checkName) {
    return res.status(409).end();
  }
  count++;
  const id = count;
  const user = { id, name };
  users.push(user);
  res.status(201).json(user);
}

const update = (req, res) => {
  const name = req.body.name;
  const id = parseInt(req.params.id, 10);
  if (!name) {
    return res.status(400).end();
  }
  if (Number.isNaN(id)) {
    return res.status(400).end();
  }
  const userByName = users.filter((user) => {
    return user.name === name;
  })[0];
  if (userByName) {
    return res.status(409).end();
  }
  const user = users.filter((user) => {
    return user.id === id;
  })[0];
  if (!user) {
    return res.status(404).end();
  }

  user.name = name;
  res.json(user);
}

module.exports = {
  index: index,
  show: show,
  destroy: destroy,
  create: create,
  update: update,
};
```

node-api/api/user/index.js

```js
// node-api/api/user/index.js
const express = require('express');
const router = express.Router();
const ctrl = require('./users/user.ctrl);

router.get('/', ctrl.index);
router.get('/:id', ctrl.show);
router.delete('/:id', ctrl.destroy);
router.post('/', ctrl.create);
router.put('/:id', ctrl.udpate);
module.exports = router;
```

### 테스트 코드 이동

node-api/index.spec.js 의 내용을 node-api/api/user/user.spec.js 로 복사

```js
// node-api/api/user/user.spec.js
const app = require('../../index.js');
const request = require('supertest');
const should = require('should');

describe('GET /users', () => {
  describe('성공시', () => {
    it('유저 객체를 담은 배열로 응답한다', (done) => { // done
      request(app)
        .get('/users')
        .end((err, res) => {
          res.body.should.be.instanceOf(Array);
          done();
        });
    });
    it('최대 limit 갯수만큼 응답한다', (done) => {
      request(app)
        .get('/users?limit=2')
        .end((err, res) => {
          res.body.should.be.lengthOf(2);
          done();
        });
    });
  });
  describe('실패시', () => {
    it('limit 이 숫자형이 아니면 400을 응답한다', (done) => {
      request(app)
        .get('/users?limit=two')
        .expect(400)
        .end(done);
    });
  });
});
describe('GET /users/:id', () => {
  describe('성공시', () => {
    it('id가 2인 유저 객체를 반환한다', (done) => {
      request(app)
        .get('/users/2')
        .end((err, res) => {
          res.body.should.have.property('id', 2);
          done();
        });
    });
  });
  describe('실패시', () => {
    it('id가 숫자가 아닐 경우 400으로 응답한다', (done) => {
      request(app)
        .get('/users/one')
        .expect(400)
        .end(done);
    });
  });
  it('id로 유저를 찾을 수 없을 경우 404으로 응답한다', (done) => {
    request(app)
      .get('/users/999')
      .expect(404)
      .end(done);
  });
});
describe('DELETE /users/:id', () => {
  describe('성공시', () => {
    it('204를 응답한다', (done) => {
      request(app)
        .delete('/users/1')
        .expect(204)
        .end(done);
    });
  });
  describe('실패시', () => {
    it('id 가 숫자가 아닐경우 400으로 응답한다', (done) => {
      request(app)
        .delete('/users/one')
        .expect(400)
        .end(done);
    });
  });
});
describe('POST /users', () => {
  describe('성공시', () => {
    let name = 'daniel';
    let body;
    before((done) => {
      request(app)
        .post('/users')
        .send({ name: 'daniel' })
        .expect(201)
        .end((err, res) => {
          body = res.body;
          done();
        });
    });
    it('생성된 유저 객체를 반환한다', (done) => {
      body.should.have.property('id');
      done();
    });
    it('입력한 name을 반환한다', (done) => {
      body.should.have.property('name', name);
      done();
    });
  });
  describe('실패시', () => {
    it('name 파라메터 누락 시 400 을 반환한다', (done) => {
      request(app)
        .post('/users')
        .send({})
        .expect(400)
        .end(done);
    });
    it('name 이 중복일 경우 409 를 반환한다', (done) => {
      request(app)
        .post('/users')
        .send({ name: 'daniel' })
        .expect(409)
        .end(done);
    })
  });
});
describe('PUT /users/:id', () => {
  describe('성공시', () => {
    it('변경된 name 을 반환한다', (done) => {
      const name = 'charlie';
      request(app)
        .put('/users/3')
        .send({
          name: name
        })
        .end((err, res) => {
          res.body.should.have.property('name', name);
          done();
        })
    });
  });
  describe('실패시', () => {
    it('정수가 아닌 id 일 경우 400 을 반환한다.', (done) => {
      request(app)
        .put('/users/one')
        .send({})
        .expect(400)
        .end(done);
    });
    it('name 이 없을 경우 400을 반환한다', (done) => {
      request(app)
        .put('/users/1')
        .send({})
        .expect(400)
        .end(done);
    });
    it('없는 유저일 경우 404 을 반환한다', (done) => {
      request(app)
        .put('/users/999')
        .send({ name: 'foo' })
        .expect(404)
        .end(done);
    });
    it('이름이 중복일 경우 409 을 반환한다', (done) => {
      request(app)
        .put('/users/3')
        .send({ name: 'beck' })
        .expect(409)
        .end(done);
    });
  })
});
```

package.json

```js
{
  ...
  "script": {
    "test": "mocha api/user/user.spec.js",
    "start": "node index.js"
  },
  ...
}
```

### 테스트 환경 개선

실행 시 `NODE_ENV` 를 통해 `test` 변수를 전달

mocha 의 watch option `-w` 으로 소스 코드 수정 시 마다 테스트코드가 자동으로 돌 수 있도록 함

package.json

```js
{
  "script": {
    "test": "NODE_ENV=test mocha api/user/user.spec.js -w",
    "start": "node bin/www.js"
  }
}
```

node-api/index.js

```js
// node-api/index.js
var express = require('express');
var app = express();
var morgan = require('morgan');
var user = require('./api/user');

if (process.env.NODE_ENV !== 'test') {
  app.use(morgan('dev'));
}

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.use('/users', user);

app.get('/', function (req, res) {
  return res.json('Hello');
});

module.exports = app;
```

node-api/bin/www.js

```js
// node-api/bin/www.js
const app = require('../index');
app.listen(3000, () => {
  console.log('Server is running on 3000 port');
});
```

package.json

```js
// node-api/package.json
{
  "name": "api",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "dependencies": {
    "body-parser": "^1.19.0",
    "express": "4.17.1",
    "morgan": "1.10.0"
  },
  "devDependencies": {
    "mocha": "9.0.2",
    "should": "^13.2.3",
    "supertest": "6.1.4"
  },
  "scripts": {
    "test": "NODE_ENV=test mocha api/user/user.spec.js",
    "start": "node bin/www.js"
  },
  "author": "",
  "license": "ISC"
}
```

## 참고
- [https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api)
- [http://expressjs.com/](http://expressjs.com/)
