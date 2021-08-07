---
layout: single
title:  heroku에 expressjs api 서버 만들기 - 001
categories: 
  - make expressjs api server in heroku
tags: 
  - javascript
  - expressjs
  - backend
  - heroku
---

## heroku 서비스 시작하기

### 초간단 expressjs 서비스 만들기

서비스 요청 시 'Hello'를 응답하는 간단한 expressjs 서비스를 생성함

서비스를 생성하기 전에 nvm, nodejs 그리고 npm 이 설치되어야 함

nvm, nodejs, npm 설치하기 :  [https://cmjeon.github.io/how%20to%20install/install-nvm,-node,-npm-001/](https://cmjeon.github.io/how%20to%20install/install-nvm,-node,-npm-001/)

expressjs 서비스용 폴더 생성

```bash
$ mkdir simple-expressjs
$ cd simple-expressjs
```

nodejs 용 Web fremework 인 expressjs 를 설치

```bash
$ npm i express --save
```

index.js 파일 생성

```bash
touch index.js
```

index.js 에 내용 입력

```js
// simple-expressjs/index.js
const express = require('express');
const app = express();

app.get("/", (req, res) => {
  res.send('Hello');
});

app.listen(3000, () => {
  console.log('Server is running');
});
```

로컬에서 실행해 보기

```bash
$ node index.js
Server is running
```

브라우저에서 `localhost:3000` 을 입력하면 'Hello' 를 확인할 수 있음

초간단 expressjs 서버가 완성되었음

### heroku 에 업로드 하기

heroku 사이트 : [https://id.heroku.com/login](https://id.heroku.com/login)







## 참고

- [https://expressjs.com](https://expressjs.com)