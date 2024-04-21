---
layout: single
title:  만들면서 학습하는 리액트(react):소개편 - 데이터를 화면에 출력하는 방법 - 014
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 소개편 

### 데이터를 화면에 출력하는 방법

리액트/뷰의 특징은 화면에 출력되는 유저 인터페이스를 `상태`로 관리할 수 있음

준비편에서
- 뷰는 템플릿 객체로부터 HTML 마크업 문자열을 가져와 엘리먼트의 innerHTML 속성에 저장
- 모델(store)의 데이터가 변경되면 컨트롤러가 뷰의 render()를 호출해줌

이것을 다시 요약해보면

```javascript
const title = document.createElement("h1"); // 1
document.body.appendChild(title); // 2
let data = "Hello World"; // 3
title.textContent = data; // 4
let data = "안녕하세요."; // 5
title.textContent = data; // 6
...
```

1. 문자열을 출력할 엘리먼트를 생성한다.
1. 기존 문서에 추가한다.
1. 데이터를 준비한다.
1. 엘리먼트에 데이터를 할당한다.
1. 데이터를 준비한다.
1. 엘리먼트에 데이터를 할당한다.

데이터가 변경되면 엘리먼트에 데이터를 반영해줘야 함

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html](https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html)
