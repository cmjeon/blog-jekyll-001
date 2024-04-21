---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 022
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 사용편

### 조건부 렌더링

#### 요구사항

> 검색어를 입력하면 x 버튼이 보이고 검색어를 삭제하면 x 버튼을 숨긴다

#### 조건부 렌더링 방식

엘리먼트 변수를 사용하는 방식

    ```javascript
    // main.js
    class App extends React.Component {
      ...
      render() {
        let resetButton = null;

        if(this.state.searchKeyword.length > 0) {
          resetButton = <button type="reset" className="btn-reset"></button>;
        }

        return (
          <>
          <div className="container">
            <form>
              <input
                ...
              />
              { resetButton }
            </form>
          </div>
          </>
        );
      }
    }
    ```

삼항 연산자를 사용하는 방식

    ```javascript
    // main.js
    class App extends React.Component {
      ...
      render() {
        return (
          <>
          <div className="container">
            <form>
              <input
                ...
              />
              {this.state.searchKeyword.length > 0 ? (
                <button type="reset" className="btn-reset"></button>
              ) : null}
            </form>
          </div>
          </>
        );
      }
    }
    ```

&& 연산자를 사용하는 방식

    ```javascript
    // main.js
    class App extends React.Component {
      ...
      render() {
        return (
          <>
          <div className="container">
            <form>
              <input
                ...
              />
              {this.state.searchKeyword.length > 0 && (
                <button type="reset" className="btn-reset"></button>
              )}
            </form>
          </div>
          </>
        );
      }
    }
    ```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html](https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html)
