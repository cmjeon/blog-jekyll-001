---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 004
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 검색결과1

#### 요구사항

> 검색 결과가 검색폼 아래 위치한다. 검색 결과가 없을 경우와 있을 경우를 구분한다.

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
        <div id="search-result-view"></div>
      </div>
    </div>
    ...
  </body>
</html>
```

- SearchResultView.js 생성

```javascript
import { qs } from "../helper.js";
import View from "./View.js";

export default class SearchResultView extends View {
  constructor() {
    super(qs("#search-result-view"));

    this.template = new Template();
  }

  show(data = []) {
    this.element.innerHTML = data.length > 0 ? this.template.getList(data) : this.template.getEmptyMessage();
    super.show();
  }
}

class Template {
  getList(data = []) {
    return `
      <ul class="result">
        ${data.map(this._getItem).join("")}
      </ul>
    `;
  }

  getEmptyMessage() {
    return `
      <div class="empty-box">검색결과가 없습니다.</div>
    `;
  }

  _getItem({}) {
    return `
      <li>
        <img src="${imageUrl}" alt="${name}" />
        <p>${name}</p>
      </li>
    `;
  }
}
```

- Store.js 수정

```javascript
...
export default class Store {
  constructor(storage) {
    ...
    this.storage = storage;
    this.searchKeyword = "";
    this.searchResult = [];
  }

  search(keyword) {
    this.searchKeyword = keyword;
    this.searchResult = this.storage.productData.filter(
      (product) => product.name.includes(keyword)
    );
  }
}
```

- main.js 수정

```javascript
...
import SearchFormView from "./views/SearchFormView.js";
import SearchResultView from "./views/SearchResultView.js";
...
function main() {
  ...
  const views = {
    searchFormView: new SearchFormView(),
    searchResultView: new SearchResultView(),
  }
  ...
}
```

- Controller.js 수정

```javascript
export default class Controller {
  constructor(store, { searchFormView, searchResultView }) {
    ...
    this.searchFormView = searchFormView;
    this.searchResultView = searchResultView;
    ...
  }
  ...
  search(searchKeyword) {
    console.log(tag, "search", searchKeyword);
    this.store.search(searchKeyword);
    this.render();
  }
  ...
  render() {
    if(this.store.serachKeyword.length > 0) {
      this.searchResultView.show(this.store.searchResult);
      return;
    }

    this.searchResultView.hide();
  }
}
```

### [순수JS 2] 검색결과2

#### 요구사항

> X 버튼을 클릭하면 검색폼이 초기화 되고, 검색 결과가 사라진다.

#### 구현

- Controller.js 수정

```javascript
...
export default class Controller {
  ...
  reset() {
    console.log(tag, 'reset');
    this.store.searchKeyword = "";
    this.store.searchResult = [];
    this.render();
  }
  ...
}

```

### 중간정리

- 리액트 비교 학습 대상으로의 순수 자바스크립트 선택
- 화면개발을 위해 MVC패턴을 적용
- View는 눈에 보이는 UI
- Store로 구현한 Model은 데이터를 관리
- View와 Store를 이용하여 어플리케이션을 동작시키는 것이 Controller

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html)
