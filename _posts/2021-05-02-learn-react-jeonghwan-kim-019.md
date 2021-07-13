---
layout: single
title:  만들면서 학습하는 리액트(react):소개편 - 019
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 소개편 

### 템플릿 언어

```javascript
const h1 = document.createElement("h1") // 1
const header = document.createElement("header") // 2
header.appendChild(h1) // 3
```

1. 엘리먼트 h1을 생성
1. header 엘리먼트를 생성
1. appendChild 로 h1 엘리먼트를 header 엘리먼트의 자식으로 만듬

트리 형태의 웹 문서 구조상 이런 코드가 늘어나면 가독성이 떨어짐

이런 문제를 해결하기 위해 나온게 템플릿 언어, 핸들바, Pug, Angular, Vue 등

```javascript
const h1 = React.createElement("h1", null, "Hello world")
const header = React.createElement("header", null, h1)
```

리액트도 createElement() 함수를 사용하는 방식은 읽기 어렵다

따라서 리액트에서는 `JSX(JavaScript XML)`라는 자바스크립트 확장 문법을 사용함

### JSX

```html
<h1>Hello world</h1> // React.createElement('h1', null, 'Hello world')
```

JSX를 사용하면 리액트 UI 코드의 모습이 HTML과 닮음

DOM 구조와 유사하기 때문에 코드로부터 UI을 쉽게 추측할 수 있음

JSX 코드는 바벨에 의해 변환됨

그리고 리액트 엘리먼트를 반환하기 때문에 ReactDOM.render()의 인자로 전달 가능

```javascript
// 1
const element = (
  <header>
    <h1>Hello world</h1>
  </header>
)

// 2
ReactDOM.render(element, document.querySelector("#app"))
```

1. JSX는 자바스크립트 표현식으로 값으로 평가되고 리액트 엘리먼트를 반환
1. 리액트 엘리먼트를 ReactDOM.render() 함수를 사용하여 렌더링

### 중간정리

리액트가 UI를 어떻게 다루는지?

리액트 앱을 이루는 최소단위가 리액트 엘리먼트, React.createElement() 함수로 생성

리액트 엘리먼트를 가상돔으로 만들어 실제돔에 반영해 주는 것이 ReactDOM

리액트 코딩을 하려면 Babel 같은 트랜스파일러의 도움이 필요, Babel은 클래스, JSX를 사용할 수 있게 해줌

JSX는 UI코드의 가독성을 높여줌

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html](https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html)