---
layout: single
title:  만들면서 학습하는 리액트(react):소개편 - 리액티브 - 015
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 소개편 

### 리액티브

- 특정 값에 의존해 자동으로 반응하는 것을 `리액티브(reactive) 하다` 고 함
- 데이터가 변경될 때마다 엘리먼트에 자동으로 반영시키는 방법

```javascript
const state = {
  _data: "Hello world",
};

const render = () => (document.body.innerHTML = `<p>${state.data}</p>`); // 2-1, 3-3, 4-3

Object.defineProperty(state, 'data', {
  get() {
    console.log('get'); // 2-2, 3-4, 4-4
    return state._data;
  },
  set(value) {
    console.log('set'); // 3-1, 4-1
    state._data = value;
    render(); // 3-2, 4-2
  }
}); // 1

render(); // 2
state.data = "안녕"; // 3
state.data = "Hello World"; // 4

// result
// get
// set
// get
// set
// get
```

1. Object.definePropoerty : state 객체에 'data'라는 프로퍼티를 정의하고, 프로퍼티를 호출하면 get() 함수를 호출하고, 프로퍼티에 값을 할당할 때 set() 함수를 호출하게 함
1. render() 함수를 호출
    1. render() 함수는 state.data 를 호출함
    1. state.data 가 호출 되면 get() 함수가 호출됨 // get
1. state.data 에 "안녕" 을 할당함
    1. state.data 에 값을 할당하면 set() 함수가 호출됨 // set
    1. render() 함수를 호출
    1. render() 함수는 state.data 를 호출함
    1. state.data 가 호출 되면 get() 함수가 호출됨 // get
1. state.data 에 "Hello World" 을 할당함
    1. state.data 에 값을 할당하면 set() 함수가 호출됨 // set
    1. render() 함수를 호출
    1. render() 함수는 state.data 를 호출함
    1. state.data 가 호출 되면 get() 함수가 호출됨 // get

- 데이터가 변경될 때 마다 render()가 자동으로 실행되게 변경함

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html](https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html)