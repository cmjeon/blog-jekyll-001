---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 009
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 추천 검색어 2

#### 요구사항

> 목록에서 검색어를 클릭하면 선택된 검색어의 검색 결과 화면으로 이동한다

#### 구현

- KeywordListView.js 수정

```javascript
...
export default class KeywordListView extends View {
  constructor() {
    ...
    this.template = new Template();
    this.bindEvents();
  }
  ...
  bindEvents() {
    delegate()(this.element, "click", "li", event => this.handleClick(event));
  }

  handleClick(event) {
    console.log(tag, "handleClick", event.target.dataset.keyword);
    const value = event.target.dataset.keyword;
    this.emit("@click", { value });
  }
  ...
}
```

- Controller.js 수정

```javascript
...
export default clas Controller {
  ...
  subscribeViewEvents() {
    ...
    this.tabView.on("@change", (event) => this.changeTab(event.detail.value));
    this.keywordListView.on("@click", (event) => this.search(event.detail.value));
  }
  ...
}
```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0)