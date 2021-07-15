---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 029
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
  - Big O
---

## 사용편

### 최근 검색어 1

#### 요구사항

> 최근 검색어 이름, 검색일자, 삭제 버튼이 목록 형태로 탭 아래 위치한다 (추천 검색어와 비슷)
> 목록에서 검색어를 클릭하면 선택된 검색어로 검색 결과 화면으로 이동한다 (추천 검색어와 같음)

#### 구현

```javascript
// store.js
...
class Store {
  ...
  getHistoryList() {
    return this.storage.historyData.sort(this._sortHistory);
  }
  _sortHistory(history1, history2) {
    return history2.date > history1.datel;
  }
  ...
}
```

```javascript
// main.js
import { formatRelativeDate } from "./js/helpers.js"
...
class App extends React.Component {
  constructor() {
    ...
    this.state = {
      ...
      keywordList: [],
      historyList: [],
    }
  }
  ...
  componentDidMount() {
    const keywordList = store.getKeywordList();
    const historyList = store.getHistoryList();

    this.setState({
      keywordList,
      historyList
    })
  }
  ...
  render() {
    ...
    const historyList = (
      <ul className="list">
        { this.state.historyList.map((item, index) => {
          return (
            <li
              key={ item.id }
              onClick={ () => this.search(item.keyword) }
            >
              <span>{ item.keyword }</span>
              <span className="date">{ formatRelativeDate(item.date) }</span> 
              <button className="btn-remove"></button>
            </li>
          );
        })}
      </ul>
    )
    ...
    const tabs = (
      <>
        ...
        </ul >
        { this.state.selectedTab === TabType.KEYWORD && keywordList }
        { this.state.selectedTab === TabType.HISTORY && historyList }
      </>
    )
  }
}
...
```

### 최근 검색어 2

#### 요구사항

> 목록에서 x 버튼을 클릭하면 선택된 검색어가 목록에서 삭제된다

#### 구현

```javascript
// store.js
...
class Store {
  ...
  removeHistory(keyword) {
    this.storage.historyData = this.storage.historyData.filter(
      (history) => history.keyword !== keyword
    );
  }
  ...
}
```

```javascript
// main.js
...
class App extends React.Component {
  ...
  handleClickRemoveHistory(event, keyword) {
    event.stopPropagation();
    store.removeHistory(keyword);
    const historyList = store.getHistoryList();
    this.setState({ historyList });
  }
  ...
  render() {
    ...
    const historyList = (
      <ul className="list">
        ...{
          return (
            <li
              key={ item.id }
              onClick={ () => this.search(item.keyword) }
            >
              ...
              <button className="btn-remove" onClick={ (event) => this.handleClickRemoveHistory(event, item.keyword) }></button>
            </li>
          );
        })}
      </ul>
    )
    ...
  }
}
...
```

### 최근 검색어 3

#### 요구사항

> 검색시마다 최근 검색어 목록에 추가된다

#### 구현

```javascript
// store.js
import storage from './storage.js';
import { createNextId } from "./helper.js" 
...
class Store {
  ...
  search(searchKeyword) {
    this.addHistory(searchKeyword);
    return this.storage.productData.filter(
      (product) => product.name.includes(searchKeyword)
    );
  }

  addHistory(keyword = "") {
    keyword = keyword.trim();
    if(!keyword) {
      return;
    }
    const hasHistory = this.storage.historyData.some((history) => history.keyword === keyword );
    if(hasHistory) this.removeHistory(keyword);

    const id = createNextId(this.storage.historyData);
    const date = new Date();
    this.storage.historyData.push({ id, keyword, date });
    this.storage.historyData = this.storage.historyData.sort(this._sortHistory);
  }
  ...
}
...
```

```javascript
// main.js
...
class App extends React.Component {
  ...
  search(searckKeyword) {
    const searchResult = store.search(searchKeyword);
    const historyList = store.getHistoryList();
    this.setState({
      ...
      submitted: true,
      historyList,
    })
  }
}
```

### 중간정리

탭과 추천 검색어, 최근 검색어를 구현하였음

컴포넌트 생명주기, componentDidMount()

추천 검색어/최근 검색어 순서

1. 스토어에서 데이터 가져옴
1. 이를 컴포넌트 setState를 이용해 state에 저장
1. render()

순수 자바스크립트에서는 모델이 스토리지 데이터 + 어플리케이션 상태(selectedTab, searchKeyword 등)를 관리하였음

리액트를 사용하면 모델은 스토리지 데이터만 관리하고 컴포넌트가 어플리케이션 상태를 관리함

리액트의 state를 통해 UI를 관리하고 이를 기반으로 리액트 엘리먼트를 만들어서 데이터와 UI가 연결되게 함

리액티브하다는 것은 이렇게 상태를 변경하는 것만으로도 연결된 UI에게까지 변화를 파생시킬 수 있다는 것임

### 최종정리

리액트를 이용해서 검색폼을 구현하였음

input 엘리먼트 상태를 리액트에서 관리할 수 있도록 컴포넌트 클래스를 사용함

컴포넌트 state에 의해 입력 값이 제어되는 input을 제어 컴포넌트라고 함

검색 결과와 탭 구현부터는 리스트 렌더링을 사용함

리액트는 성능을 올리기 위해 가상돔을 사용하는데 리스트 엘리먼트에 key가 있어야 함

추천/최근 검색어에서 초기화한 state의 값을 외부에서 가져올 수 있도록 함, componentDidMount()를 통해 데이터를 불러옴

순수 자바스크립트에서는 모델은 모든 데이터(어플리케이션 상태 + 스토리지 데이터)를 관리함, 뷰는 돔과 밀접하게 움직이면서 화면을 그림, 컨트롤러는 이 둘을 관리하면서 어플리케이션 상태와 이에 따라 UI를 수시로 변화시킴

리액트에서는 모델은 스토리지 데이터만 가지고 있고 어플리케이션 상태는 컴포넌트에서 관리함

컴포넌트는 어플리케이션 상태를 관리하고 상태가 변경되면 render() 함수를 호출하여 UI를 변경함, 즉 state의 변화가 UI까지 자동으로 변경시킴

어플리케이션 상태가 변할 때마다 수시로 돔을 조작했지만 리액트는 돔 앞에 가상돔을 추가 가상돔을 수정하게끔 함

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html](https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html)