---
layout: single
title:  javascript-001 - 콘솔에 출력, script async와 defer의 차이점
categories: 
  - learning javascript by ellie
tags: 
  - javascript
---

## chocolatey 설치

```bash
> @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

# 설치확인
> choco -version
0.10.15
```

## node.js 설치

```bash
// node.js 설치
$ choco install nodejs-lts

// node.js 설치 확인
$ node -v
```

- 웹개발에 맞게 javascript를 학습

## javascript 실행

- index.html

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
      <script src="main.js"></script>
    </head>
    <body>
    </body>
  </html>
  ```

- main.js

  ```javascript
  console.log('Hello World!');
  ```

- chorme 브라우저에서 index.html 을 열고 개발자 도구 실행(F12) > 'Hello World!' 출력확인

## dev tools

### console 

- 간단한 javascript 사용 가능

### source

- 디버깅 가능

### network

- 웹페이지의 네트워크 확인

## 자바스크립트의 공식사이트는?

- es6 사이트가 있지만 너무 어려움
- developer.mozilla.org 를 추천

## async vs defer

- javascript의 위치에 따라 브라우저가 웹페이지를 해석하는 순서

### 첫번째 방법

- 소스

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
      **<script src="main.js"></script>**
    </head>
    <body>
    </body>
  </html>
  ```

- 실행순서

  ```
  parsing HTML > fetching js > excuting js > parsing HTML       
  ```

- 특징
  - 페이지를 해석하다가 자바스크립트를 다운받아 실행하고 다시 페이지를 해석
  - 페이지가 완성되는 시점이 느리다

### 두번째 방법

- 소스

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      **<script src="main.js"></script>**
    </body>
  </html>
  ```

- 실행순서

  ```
  parsing HTML > fetching js > excuting js       
  ```

- 특징
  - 페이지를 다 해석하여 완성하고 자바스크립트를 다운받아 실행
  - 페이지가 표현된 직후에는 자바스크립트를 이용한 액션이 불가능
  
### 세번째 방법

- 소스

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      **<script async src="main.js"></script>**
    </body>
  </html>
  ```

- 실행순서

  ```
  parsing HTML >   blocked   > parsing HTML
   fetching js > excuting js  
  ```

- 특징
  - 페이지를 해석하면서 자바스크립트를 다운로드 하므로 시간절약
  - 자바스크립트가 실행될 때 페이지가 온전히 해석되지 않았다면 오류 발생
  - 시간적인 장점도 그다지...

### 네번째 방법

- 소스

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      **<script defer src="main.js"></script>**
    </body>
  </html>
  ```

- 실행순서

  ```
      parsing HTML        > excuting js
  fetching js
  ```

- 특징
  - 페이지를 해석하면서 자바스크립트를 다운받고, 페이지 해석이 종료되면 자바스크립트를 실행
  - 가장 좋은 방법

## async vs defer

### async 

- 소스

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      **<script async src="a.js"></script>**
      **<script async src="b.js"></script>**
      **<script async src="c.js"></script>**
    </body>
  </html>
  ```

- 실행순서

  ```
  parsing HTML  > blocked  > blocked     > blocked          > parsing HTML
  fetching a.js            > executing a.js
  fetching b.js > executing b.js
  fetching c.js                          > executing ca.js
  ```

- 특징
  - javascript 가 실행순서를 지키지 않고 실행됨
  - javascript 가 서로 의존적이라면 문제 발생


### async 

- 소스

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      **<script defer src="a.js"></script>**
      **<script defer src="b.js"></script>**
      **<script defer src="c.js"></script>**
    </body>
  </html>
  ```

- 실행순서

  ```
  parsing HTML          > executing a.js > executing b.js > executing c.js
  fetching a.js     >
  fetching b.js >
  fetching c.js         >
  ```

- 특징
  - javascript 를 비동기적으로 다운된 다음 순서대로 실행됨


## Use strict

- js 상단에 `'use strict';` 라고 선언
- Whole-script strict mode syntax
- ECMA 5 에서 정의되었음
- 예컨데 선언되지 않은 변수의 사용 등이 불가능해 짐
- javascript 성능 개선에도 도움

## 참고
- [https://www.youtube.com/watch?v=tJieVCgGzhs](https://www.youtube.com/watch?v=tJieVCgGzhs)