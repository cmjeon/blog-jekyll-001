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


node-api/api/user/index.js

```js
// node-api/api/user/index.js
const express = require('express');
const router = express.Router();

var users = [
  ...
];

router.get('/', function(req, res) {
  ...
});
router.get('/:id', function(req, res) {
  ...
});
router.delete('/:id', function(req, res) {
  ...
});
router.post('/', function(req, res) {
  ...
});
router.put('/:id', function(req, res) {
  ...
});
module.exports = router;
```

node-api/index.js

```js
// node-api/index.js
...
var bodyParser = require('body-parser');
var user = require('./api/user');
...
app.user(bodyParrser.urlencoded({ extended: true }));
app.use('/users', user);
...
```

### 컨트롤러 함수로 분리

node-api/api/user/user.ctrl.js

```js
// node-api/api/user/user.ctrl.js
var users = [
  ...
];

const index = function (req, res) {
  req.query.limit = req.query.limit || 10;
  const limit = parseInt(req.query.limit, 10);
  if(Number.isNaN(limit)) {
    return res.status(400).end();
  }
  res.json(users.slice(0, limit));
}

const show = function(req, res) {
  const id = parseInt(req.params.id, 10);
  if(Number.isNaN(id)) {
    return res.status(400).end();
  }
  const user = users.filter((user) => { user.id = id })[0];
  if(!user) {
    return res.status(404).end();
  }
  res.json(user);
}

const destroy = function(req, res) {
  const id = parseInt(req.params.id, 10);
  if(Number.isNaN(id)) return res.status(400).end();
  const user = users.find((user) => { user == id });
  if(!user) return res.status(404).end();
  users = users.filter((user) => { user.id !== id });
  res.status(204).end();
}

const create = function(req, res) {
  const name = req.body.name;
  if(!name) return res.status(400).end();
  if(users.filter((user) => { return user.name === name}).length) return res.status(409).end();
  const id = Date.now();
  const user = { id, name };
  users.push(user);
  res.status(201).json(user);
}

const update = (req, res) => {
  const id = parseInt(req.params.id, 10);
  if(Number.isNaN(id)) return res.status(400).end();
  
  const name = req.body.name;
  if(!name) return res.status(400).end();
  if(users.filter((users) => {user.name === name}).length) return res.status(409).end();

  const uesr = users.filter((user) => { user.id === id})[0];
  if(!user) return res.status(404).end();
  
  user.name = name;
  res.json(user);
}

module.experts = {
  index: index,
  show: show,
  destroy: destroy,
  create: create,
  update: update
}
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

node-api/index.spec.js 의 내용을 node-api/api/user/user-spec.js 로 복사

```js
...
cons app = require('../../index');
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

package.json

```js
{
  ...
  "script": {
    "test": "NODE_ENV=test mocha api/user/user.spec.js",
    "start": "node index.js"
  },
  ...
}
```

node-api/index.js

```js
// node-api/index.js
...
var user = require('./api/user');

if(process.env.NODE_ENV !== 'test') {
  app.user(morgan('dev'))
}

app.use(bodyParser.json());
...
/* 삭제
app.listen(3000, function() {
  ...
})
*/
export module = app;
```

node-api/bin/www.js

```js
const app = require('../index');
app.listen(3000, () => {
  console.log('Server is running on 3000 port');
});
```

package.js

```js
{
  "script": {
    "test": "NODE_ENV=test mocha api/user/user.spec.js",
    "start": "node bin/www.js"
  }
}
```

## 참고
- [https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api)
- [http://expressjs.com/](http://expressjs.com/)