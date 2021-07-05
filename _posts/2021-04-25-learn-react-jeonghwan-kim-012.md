---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 012
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 최근 검색어 2

#### 요구사항

> 목록에서 x 버튼을 클릭하면 선택된 검색어가 목록에서 삭제된다

#### 구현

- historyListView 수정

```javascript
import { delegate, formatRelativeDate, qs } from "../helpers.js";
...
export default class HistoryListView extends KeywordListView {
  ...
  bindEvents() {
    delegate(this.element, "click", "button.btn-remove", (event) => this.handleClickRemoveButton(event));
    super.bindEvents();
  }

  handleClickRemoveButton(event) {
    console.log(tag, "handleClickRemoveButton", event.target);
    const value = event.target.parentElement.dataset.keyword;
    this.emit("@remove", { value });
  }
}
```

- Store.js 수정

```javascript
...
export default class Store {
  ...
  }

  removeHistory(keyword) {
    this.storage.historyData = this.storage.historyData.filter(history => history.keyword !== keyword);
  }
}
```

- Controller 수정

```javascript
...
export default class Controller {
  ...
  subscribeViewEvents() {
    ...
    this.historyListView
      .on("@click", (event) => this.search(event.detail.value));
      .on("@remove", (event) => this.removeHistory(event.detail.value));
  }
  ...
  removeHistory(keyword) {
    this.store.removeHistory(keyword);
    this.render();
  }
```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html)