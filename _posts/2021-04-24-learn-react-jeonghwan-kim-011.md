---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 011
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 최근 검색어 1

#### 요구사항

> 최근 검색어 이름, 검색일자, 삭제 버튼이 목록 형태로 탭 아래 위치한다 (추천 검색어와 비슷)   
> 목록에서 검색어를 클릭하면 선택된 검색어로 검색 결과 화면으로 이동한다 (추천 검색어와 같음)

#### 구현

- index.html

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
        ...
        <div id="keyword-list-view"></div>
        <div id="history-list-view"></div>
      </div>
    </div>
    ...
  </body>
</html>
...
```

- KeywordListView.js 수정

```javascript
...
export default class KeywordListView extends View {
  constructor(element = qs("#keyword-list-view"), template = new Template()) {
    console.log(tag, "constructor");
    super(element);
    this.template = template;
    this.bindEvents();
  }
  ...

}
```

- historyListView.js 생성

```javascript
import KeywordListView from "./KeywordListView.js";
import { qs, formatRelativeDate } from '../helpers.js';

const tag = "[HistoryListView]";

export default class HistoryListView extends KeywordListView {
  constructor() {
    super(qs("#history-list-view"), new Temeplate());
  }
}

class Template {
  getEmptyMessage() {
    return `<div class="empty-box">검색 이력이 없습니다</div>`
  }

  getList(data=[]) {
    return `
      <ul class="list">
        ${data.map(this._getItem).join("")}
      </ul>
    `
  }

  _getItem({ id, keyword, date }) {
    return `
      <li data-keyword="${keyword}">
        ${keyword}
        <span class="date">${formatRelativeDate(date)}</span>
        <button class="btn-remove"></button>
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

  getHistoryList() {
    return this.storage.historyData.sort(this._sortHistory);
  }

  _sortHistory(history1, history2) {
    return history2.date > history1.date;
  }
}
```

- main.js 수정

```javascript
...
function main() {
  ...
  const views = {
    ...,
    keywordListView: new KeywordListView(),
    historyListView: new HistoryListView(),
  }
}
```

- Controller.js 수정

```javascript
...
export default class Controller {
  constructor(
    store,
    { searchFormView, searchResultView, tabView, keywordListView, historyListView }
  ){
    ...
    this.keywordListView = keywordListView;
    this.historyListView = historyListView;
    ...
  }
  ...
  subscribeViewEvents() {
    ...
    this.keywordListView.on("@click", (event) => this.search(event.detail.value));
    this.historyListView.on("@click", (event) => this.search(event.detail.value));
  }
  render() {
    ...
    if(this.store.selectedTab === TabType.KEYWORD) {
      this.keywordListView.show(this.store.getKeywordList());
      this.historyListView.hide();
    } else if(this.store.selectedTab === TabType.HISTORY) {
      this.keywordListView.hide();
      this.historyListView.show(this.store.getHistoryList());
    } else {
      ...
    }
  }

  renderSearchResult() {
    ...
    this.keywordListView.hide();
    this.historyListView.hide();
    ...
  }
}
```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html)