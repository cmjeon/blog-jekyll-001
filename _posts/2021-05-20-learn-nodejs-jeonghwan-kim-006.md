---
layout: single
title:  테스트주도개발(TDD)로 만드는 NodeJS API 서버 - 006
categories: 
  - learning nodejs by jeonghwan-kim
tags: 
  - nodejs
  - expressjs
  - rest-api
  - backend
---

## 데이터베이스

### 데이터베이스 소개

SQL

- MySQL, PostgreSQL, Aurora, Sqlite

NoSQL : json 형식

- MongoDB, DynamoDB

In Memory DB

- Redis, Memcashed


### ORM 소개

쿼리

- insert table('name') value('alice');
- select * from users;
- update users set name='bek' where id=1;
- delete from users where id=1;

ORM(Object Relational Mapping)

- 데이터베이스를 객체로 추상화해 놓은 것
- 쿼리를 직접 작성하는 대신 ORM 의 메소드로 데이터 관리할 수 있음
- 노드에서 SQL ORM은 시퀄라이져(Sequelize)가 있음

[https://sequelize.org/master/](https://sequelize.org/master/)

### 노드의 ORM 시퀄라이져

쿼리문 대신 메소드를 사용

- insert table('name') value('alice');   
-> User.create({name: 'alice'})
- select * from users;   
-> User.findAll();
- update users set name='bek' where id=1;   
-> User.update({name: 'bek'}, {where:{id:1}});
- delete from users where id=1;   
-> destory({where:{id:1}});

`User` 는 모델임

모델

- 데이터베이스 테이블을 ORM 으로 추상화한 것
- User 모델
    - sequelize.define(): 모델 정의
    - sequelize.sync() : 데이터베이스 연동

### 모델 정의

sequelize, sqlite3 설치

```bash
# sequelize, sqlite3 설치
$ npm i sequelize sqlite --save
```

node-api/models.js

```js
// node-api/models.js
const Sequelize = require('sequelize');
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: './db.sqlite'
});

const User = sequelize.define('User', {
  name: Sequelize.STRING // varchar 255
});

module.exports = {
  Sequelize,
  sequelize,
  User
}
```

### 데이터베이스 - ORM 동기화

node-api/bin/sync-db.js

```js
const models = require('../models');

module.exports = () => {
  return models.Sequelize.sync(force: true); // 원래의 DB를 삭제하고 새로 생성
}
```

sync-db 실행

```bash
$ node bin/sync-db.js
```

node-api/bin/www.js

```js
const app = require('../index');
const syncDb = require('./sync-db');

syncDb().then( () => {
  console.log('Sync database!');
  app.listen(3000, () => {
    console.log('Server is running on 3000 port');
  })
})
```

### 데이터베이스와 index 컨트롤러 연동 1

API 로직인 user.ctrl.js 에서 모델을 연동하여 디비를 연결

describe, it 에 only() 함수로 해당 테스크 테이스만 실행할 수 있음

node-api/api/user/user.spec.js

```js
...
const app = require('../../index');
const models = require('../../models);

describe('GET /users는', () => {
  describe('성공시', () => {
    before(( ) => {
      return models.sequelize.sync({force: true}); // create table
    });
    it.only('유저 객체를 담은 배열로 응답한다', (done) => { 
      ...
    })
  })
})
```

node-api/api/user/user.ctrl.js

```js
// node-api/api/user/user.ctrl.js
const models = require('../../models');

/* 삭제
var users = [
  ...
];
*/

const index = function(req, res) {
  req.query.limite = req.query.limit(2);
  const limit = parseInt(req.query.limit, 10);
  if(Number.isNaN(limit)) {
    return res.status(400).end();
  }

  models.User.findAll({})
    .then(users => {
      res.json(users);
    })
}
```

### 데이터베이스와 index 컨트롤러 연동 2

데이터베이스 초기화 (샘플 데이터 등록)

only 이동

node-api/api/user/user.spec.js

```js
...
const app = require('../../index');
const models = require('../../models');

describe.only('GET /users는', () => {
  describe('성공시', () => {
    const users = [
      {name: 'alice'},
      {name: ' bek'},
      {name: 'chris'},
    ]
    before(( ) => {
      return models.sequelize.sync({force: true}); // create table
    });
    before((done) => {
      return models.User.bulkCreate(users); // insert data
    });
    it('유저 객체를 담은 배열로 응답한다', (done) => { // only 로 해당 테스크 테이스만 실행할 수 있음
      ...
    })
  })
})
```

node-api/api/user/user.ctrl.js

```js
// node-api/api/user/user.ctrl.js
const models = require('../../models');

const index = function(req, res) {
  req.query.limite = req.query.limit(2);
  const limit = parseInt(req.query.limit, 10);
  if(Number.isNaN(limit)) {
    return res.status(400).end();
  }

  models.User.findAll({ 
    limit: limit
  })
    .then(users => {
      res.json(users);
    })
}
```

테스트 결과에 쿼리문 로그를 숨김

node-api/models.js

```js
...
const sequelize = new Seqielize({
  dialect: 'sqlite',
  storage: './db.sqlite',
  logging: false, //console.log
})
...
```

### 데이터베이스와 show 컨트롤러 연동

only 이동

node-api/api/user/user.spec.js

```js
...
describe.only('GET /users/:id는', () => {
  ...
})
...
```

node-api/api/user/user.ctrl.js

```js
// node-api/api/user/user.ctrl.js
const models = require('../../models');

const index = function(req, res) {
  ...
}

const show = function(req, res) {
  const id = parseInt(req.params.id, 10);
  if(Number.isNaN(id)) return res.status(400).end();

  models.User.findOne({
    where: {
      id: id
    }
  }).then(user => {
    if(!user) return res.status(404).end();
    res.json(user);
  })
}
...
```

### 데이터베이스와 destroy 컨트롤러 연동

only 이동

node-api/api/user/user.spec.js

```js
...
describe.delete('DELETE /users/:id는', () => {
  ...
})
...
```

node-api/api/user/user.ctrl.js

```js
// node-api/api/user/user.ctrl.js
const models = require('../../models');

const index = function(req, res) {
  ...
}

const show = function(req, res) {
  ...
}

const destroy = (req, res) => {
  ...
  if(Number.isNaN(id)) return res.status(400).end();
  model.User.destory({
    where:{
      id: id
    }
  }).then(user => {
    res.status(204).end();
  })
}
...
```

### 데이터베이스와 create 컨트롤러 연동

only 이동

before 복사

node-api/api/user/user.spec.js

```js
// node-api/api/user/user.spec.js
...
describe.only('POST /users', () => {
  const users = [
    {name: 'alice'},
    {name: ' bek'},
    {name: 'chris'},
  ]
  before(( ) => {
    return models.sequelize.sync({force: true}); // create table
  });
  before((done) => {
    return models.User.bulkCreate(users); // insert data
  });
  ...
})
...
```

user.name 에 unique 설정

node-api/models.js

```js
// node-api/models.js
...
const User = sequelize.define('User', {
  name: {
    type: Sequelize.STRING,
    unique: true
  }
})
...
```

node-api/api/user/user.ctrl.js

```js
// node-api/api/user/user.ctrl.js
const models = require('../../models');

const index = function(req, res) {
  ...
}

const show = function(req, res) {
  ...
}

const destroy = (req, res) => {
  ...
}

const create = (req, res) => {
  const name = req.body.name;
  if(!name) return res.status(400).end();
  models.User.create({name})
    .then(user => {
      res.status(201).json(user);
    })
    .catch(err => {
      if(err.name === 'SequelizeUniqueConstraintError') {
        return res.status(409).end();
      }
      res.status(500).end();
    })

}
...
```

### 데이터베이스와 update 컨트롤러 연동

마지막 테스트케이스이므로 only 삭제

before 복사

node-api/api/user/user.spec.js

```js
// node-api/api/user/user.spec.js
...
describe('PUT /users/:id', () => {
  const users = [
    {name: 'alice'},
    {name: ' bek'},
    {name: 'chris'},
  ]
  before(( ) => {
    return models.sequelize.sync({force: true}); // create table
  });
  before((done) => {
    return models.User.bulkCreate(users); // insert data
  });
  ...
});
...
```

node-api/api/user/user.ctrl.js

```js
// node-api/api/user/user.ctrl.js
const models = require('../../models');

const index = function(req, res) {
  ...
}

const show = function(req, res) {
  ...
}

const destroy = (req, res) => {
  ...
}

const create = (req, res) => {
  ...
}

const update = (req, res) => {
  const id = parseInt(req.params.id, 10);
  if(Number.isNaN(id)) return res.status(400).end();

  const name = req.body.name;
  if(!name) return res.status(400).end();

  model.User.findIne({
    where: {
      id:id
    }
  })
    .then(user => {
      if(!user) return res.status(404).end();
      user.name = name;
      user.save()
        .then(user => {
          res.json(user);
        })
        .catch(err => {
          if(err.name === 'SequelizeUniqueConstraintError') {
            return res.status(409).end();
          }
          res.status(500).end();
        });
    });
}
...
```

## 끗

### 마무리

실제 데이터 확인

```bash
# 기존 데이터 삭제
$ rm db.sqlite
```

postman 을 통해 테스트 진행

데이터베이스가 test 상에서만 강제로 초기화되도록 설정

node-api/sync-db.js

```js
// node-api/sync-db.js
const models = require('../models');

module.exports = () => {
  const options = {
    force: process.env.NODE_ENV === 'test' ? true : false
  };
  return models.sequelize.sync(options);
}
```



## 참고
- [https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api)
- [http://expressjs.com/](http://expressjs.com/)