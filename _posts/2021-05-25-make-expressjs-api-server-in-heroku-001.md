---
layout: single
title:  heroku 에 expressjs api 서버 만들기 - 001
categories: 
  - make expressjs api server in heroku
tags: 
  - javascript
  - expressjs
  - backend
  - heroku
---

## 초간단 expressjs 서비스 만들기

서비스 요청 시 'Hello'를 응답하는 간단한 expressjs 서비스를 생성함

서비스를 생성하기 전에 nvm, nodejs 그리고 npm 이 설치되어야 함

nvm, nodejs, npm 설치하기 :  [https://cmjeon.github.io/how%20to%20install/install-nvm,-node,-npm-001/](https://cmjeon.github.io/how%20to%20install/install-nvm,-node,-npm-001/)

expressjs 서비스용 폴더 생성

```bash
$ mkdir simple-expressjs
$ cd simple-expressjs
```

npm 초기화

```bash
npm init --yes
```

package.json 파일이 생성됨

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
const port = process.env.PORT;

app.get("/", (req, res) => {
  res.send('Hello');
});

app.listen(port, () => {
  console.log('Server is running', port);
});
```

로컬에서 실행해 보기

```bash
$ PORT=3000 node index.js
Server is running 3000
```

브라우저에서 `localhost:3000` 을 입력하면 'Hello' 를 확인할 수 있음

초간단 expressjs 서버가 완성되었음

## heroku 에 업로드할 준비하기

node, npm 버전 확인

```bash
$ node -v
v14.17.1
$ npm -v
6.14.13
```

package.json 에 내용추가

- "engines" 에 node, npm 버전 정보 추가
- "script" 에 `"start" "node index.js" 추가

```json
// simple-expressjs/package.json
{
  "name": "simple-expressjs",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  },
  "engines": {
    "node": "14.17.1",
    "npm": "6.14.13"
  }
}
```

.gitignore 파일 생성

```bash
touch .gitignore
```

.gitignore 에 내용 입력

```js
// simple-expressjs/.gitignore
node_modules
```

## heroku 회원가입

heroku 사이트 : [https://id.heroku.com/login](https://id.heroku.com/login)

## heroku CLI 설치

[https://devcenter.heroku.com/articles/heroku-cli](https://devcenter.heroku.com/articles/heroku-cli)

### macOS

```bash
$ brew tap heroku/brew && brew install heroku
```

### Windows

```bash
$ choco install heroku-cli
```

### Ubuntu 16+

```bash
$ sudo snap install --classic heroku
```

## heroku 에 푸시 준비

heroku CLI 로그인

```bash
$ heroku login
```

git 초기화

```bash
$ git init
```

heroku URL 생성

```bash
$ heroku create
Creating app... done,
```

https://git.heroku.com/xxx-xxx-xxx.git 주소 복사

리모트저장소 URL로 설정

```bash
$ git remote add heroku https://git.heroku.com/xxx-xxx-xxx.git
```

git 에 변경내용 추가

```bash
$ git add .
$ git commit -m 'first commit'
```

heroku remote 에 변경내용 push

```bash
# main은 브랜치 이름에 맞게 변경
$ git push heroku main
```

heroku open 실행하거나 브라우저에서 URL로 확인

```bash
$ heroku open
```

브라우저에서 `https://xxx-xxx-xxx.herokuapp.com/` 입력

## 참고

- [https://expressjs.com](https://expressjs.com)
- [https://medium.com/@yoobi55/express-node-js-를-이용해-서버를-만들어-heroku에-올리는-방법-3a5134fc8743](https://medium.com/@yoobi55/express-node-js-를-이용해-서버를-만들어-heroku에-올리는-방법-3a5134fc8743)
