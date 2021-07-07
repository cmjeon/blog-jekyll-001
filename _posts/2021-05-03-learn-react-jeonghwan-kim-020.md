---
layout: single
title:  만들면서 학습하는 리액트(react):소개편 - 020
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 소개편 

### 시작점

브랜치 전환

```bash
$ git checkout intro/jsx
```

실무에서는 CRA(create-react-app) 같은 툴로 개발환경을 구성함

index.html 수정

```html
<html>
  <head>
    ...
    <title>Lecture React</title>
    <link rel="stylesheet" href="./style.css" /> <!-- style.css 복사 -->
  </head>
  ...
  <body>
    ...
    <script type="text/babel" src="js/main.js" data-type="module" data-presets="react"></script>
  </body>
</html>
```

js/main.js 생성

```javascript
// 2-react/js/main.js
const element = <h1>Hello world</h1>;
ReactDOM.render(element, document.querySelector("#app"));
```

### 헤더 구현

브랜치 이동

```bash
$ git checkout intro/scaffolding
```

main.js 수정

```javascript
const element = (
  <header>
    <h2 className="container">검색</h2>
  </header>
);
```

JSX 는 모양은 HTML이지만 실제로는 자바스크립트라서 사용방법이 HTML과 다름

속성 이름이 HTML의 경우 소문자만 사용하지만 JSX는 카멜 케이스를 사용(onclick -> onClick)

class -> className

### 최종 정리

생략

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html](https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html)