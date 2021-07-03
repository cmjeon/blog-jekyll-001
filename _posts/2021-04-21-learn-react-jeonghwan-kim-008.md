---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 008
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 추천 검색어 1

#### 요구사항

> 번호, 추천 검색어 이름이 목록 형태로 탭 아래 위치한다

#### 구현

- index.html 수정

```html
...
<html>
  ...
  <body>
    ...
    <div>
      ...
      </form>
      <div class="content">
        <div id="tab-view"></div>
        <div id="keyword-list-view"></div>
      </div>
    </div>
    ...
  </body>
</html>
...
```

- KeywordListView.js 생성

```javascript
import { delegate, qs } from "../helpers.js";
import View from "./View.js";

const tag = "[KeywordListView]";

export default class KeywordListView extends View {
  constructor() {
    super(qs("#keyword-list-view"));
    
    this.template = new Template();
  }

  show(data=[]) {
    this.element.innerHTML = data.length > 0 ? this.template.getList(data) : this.template.getEmptyMessage();
    super.show();
  }
}

class Template {
  getEmptyMessage() {
    return `<div class="empty-box">추천 검색어가 없습니다</div>`
  }

  getList(data=[]) {
    return `
      <ul class="list">
        ${data.map(this._getItem).join("")}
      </ul>
    `
  }

  _getItem(id, keyword) {
    return `
      <li data-keyword="${keyword}">
        <span class="number">${id}</span>
        ${keyword}
      </li>
    `
  }
}
```

- Store.js 수정

```javascript
...
export default class Store {
  ...
  }

  getKeywordList() {
    return this.storage.keywordData;
  }
}
```

- main.js 수정

```javascript
function main() {
  const views = {
    ...,
    tabView: new TabView(),
    keywordListView: new KeywordListView();
  }
}
```

- Constroller.js 수정

```javascript
...
export default class Controller {
  constructor(store, { searchFormView, searchResultView, tabView, keywordListView }) {
    ...
    this.keywordListView = keywordListView;
    ...
  }
  ...
  render() {
    ...

    this.tabView.show(this.store.selectedTab);
    if(this.store.selectedTab === TabType.KEYWORD) {
      this.keywordListView.show(this.store.getKeywordList())
    } else if (this.store.selectdTab === TabType.HISTORY) {
      this.keywordListView.hide();
    } else {
      throw "사용할 수 없는 탭입니다.";
    }
    
    this.searchResutlView.hide();
  }

  renderSearchResult() {
    this.tabView.hide();
    this.keywordListView.hide();
    this.searchResultView.show(this.store.searchResult);
  }
}
```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0)