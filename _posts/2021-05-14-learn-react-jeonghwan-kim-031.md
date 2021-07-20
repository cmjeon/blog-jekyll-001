---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 031
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 사용편2

### KeywordList, HistoryList

#### 요구사항

> 번호, 추천 검색어 이름이 목록 형태로 탭 아래 위치한다   
> 목록에서 검색어를 클릭하면 선택된 검색어의 검색 결과 화면으로 이동한다   

> 최근 검색어 이름, 검색일자, 삭제 버튼이 목록 현태로 탭 아래 위치한다   
> 목록에서 검색어를 클릭하면 선택된 검색어로 검색 결과 화면으로 이동한다   
> 목록에서 x 버튼을 클릭하면 선택된 검색어가 목록에서 삭제된다   
> 검색시마다 최근 검색어 목록에 추가된다

추천 검색어와 최근 검색어는 비슷한 데이터와 모양을 가지고 있음

공통의 요구사항을 코드 재사용으로 제작 해보겠음

컴포넌트에서 코드를 재사용할 수 있는 방법

- 상속
- 조합:컴포넌트 담기
- 조합:특수화

### 상속 1

src/components 에 List.js 생성

```js
// src/components/List
import React from 'react';

export default class List extends React.Component {
  constructor() {
    super();

    this.state = {
      data: []
    }
  }

  renderItem(item, index) {
    throw "renderItem()을 구현하세요"
  }

  render() {
    return (
      <ul className="list">
        {this.state.data.map((item, index) => {
          return (
            <li key={item.id} onClick={()=>this.props.onClick(item.keyword)}>
              {this.renderItem(item, index)}
            </li>
          )
        })}

    )
  }
}
```

src/components 에 KeywordList.js 생성

```js
// src/components/KeywordList.js
import React from 'react';
import List from './List.js';
import store from '../Store.js';

export default class KeywordList extends List {
  componentDidMount() {
    const data = store.getKeywordList();
    this.setState({
      data,
    })
  }

  renderItem(item, index) {
    return (
      <>
        <span className="number">{index + 1}</span>
        <span>{item.keyword}</span>
      </>
    )
  };
}
```

App.js 수정

```js
// src/App.js
...
import store from './Store.js';
import keywordList from './components/KeywordList.js';

export default class App extends React.Component {
  ...
  render() {
    ...
    return (
      <>
        ...
        <div className="container">
          ...
          <div className="content">
            {submitted ? (
              <SearchResult data={searchResult} />
            ) : (
              <>
                ...
                {selectedTab === TabType.KEYWORD && (
                  <KeywordList onClick={(keyword) => this.search(keyword)} />
                )}  
                {selectedTab === TabType.HISTORY && <>TODO: 최근 검색어</>}  
              </>
            )}
          </div>
        </div>
      </>
    );
  }
}
```

### 상속 2

src/components 에 HistoryList.js 생성

```js
import React from 'react';
import store from '../Store.js';
import List from './List.js';
import { formatRelativeDate } from '../helpers.js';

export default class HistoryList extends List {
  componentDidMount() {
    this.fetch(); 
  }

  fetch() {
    const data = store.getHistoryList();
    this.setState({ data });
  }

  handleClickRemoveHistory(event, keyword) {
    event.stopPropagation();
    store.removeHistory(keyword);
    this.fetch();
  }

  renderItem(item) {
    return (
      <>
        <span>{item.keyword}</span>
        <span className='date'>{formatRelativeDate(item.date)}</span>
        <button
          className='btn-remove'
          onClick={(event)=>this.handleClickRemoveHistory(event, item.keyword)} 
        />
      </>
    )
  }
}
```

App.js 수정

```js
// src/App.js
...
import store from './Store.js';
import keywordList from './components/KeywordList.js';
import HistoryList from './components/HistoryList.js';

export default class App extends React.Component {
  ...
  render() {
    ...
    return (
      <>
        ...
        <div className="container">
          ...
          <div className="content">
            {submitted ? (
              <SearchResult data={searchResult} />
            ) : (
              <>
                ...
                {selectedTab === TabType.KEYWORD && (
                  <KeywordList onClick={(keyword) => this.search(keyword)} />
                )}  
                {selectedTab === TabType.HISTORY && (
                  <HistoryList onClick={(keyword) => this.search(keyword)} />
                )}
              </>
            )}
          </div>
        </div>
      </>
    );
  }
}
```

### 조합: 컴포넌트 담기 1

리액트는 클래스 상속으로 컴포넌트 재활용을 권장하지 않는다

> Facebook 에서는 수천 개의 React 컴포넌트를 사용하지만, 컴포넌트를 상속 계층 구조로 작성을 권장할만한 사례를 아직 찾지 못했습니다.

대신 props를 통해 컴포넌트를 합성하는 방법을 권장함

List 컴포넌트를 조합할 수 있는 방식 변경하고 KeywordList, HistoryList 컴포넌트를 조합

List.js 수정

```js
import React from 'react';

const List = ({data=[], onClick, renderItem}) => {
  return (
    <ul className="list">
      {data.map((item, index) => {
        <li key={item.id} onClick={()=>onClick(item.keyword)}>
          {renderItem(item, index)}
        </li>
      })}
    </ul>
  )
}
export default List;
```

KeywordList.js 수정

