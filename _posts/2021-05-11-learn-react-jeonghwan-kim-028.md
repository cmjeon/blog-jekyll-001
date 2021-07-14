---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 028
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
  - Big O
---

## 사용편

### 추천 검색어 1

#### 요구사항

> 번호, 추천 검색어 이름이 목록 형태로 탭 아래 위치한다

#### 구현

```javascript
// main.js
...
class App extends React.Component {
  constructor() {
    ...
    this.state = {
      ...
      keywordList: [],
    }
  }
  ...
}
...
```

리액트 컴포넌트는 생성부터 소멸까지 일련의 생명 주기를 가짐

- 컴포넌트 상태 등 초기화 작업을 완료하면 컴포넌트 객체가 생성됨 (contructor)
- 리액트 앨리먼트를 이용해 가상돔을 만들고 이것을 실제 돔에 반영 (render)
- 돔에 반영되는 것을 `마운트된다`고 표현하는데 마운트가 완료되면 (componentDidMount) 으로 이벤트를 바인딩하거나 외부 데이터를 가져오는 등의 작업을 수행
- 컴포넌트가 사라지기 전에 즉 언마운트 직전에 (componentWillUnmount) 이벤트 핸들러를 제거하거나 리소스 정리작업을 수행
- 마지막으로 컴포넌트는 소멸됨

```javascript
// store.js
...
class Store {
  ...
  getKeywordList() {
    return this.storage.keywordData;
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
  componentDidMount() {
    const keywordList = store.getKeywordList();
    this.setState({ keywordList });
  }
  ...
  render() {
    ...
    const keywordList = (
      <ul className="list">
        { this.state.keywordList.map( (item, index) => {
          return (
            <li key={ item.id }>
              <span className="number">{ index + 1 }</span>
              <span>{ item.keyword }</span>
            </li>
          )
        })}
      </ul>
    )

    const tabs = (
      <>
        ...
        { this.state.selectedTab === TabType.KEYWORD && ( keywordList )}
        { this.state.selectedTab === TabType.HISTORY && (<>최근 검색어</> ) }
      </>
    )
    ...
  }
}

```

### 추천 검색어 2

#### 요구사항

> 목록에서 검색어를 클릭하면 선택된 검색어의 검색 결과 화면으로 이동한다

#### 구현

```javascript
// main.js
...
class App extends React.Component {
  search(searchKeyword) {
    const searchResult = store.search(searchKeyword);
    this.setState({
      searchKeyword,
      searchResult,
      submitted: true,
    })
  }
  ...
  render() {
    const searchForm = (
    
    )

    const keywordList = (
      <ul className="list">
        ...
          <li key={ id } onClick={ () => this.search(item.keyword) }>
            ...
          </li>
        ))}
      </ul>
    )
  }
}
...
```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html](https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html)