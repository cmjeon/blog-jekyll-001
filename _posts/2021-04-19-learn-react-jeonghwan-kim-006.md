---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 006
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 탭2

#### 요구사항

> 기본으로 추천 검색어 탭을 선택한다

#### 구현

- TabView.js 수정

```javascript
import { qsAll } from "../helper.js";
...
export const TabType = {
  KEYWORD: "KEYWORD",
  HISTORY: "HISTORY",
}
...
export default class TabView extends View {
  ...
  show(selectedTab) {
    this.element.innerHTML = this.template.getTabList();
    qsAll("li", this.element).forEach(li => {
      li.className = li.dataset.tab === selectedTab ? "active" : "";
    });

    super.show();
  }
}
```

- Store.js 수정

```javascript
import { TabType } from './views/TabView.js';
...
export default class Store {
  constructor(storage) {
    ...
    this.searchResult = [];
    this.selectedTab = TabType.KEYWORD;
  }
}
```

- Controller.js 수정

```javascript
...
export default class Controller {
  ...
  render() {
    ...
    }

    this.tabView.show(this.store.selectedTab);
    ...
  }
}
```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0)