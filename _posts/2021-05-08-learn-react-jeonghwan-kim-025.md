---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 025
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
  - Big O
---

## 사용편

### 검색 결과가 없을 경우

#### 요구사항

> 검색 결과가 검색폼 아래 위치한다. 검색 결과가 없을 경우와 있을 경우를 구분한다.

#### 구현

```javascript
// main.js
class App extends React.Component {
  constructor() {
    super();

    this.state = {
      searchKeyword: "",
      searchResult: [], 
    }
  }
  ...
  render() {
    <>
      <div className="container">
        ...
        <div className="content">
          {this.state.searchResult.length > 0 ? (
            <div>TODO: 검색 결과 목록 표시하기</div>
          ):(
            <div className="empty-box">검색 결과가 없습니다</div>
          )}
        </div >
    </>
  }
}
```

### 검색 결과가 있을 경우

검색결과를 관리하기 위해 Store.js, storage.js, helpers.js 를 복사해온다

```javascript
// Store.js
import storage from './storage.js';
class Store { // export default 삭제
  ...
}

const store = new Store(storage);
export default store;
```

```javascript
// main.js
import store from './Store.js';

class App extends React.Component {
  ...
  handleSubmit(event) {
    event.preventDefault();
    this.search(this.state.searchKeyword)
  }
  search(keyword) {
    const searchResult = store.search(searchKeyword);
    this.setState({ searchResult });
  }
  ...
  render() {
    <>
      <div className="container">
        ...
        <div className="content">
          {this.state.searchResult.length > 0 ? (
            <ul className="result">
              {this.state.searchResult.map(item => {
                return (
                  <li>
                    <img src={ item.imageUrl } alt={ item.name } />
                    <p>{ item.name }</p>
                  </li>
                )
              })}
            </ul>
          ):(
            <div className="empty-box">검색 결과가 없습니다</div>
          )}
        </div >
    </>
  }
}
```

setState는 변경된 필드만 기존 필드와 병합하는 방식으로 필드를 관리함

Store에서는 searchResult를 더이상 보관하지 않음, 그래서 search() 함수를 수정

```javascript
// Store.js
class Store {
  ...
  search(searchKeyword) {
    return this.storage.productData.filter(
      (product) => product.name.includes(keyword)
    );
  }
  ...
}
```

데이터 흐름

1. 처음에는 searchResult 가 빈배열 > 검색결과가 없다고 보여짐
1. 검색어를 입력하고 엔터를 치면 handleSubmit() 함수 호출
1. search() 함수 호출
1. search() 함수는 입력한 검색어로 Store에서 search() 함수를 호출
1. Store의 search() 함수는 searchResult를 반환
1. 반환된 searchResult는 setState로 searchResult 상태를 갱신
1. render() 함수 호출
1. 검색 결과 노출

### 리스트와 키

key는 엘리먼트에 안정적인 고유성을 부여하기 위해 배열 내부의 엘리먼트에 지정해야 함

한마디로 searchResult의 li에 key 속성이 있어야 한다는 뜻

리액트의 가상돔과 이전 가상돔과

1. 앨리먼트 타입이 다를 경우
2. Key 값이 다를 경우

위의 두가지 가정하에 재조정 알고리즘을 사용하여 빅오계산법에 의해 O(n^3)만큼의 계산복잡도를 O(n)로 줄일 수 있다고 함

li 태그에 key를 등록

```javascript
// main.js
import store from './Store.js';

class App extends React.Component {
  ...
  render() {
    <>
      <div className="container">
        ...
        <div className="content">
          {this.state.searchResult.length > 0 ? (
            <ul className="result">
              {this.state.searchResult.map(item => {
                return (
                  <li key={item.id}>
                    <img src={ item.imageUrl } alt={ item.name } />
                    <p>{ item.name }</p>
                  </li>
                )
              })}
            </ul>
          ):(
            <div className="empty-box">검색 결과가 없습니다</div>
          )}
        </div >
    </>
  }
}
```

map의 index를 key로 사용하는 경우가 있는데, 없는거보다는 낫지만 최후의 수단으로 사용

검색을 하기도 전에 검색 결과가 없다고 나오는 문제

```javascript
class App extends React.Component {
  constructor() {
    this.state = {
      ...
      submitted: false,
    }
  }
  ...
  search(searchKeyword) {
    ...
    this.setState({
      searchResult = store.search(searchKeyword);
      submitted = true;
    });
  }
  ...
  render() {
    <>
      ...
      <div className="container">
        ...
        <div className="content">
          { this.state.submitted && (
            this.state.searchResult.length > 0 ? (
              ...
            )
          )}
        </div>
      </div>
    </>
  }
}
```

render() 함수가 가독성이 떨어졌음, 리펙토링 필요

```javascript
class App extends React.Component {
  ...
  render() {
    const searchForm = (
      <form
        onSubmit={ ( event ) => this.handleSubmit(event) }
        onReset={ () => this.handleReset() }
      >
        <input
          type="text"
          placeholder="검색어를 입력하세요"
          autoFocus
          value={ this.state.searchKeyword }
          onChange={ (event) => this.handleChangeInput(event) }
        />
        { this.state.searchKeyword.length > 0 && (
          <button type="reset" className="btn-reset"></button>
        )}
      </form>
    );

    const searchResult = (
      this.state.searchResult.length > 0 ? (
        <ul className="result">
          { this.state.searchResult.map((item, index) => {
            return (
              <li key={item.id}>
                <img src={item.imageUrl} alt={item.name}} />
                <p>{item.name}</p>
              </li>
            );
          })}
        </ul>
      ) : (
        <div className="empty-box">검색 결과가 없습니다</div>
      )
    );
    <>

    </>
    result (
      <>
        <header>
          <h2 className="container">검색</h2>
        </header>
        <div className="container">
          { searchForm }
          <div className="content">{ this.state.submitted && searchResult }</div>
        </div>
      </>
    );
  }
}
```

### 검색결과 초기화

#### 요구사항

> x 버튼을 클릭하면 검색폼이 초기화 되고, 검색 결과가 사라진다

#### 구현

```javascript
...
class App extends React.Component {
  ...
  handleReset() {
    this.setState({
      searchKeyword: "",
      submitted: false,
    });
  }
  handleChangeInput(event) {
    const searchKeyword = event.target.value;
    if(serachKeyword.length <= 0 && this.state.submitted) {
      return this.handleReset();
    }
    this.setState({ searchKeyword });
  }
  ...
}
```

### 중간정리

검색결과를 리스트로 렌더링하였음

리액트는 가상돔의 전/후 버전을 비교해서 최소한의 돔 조작을 하면서 성능을 높임

계산 복잡도를 줄이기 위해 리스트일 경우 key 속성을 사용하도록 함

모든 UI가 state에 의존하기 때문에 잘 설계된 state만 관리하면 UI를 예측하기 쉽게 제어할 수 있음

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html](https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html)
- [https://ko.reactjs.org/docs/lists-and-keys.html](https://ko.reactjs.org/docs/lists-and-keys.html)