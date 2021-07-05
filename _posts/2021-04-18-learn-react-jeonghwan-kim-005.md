---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 005
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 탭1

#### 요구사항

> 추천 검색어, 최근 검색어 탭이 검색폼 아래 위치한다

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
        <div id="search-result-view"></div>
      </div>
    </div>
    ...
  </body>
</html>
```

- TabView.js 생성

```javascript
import { qs } from "../helper.js";
import View from "./View.js";

const TabType = {
  KEYWORD: 'KEYWORD',
  HISTORY: 'HISTORY',
}

const TabLabel = {
  [TabType.KEYWORD] : '추천 검색어',
  [TabType.HISTORY] : '최근 검색어',
}

export default class TabView extends View {
  constructor() {
    super(qs("#tab-view"));
    this.template = new Template();
  }

  show() {
    this.element.innerHTML = this.template.getTabList();
    super.show();
  }
}

class Template {
  getTabList() {
    return `
      <ul class="tabs">
        ${Object.values(TabTypes)
          .map(tabType => ({ tabType, tabLabel: TabLabel[tabType]}))
          .map(this._getTab)
          .join("")
        }
      </ul>
    `;
  }

  _getTab({ tabType, tabLabel }) {
    return `
      <li data-tab="${tabType}">
        ${tabLabel}
      </li>
  }
}
```

- main.js 수정

```javascript
...
import SearchResultView from "./views/SearchResultView.js";
import TabView from "./views/TabView.js";
...
function main() {
  ...
  const views = {
    ...,
    searchResultView: new SearchResultView(),
    tabVew: new TabView()
  }
  ...
}
```

- Controller.js 수정

```javascript
...
export default class Controller {
  constructor(store, { searchFormView, searchResultView, tabView }) {
    ...
    this.searchResultView = searchResultView;
    this.tabView = tabView;
    
    subscribeViewEvents();
    this.render();
  }
  ...
  render() {
    if(this.store.serachKeyword.length > 0) {
      return this.renderSearchResult();
    }

    this.tabView.show();
    
    this.searchResultView.hide();
  }

  renderSearchResult() {
    this.tabView.hide();
    this.searchResultView.show(this.store.searchResult);
  }
}
```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0)