```js
// src/components/KeywordList.js
import React from 'react';
import store from '../Store.js';
import List from './List.js';

export default class KeywordList extends React.Component {
  constructor() {
    super();
    this.state = {
      keywordList: [],
    }
  }

  componentDidMount() {
    const keywordList = store.getKeywordList();
    this.setState(
      { keywordList }
    );
  }

  render() {
    return (
      <List
        data={this.state.keywordList}, 
        onClick={this.props.onClick}, 
        renderItem={(item, index)=>{
          return (
            <>
              <span className="number">{index + 1}</span>
              <span>{item.keyword}</span>
            </>
          )
        }}
      />
    )
  }
}
```

### 조합: 컴포넌트 담기 2

HistoryList.js 수정

```js
// src/components/HistoryList.js
import React from 'react';
import { formatRelativeDate } from '../helpers.js';
import store from '../Store.js';
import List from './List.js';

export default class HistoryList extends React.Component {
  constructor() {
    super();
    this.state={
      historyList: [] 
    }
  }

  componentDidMount() {
    this.fetch();
  }

  fetch() {
    const historyList = store.getHistoryList();
    this.setState({ historyList })
  }

  handleClickRemove(event, keyword) {
    event.stopPropagation();
    store.removeHistory(keyword);
    this.fetch();
  }

  render() {
    return (
      <List
        data={this.state.historyList}
        onClick={this.props.onClick}
        renderItem={(item) => {
          return (
            <>
              <span>{item.keyword}</span>
              <span className="date">{formatRelativeDate(item.date)}</span>
              <button
                className="btn-remove"
                onClick={(event)=>this.handleClickRemove(event, item.keyword)}
              >
            </>;
          )
        }}
      />
    )
  }
}
```
 
### 조합: 특수화 1

List.js 수정

```js
import React from 'react';
import {formateRelativeDate} from '../helpers.js';

const List = ({
  data=[], 
  onClick, 
  renderItem, 
  hasIndex = false, 
  hasDate = false, 
  onRemove 
}) => {
  const handleClickRemove(event, keyword) {
    event.stopPropagation();
    onRemove(keyword),
  }
  return (
    <ul className="list">
      {data.map((item,index) => {
        <li key={item.id} onClick={() => onClick(item.keyword)}>
          {hasIndex && <span className="number">{index + 1}</span>}
          <span>{item.keyword}</span>
          {hasDate && <span classNAme="date">{formatRelativeDate(item.date)}</span>}
          {!!onRemove && (
            <button
              className="btn-remove"
              onClick={(event)=>handleClickRemove(event, item.keyword)}
            >
          )
        </li>
      })}
    </ul >
  )
}
export default List;
```

KeywordList.js

```js
// src/components/KeywordList.js
...
export default class KeywordList extends React.Component {
  ...
  render() {
    const { keywordList } = this.state;
    const { onClick } = this.props;

    return (
      <List
        data={keywordList}, 
        onClick={onClick}, 
        hasIndex
      />
    )
  }
}
```

### 조합: 특수화 2

HistoryList.js 수정

```js
// src/components/HistoryList.js
...
export default class HistoryList extends React.Component {
  constructor() {
    super();
    this.state={
      historyList: [] 
    }
  }

  componentDidMount() {
    this.fetch();
  }

  fetch() {
    const historyList = store.getHistoryList();
    this.setState({ historyList })
  }

  handleClickRemove(event, keyword) {
    store.removeHistory(keyword);
    this.fetch();
  }

  render() {
    return (
      <List
        data={this.state.historyList}
        onClick={this.props.onClick}
        hasDate
        onRemove={(keyword)=>this.handleClickRemove(keyword)}
      />
    )
  }
}
```

### 중간정리

코드를 줄이는 방법은 공통 로직을 하나 만들고 이를 재사용하는 것

공통 로직을 부모 클래스에 올리고 이를 상속해서 만드는 방법

단일 역할의 함수를 조합해 또 다른 일을 하는 함수를 만드는 방법

컴포넌트를 재활용하는 방법

#### 상속.

공통 로직을 부모 클래스가 갖도록 함

익숙한 방식이나 상속 단계가 많아지면 코드를 파악하는데 다소 어려움

#### 조합:컴포넌트 담기

리액트의 props 를 활용해서 컴포넌트를 조합

렌더링 용도의 render props를 전달

#### 조합:특수화

이것도 props를 사용한 방식 BUT 접근의 차이

HistoryList 와 KeywordList는 List 컴포넌트의 특수한 경우임

### 최종 정리

리액트 라이브러리의 특성인 리액티브와 가상돔 그리고 컴포넌트에 대해 알아보았음

상태와 UI로 관리되는 화면을 컴포넌트라는 개념으로 추상화

상태가 필요하면 클래스 컴포넌트, 상태가 필요없으면 함수 컴포넌트

상태는 내부의 state 와 외부의 props 가 있음

컴포넌트를 사용하면 여러 장점이 있지만 하나의 상태를 다른 컴포넌트에서 필요한 경우가 있음

이 경우, 부모 컴포넌트로 state를 끌어올려서 해결

컴포넌트는 상속과 조합을 통한 재활용 방법이 있음

리액트는 조합을 권장함

리액트의 세 가지 특징을 중심으로 알아보겠음

데이터 변화에 따라 UI가 반응하는 리액티브한 성질은 상태 관리만으로도 UI관리를 할 수 있음

이는 상태와 UI를 모두 관리해야하는 MVC 패턴과 비교하면 장점임

렌더링 성능을 높이는 가상돔도 리액트의 특징임

상태와 UI를 추상화한 컴포넌트를 통해 화면 개발에 대한 사고 방식을 바꿀 수 있음

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/15/lecture-react-component.html](https://jeonghwan-kim.github.io/series/2021/04/15/lecture-react-component.html)