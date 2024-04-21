---
layout: single
title:  만들면서 학습하는 리액트(react):소개편 - 018
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 소개편 

### React

vanilla js로 DOM element 만들고 추가하기

```javascript
const p = document.createElement("p");
p.textContent = "hello world";
document.body.appendChild(p);
```

DOM에 엘리먼트가 있듯이 리액트 앱에도 엘리먼트가 있음

엘리먼트는 리액트 앱을 구성하는 최소 단위임

```javascript
const element = React.createElement("h1", null, "Hello world");
ReactDOM.render(element, document.querySelector('#app'));
```

리액트 앱의 엘리먼트는 자바스크립트 일반 객체

DOM의 엘리먼트는 DOM 객체

리액트 엘리먼트는 가상돔을 구성하는 요소

가상돔은 리액트 어플리케이션과 DOM 사이의 계층, 최소한의 연산으로 화면을 그려주는 역할

### ReactDOM

리액트가 만든 가상돔은 일반 객체임

리액트는 일반 객체로 구성된 가상돔을 사용함으로 렌더링하는 환경에 따라 여러 플랫폼에서 사용가능해짐

리액트를 웹 브라우저에서 동작하게 하려면 가상돔이 실제 DOM API를 호출하게 해야 함, 이 역할을 하는 것이 ReactDOM 라이브러리임

따라서 리액트 웹 개발은 react + reactDOM 을 사용한는 것

AOS, iOS 같은 네이티브 앱에서도 리액트를 사용할 수 있음, react + react-native

windows, mac 같은 데스크탑 어플리케이션도 개발할 수 있음, react + react-native-windows

```javascript
const element = React.createElement("h1", null, "Hello world"); // 1
ReactDOM.render(element, document.querySelector('#app')); // 2
```

1. element 라는 일반객체 생성
1. ReactDOM 이 element 객체를 받아서 가상돔을 생성

### Babel

Babel은 최신 자바스크립트 문법을 브라우저에서 사용가능하게 하기 위한 도구(트랜스파일러)

최신 자바스크립트로 개발하면 대부분의 브라우저에서 작동하도록 변환해줌

JSX 문법을 사용할 때에도 바벨이 필요함

바벨은 웹팩과 같은 번들러로 통합해서 사용됨

```html
<!DOCTYPE html>
<html lang="ko">
  ...
  <body>
    ...
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <script type="text/babel">
      const element = React.createElement("h1", null, "Hello world");
      ReactDOM.render(element, document.querySelector("#app"));
    </script>
  </body>
</html>
```

`<script type="text/bable">` 내부의 코드를 변환해줌

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html](https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html)
- [https://ko.reactjs.org/docs/react-without-es6.html](https://ko.reactjs.org/docs/react-without-es6.html)
- [https://jeonghwan-kim.github.io/series/2019/12/22/frontend-dev-env-babel.html](https://jeonghwan-kim.github.io/series/2019/12/22/frontend-dev-env-babel.html)
