---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 026
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 사용편

### 탭 보여주기

#### 요구사항

> 추천 검색어, 최근 검색어 탭이 검색폼 아래 위치한다.

#### 구현

```javascript
// main.js
import store from "./js/Store.js";

const TabType = {
  KEYWORD: "KEYWORD",
  HISTORY: "HISTORY",
};

const TabLabel = {
  [TabType.KEYWORD]: "추천 검색어",
  [TabType.HISTORY]: "최근 검색어",
}

class App extends React.Component {
  ...
  render() {
    ...
    const tabs = (
      <ul className="tabs">
        { Object.values(TabType).map(tabType => {
          return (
            <li key={tabType}>{ TabLabel[tabType] }</li>
          )
        })}
      </ul>
    )

    return (
      <>
        ...
        <div className="container">
          { searchForm }
          <div className="content">
            { this.state.submitted ? searchResult : tabs }
          </div>
        </div>
      </>
    )
  }
}
...
```

### 기본 탭 설정과 탭 선택

#### 요구사항

> 기본으로 추천 검색어 탭을 선택한다
> 각 탭을 클릭하면 탭 아래 내용이 변경된다

#### 구현

```javascript
// main.js
...
class App extends React.Component {
  constructor() {
    ...
    this.state = {
      ...
      selectedTab: TabType.KEYWORD,
    }
  }

  render() {
    ...
    const tabs = (
      <>
        <ul className="tabs">
          { Object.values(TabType).map((tabType) => (
            <li
              className={ this.state.selectedTab === tabType ? 'active' : ''}
              Key={ tabType }
              onClick={ () => this.setState({ selectedTab: tabType })}
            >
              { TabLabel[TabType] }
            </li>
          ))}
        </ul>
        { this.state.selectedTab === TabType.KEYWORD && <>추천 검색어</>}
        { this.state.selectedTab === TabType.HISTORY && <>최근 검색어</>}
      </>
    )
  }
}
```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html](https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html)