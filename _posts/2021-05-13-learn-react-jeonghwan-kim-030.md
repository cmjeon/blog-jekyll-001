---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 030
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 사용편2

### SearchForm 1

#### 요구사항

> 검색 상품명을 입력할 수 있는 폼이 위치한다   
> 검색어를 입력하면 x 버튼이 보이고 검색어를 삭제하면 x 버튼을 숨긴다

#### 구현

src/components에 SearchForm.js 생성

```javascript
// src/components/SearchForm.js
import React from 'react';

export default class SearchForm extends React.Component {
  constructor() {
    super();
    this.state = {
      searchKeyword: "",
    };
  }

  handleChangeInput(event) {
    const searchKeyword = event.target.value;
    this.setState({searchKeyword});
  }

  render() {
    return (
      <form>
        <input
          type="text"
          placeholder="검색어를 입력하세요"
          autFocus
          value="{this.state.searchKeyword}"
          onChange={(event) => this.handleChangeInput(event)}
        />
        {this.state.searchKeyword.length > 0 && (
          <button type="reset" className="btn-reset"></button>
        )}
      </form>
    )
  }
}
```

App.js 수정

```javascript
// src/App.js
import React from 'react';
import Header from './components/Header.js';
import SearchForm from './components/SearchForm.js';

export default class App extends React.Component {
  render() {
    return (
      <>
        <header title="검색">
        <div className="container">
          <SearchForm />
        </div>
      </>
    )
  }
}
```

### SearchForm 2

#### 요구사항

> 엔터를 입력하면 검색 결과가 보인다

리액트에서는 부모에서 자식으로 데이터를 props로 전달하는 방식은 자연스러움

SearchForm 에서는 엔터를 입력하면 App 으로 무엇인가가 전달되어야 하는 상황임

SearchForm 에 props 로 callback 함수를 전달해서 해결해보겠음

#### 구현

SearchForm.js 수정

```javascript
// src/components/SearchForm.js
...
export default class SearchForm extends React.Component {
  ...
  handleSubmit(event) {
    event.preventDefault();
    this.props.onSubmit(this.state.searchKeyword); 
  } 
  ...
  render() {
    const { searchKeyword } = this.state;

    return (
      <form
        onSubmit={(event) => this.handleSubmit(event)}
      >
        ...
      </form>
    )
  }
}
```

App.js 수정

```javascript
// src/App.js
...
export default class App extends React.Component {
  ...
  search(searchKeyword) {
    console.log('TODO: search', searchKeyword);
  }
  ...
  render() {
    return (
      <>
        ...
        <div className="container">
          <SearchForm
            onSubmit={(searchKeyword) => this.search(searchKeyword)}
          >
        </div>
      </>
    )
  }
}
```

#### 요구사항

> x 버튼을 클릭하거나 검색어를 삭제하면 검색 결과를 삭제한다

#### 구현

SearchForm.js 수정

```javascript
// src/components/SearchForm.js
...
export default class SearchForm extends React.Component {
  ...
  handleReset() {
    this.props.onReset();
  }
  ...
  handleChangeInput(event) {
    const searchKeyword = event.target.value;
    if(searchKeyword.length <= 0) {
      this.handleReset(); 
    }
    this.setState({searchKeyword});
  }
  ...
  render() {
    ...
    return (
      <form
        onSubmit={(event) => this.handleSubmit(event)}
        onReset={() => this.handleReset()}
      >
        {searchKeyword.length > 0 && (
          <button type="reset" className="btn-reset" />
        )}
      </form>
    )
  }
}
```

App.js 수정

```javascript
// src/App.js
...
export default class App extends React.Component {
  ...
  handleReset() {
    console.log('TODO: handleReset');
  }
  ...
  render() {
    return (
      <>
        ...
        <div className="container">
          <SearchForm
            onSubmit={(searchKeyword) => this.search(searchKeyword)}
            onReset={() => this.handleReset()}
          >
        </div>
      </>
    )
  }
}
```

### State 끌어오기

vanilla 에서 MVC 패턴을 이용해서 SearchFormView 를 만들었음

SearchFormView 는 커스텀 이벤트('@submit' 등)를 발행해서 상태변화를 전파하였음

Controller 에서는 커스텀 이벤트를 구독하고 모델과 다른 뷰들을 관리하는 방식으로 구현하였음

react 에서는 App 이라는 리액트 컴포넌트를 이용하여 만들었음

검색어를 저장할 searchKeyword 라는 state 를 만들었음

input 에서 event 가 발생하면 change 이벤트가 발생하고 그 상태값인 searchKeyword 를 변경하였음

리액트 컴포넌트는 state 의 변화를 알고 화면을 render 하였음

component 에서는 SearchForm 이라는 컴포넌트를 만들었음

원래 App 에 있던 searchKeyword 상태를 SearchForm 이라는 컴포넌트 내부로 가져옴

이렇게 되면 searchKeyword 는 SearchForm 내부에서 관리하므로 외부에서 해당 상태에 직접 접근하도록 해서는 안됨

하지만, 요구사항 중에 추천 검색어, 최근 검색어 선택 시 SearchForm 안의 input 에 값을 설정해야하는 경우가 있음

searchKeyword 처럼 여러 컴포넌트가 의존하는 상태는 가장 가까운 부모 컴포넌트로 `state 끌어올리기` 해야 함

 > 종종 동일한 데이터에 대한 변경사항을 여러 컴포넌트에 반영해야 할 필요가 있습니다. 이럴 때는 가장 가까운 부모 컴포넌트로 state를 끌어올리는 것이 좋습니다.

App.js 수정

