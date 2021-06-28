---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 003
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 검색폼1

#### 요구사항

> 검색 상품명을 입력할 수 있는 폼이 위치한다.

#### 구현

- index.html 수정

    ```html
    ...
    <html>
      ...
      <body>
        ...
        <div class="container">
          <form id="search-form-view">
            <input type="text" placeholder="검색어를 입력하세요" autofocus />
            <button type="reset" class="btn-reset"></button>
          </form>
        </div>
        ...
      </body>
      ...
    </html>
    ```

- SearchFormView.js 생성

    ```javascript
    // views/SearchFormView.js
    import View from "./View.js";

    export default class SearchFormView extends View {
      constructor() {
        super(qs("#search-form-view")); // views/View.js 의 element
        this.resetElement = qs("[type=reset]", this.element); 
        this.showResetButton();
      }

      showResetButton(visible = true) {
        this.resetElement.style.display = visible?"block":"none";
      }
    }
    ```

- main.js 수정

    ```javascript
    function main() {
      ...
      const views = {
        searchFormView: new SearchFormView()
      }
      ...
    }
    ```

- Controller.js 수정

    ```javascript
    export default class Controller {
      constructor(store, { searchFormView }) {
        ...
        this.searchForm = searchFormView;
      }
    }
    ```

#### 전체 순서 복기

1. 진입점에서 main.js 에서 main 함수 호출
1. store, view 객체 생성해서 Controller 객체 생성에 인자로 사용
1. view 객체를 생성할 때 쓰는 SearchFormView.js의 constructor 함수 실행
1. super(qs...)가 실행되며 결과를 this.element 에 저장
1. qs("[type=reset]"가 실행되며 결과를 this.resetElement 에 저장
1. showResetButton(false) 실행
1. showResetButton(false)가 실행되며, this.resetElement.style.display를 'none'으로 처리

### [순수JS 1] 검색폼2

#### 요구사항

> 검색어를 입력하면 X 버튼이 보이고 검색어를 삭제하면 X 버튼을 숨긴다

#### 구현

- 브랜치 이동

    ```bash
    $ git checkout -f ready/search-form-1
    ```

- SearchFormView 수정

    ```javascript
    export default class SearchFormView extends View {
      constructor() {
        ...
        super(qs("#search-form-view"));
        
        this.inputElement = qs("[type=text]"), this.element);
        ...
        this.showResetButton(false);
        this.bindEvents();
      }
      ...
      bindEvents() {
        on(this.inputElement, "keyup", () => this.handleKeyUp());
      }

      handleKeyUp() {
        console.log(tag, "handleKeyUp", this.inputElement.value);
        const { value } = this.inputElement;
        this.showResetButton(value.length > 0);
      }
    }
    ```

### [순수JS 1] 검색폼3

#### 요구사항

> 엔터를 입력하면 검색 결과가 보인다

#### 구현

- 브랜치 이동

    ```bash
    $ bit checkout -f ready/search-form-2
    ```

- SearchFormView.js 수정

    ```javascript
    export default class SearchFormView extends View {
      ...
      bindEvents() {
        on(this.element, "submit", event => this.handleSubmit(event));
      }
      ...
      handleSubmit(event) {
        event.preventDefault(); // 브라우저의 submit 기본동작 취소
        console.log(tag, "handleSubmit");
        const { value } = this.inputElement;
        this.emit("@submit", { value });
      }
    }
    ```
    
- Controller.js 수정

    ```javascript
    export default class Controller {
      constructor(store, { searchFormView }) {
        ...
        this.subscribeViewEvents();
      }

      subscribeViewEvents() {
        this.searchFormView.on('@submit', (event) => this.search(event.detail.value)); // event.detail 에 value 가 있는걸 어떻게 알았지?
      }
      search(keyword) {
        console.log(tag, keyword);
      }
    }
    ```

### [순수JS 1] 검색폼4

#### 요구사항

> X 버튼을 클릭하거나 검색어를 삭제하면 검색 결과를 삭제한다

#### 구현

- SearchFormView.js 수정

    ```javascript
    ...
    export default class SearchFormView extends View {
      ...
      bindEvents() {
        ...
        on(this.resetElement, "click", () => this.handleReset());
      }

      handleKeyup() {
        ...
        if(value.length <= 0) {
          this.handleReset();
        }
      }
      ...
      handleReset() {
        this.emit('@reset');
      }
    }
    ```

- Controller.js 수정

    ```javascript
    ...
    export default class Controller {
      ...
      subscribeViewEvents() {
        this.searchFormView.on('@submit', (event) =>
          ...
        ).on('@reset', () =>
          this.reset()
        )
      }
      ...
      reset() {
        console.log(tag, 'reset');
      }
    }
    ```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0)