```javascript
// src/App.js
...
export default class App extends React.Component {
  consturctor() {
    super();

    this.state = { searchKeyword: "" }
  }
  ...
  handleChangeInput(searchKeyword) {
    if(searchKeyword.length <= 0) {
      this.handleReset();
    }
    this.setState({ searchKeyword });
  }
  ...
  render() {
    return (
      <>
        ...
        <div className="container">
          <SearchForm
            value={this.state.searchKeyword}
            onChange={(value) => this.handleChangeInput(value)}
            onSubmit={(searchKeyword) => this.search(searchKeyword)}
            onReset={() => this.handleReset()}
          / >
        </div>
      </>
    )
  }
}
```

SearchForm 에서는 관리하는 state 가 없으므로 함수형으로 변경함

SearchForm.js 수정

```javascript
// src/components/SearchForm.js
import React from 'react';

const SearchFrom = ({value, onChange, onSubmit, onReset}) => {
  const handleSubmit = (event) => {
    event.preventDefault();
    onSubmit();
  }

  const handleReset = () => {
    onReset();
  }

  const handleChangeInput = () => {
    const searchKeyword = event.target.value;
    onChange(event.taget.value); 
  }

  return (
    <form
      onSubmit={handleSubmit}
      onReset={handleReset}
    >
      <input
        type="text"
        placeholder="검색어를 입력하세요"
        autoFocus
        value={value}
        onChange={handleChangeInput}
      />
      {value.length > 0 && (
        <button type="reset" className="btn-reset" />
      )}
    </form>
  )
}
export default SearchForm
```
### SearchResult

#### 요구사항

> 검색 결과가 검색폼 아래 위치한다. 검색 결과가 없을 경우와 있을 경우를 구분한다.   
> x 버튼을 클릭하면 검색폼이 초기화 되고, 검색 결과가 사라진다.

#### 구현

2-react/js 폴더의 Store.js, storage.js, helpers.js 를 3-component/src 폴더로 복사해온다

src/components 에 SearchResult.js 생성

```javascript
// src/components/SearchResult.js
import React from 'react';

const SearchResult = ({data=[]}) => {
  if(data.length <= 0) {
    return (
      <div className="empty-box">검색 결과가 없습니다</div>
    )
  }

  return (
    <ul className="result">
      {data.map(({ id, imageUrl, name }) => (
        <li key={id}>
          <img src={imageUrl} />
          <p>{name}</p>
        </li>
      ))}
    </ul>
  );
};
export default SearchResult;
```

App.js 수정

```javascript
// src/App.js
...
import SearchResult from './components/SearchResult.js';
import store from './Store.js'

export default class App extends React.Component {
  constructor() {
    super();

    this.state = {
      searchKeyword: '',
      searchResult: [],
      submitted: false,
    }
  }
  ...
  search(searchKeyword) {
    const searchResult =  store.search(searchKeyword);
    this.setState({
      searchResult,
      sumbitted:true,
    })
  }
  ...
  handleReset() {
    this.setState({
      searchKeyword: '',
      submitted: false,
      searchResult: [],
    })
  }
  ...
  render() {
    const {searchKeyword, submitted, searchResult} = this.state;
    return (
      <>
        ...
        <div className="container">
          ...
          <div className="content">
            {submitted && <SearchResult data={searchResult} />}
          </div>
        </div>
      </>
    )
  }
}
```

### Tabs

#### 요구사항

> 추천 검색어, 최근 검색어 탭이 검색폼 아래 위치한다   
> 기본으로 추천 검색어 탭을 선택한다   
> 각 탭을 클릭하면 탭 아래 내용이 변경된다

#### 구현

src/components에 Tabs.js 생성

```javascript
// src/components/Tabs.js
export const TabType = {
  KEYWORD: "KEYWORD",
  HISTORY: "HISTORY",
};

const TabLabel = {
  [TabType.KEYWORD]: "추천 검색어",
  [TabType.HISTORY]: "최근 검색어",
}

const Tabs = ({selectedTab, onChange}) => {
  return (
    <>
      <ul className="tabs">
        {Object.values(TabType).map((tabType) => (
          <li
            key={tabType}
            className={selectedTab === tabType ? "active" : ""}
            onClick={() => onChange(tabType))}
          >
            {TabLabel[tabType]}
          </li>
        ))}
      </ul>
    </>
  )
}
export default Tabs;
```

App.js 수정

```javascript
// src/App.js
...
import Tabs, {TabType} from './components/Tabs.js'

export default class App extends React.Component {
  constructor() {
    super();
    this.state = {
      ...
      submitted: false,
      selectedTab: TabType.KEYWORD
    }
  }
  ...
  render() {
    ...
    return (
      <>
        ...
        <div className="contianer">
          <div className="content">
            {submitted ? (
              <SearchResult data={searchResult}/>
            ) : (
              <>
                <Tabs
                  selectedTab={selectedTab}
                  onChange={(selectedTab) => this.setState({selectedTab})}
                />
                {selectedTab === TabType.KEYWORD && <>TODO: 추천 검색어 목록</>}
                {selectedTab === TabType.HISTORY && <>TODO: 최근 검색어 목록</>}
              </>
            )} 
          </div>
        </div>
      </>
    )
  }
}
```

### 중간정리

단일 컴포넌트로 만들었던 어플리케이션을 작은 컴포넌트로 분리하였음

SearchForm 에서 사용자 입력(searchKeyword)을 state 로 직접 관리하였음

하지만 이 state 는 다른 컴포넌트에서도 의존하기 때문에 가까운 부모 컴포넌트로 끌어올렸음

SearchResult 도 상태(searchResult)는 App에서 유지하고 data로 전달하였음

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/15/lecture-react-component.html](https://jeonghwan-kim.github.io/series/2021/04/15/lecture-react-component.html)