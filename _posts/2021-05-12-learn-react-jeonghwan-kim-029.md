---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 029
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 사용편2

### 컴포넌트를 사용하는 이유

사용편1에서 `React.Componet`를 상속받은 `App` 컴포넌트를 사용하였음

App 컴포넌트는 큰 서랍에 모든 물건을 넣어놓은거와 다름없는 상황임

서랍을 잘 칸막이해둔다며 물건을 관리하기 쉬울 것임

현재의 `App` 컴포넌트는 역할에 맞게 더 작은 컴포넌트로 나눌 수 있음

컴포넌트를 사용하면 개발자는 추상화를 통해 더 크고 복잡한 코드를 만들어 볼 수 있음

### 작은 컴포넌트로 나누기

`App` 컴포넌트를 역할에 맞게 나누어 보자

단일책임의 원칙에 따라 컴포넌트를 만들 때는 한가지 역할만을 가지도록 만들어야 함

`App` 컴포넌트의 역할을 나열해 보자

- 헤더를 출력한다
- 검색폼에 검색어 입력을 받는다
- 엔터를 누르면 검색하고 결과를 보여준다
- 추천 검색어, 최근 검색어 탭이 있다
- 각 탭을 클릭하면 추천 검색어 목록, 최근 검색어 목록이 각각 표시된다

위 역할을 각각의 컴포넌트로 만들어보자

- Header : 헤더를 출력한다
- SearchForm : 검색어 입력을 받는다
- SearchResult : 검색결과를 보여준다
- Tab : 추천 검색어, 최근 검색어 탬
- KeywordList : 추천 검색어 목록
- HistoryList : 최근 검색어 목록
- App : 부분 컴포넌트를 관리하는 전체 컴포넌트


### 프로젝트 구조 변경하기

브랜치 전환

```bash
$ git checkout -f component/scaffolding
```

3-component 폴더가 생성됨

package.json 에는 라이브러리가 있음

여러 파일들을 서로 모듈로 연결하려면 지금까지 사용한 babel standalone 으로는 한계가 있음

그래서 webpack.config.js 설정으로 babel 모듈을 사용함

package.json 에 명시된 dependencies에 있는 라이브러리들을 다운로드

```bash
$ npm i
```

프로젝트 시작

```bash
$ npm start
```

index.html 을 살펴보면 css 를 로딩하고 `./main.j` 를 로딩하였음

src 에 App.js 생성

```javascript
// src/App.js
import React from 'react';

class App extends React.Component {
  render() {
    return (
      <>
        TODO: App 컴포넌트
      </>
    );
  }
}
```

main.js 수정

```javascript
// src/main.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App.js';

ReactDOM.render(<App />, document.querySelector('#app'));
```

### Header - 함수 컴포넌트

그 동안은 Component 클래스를 상속해 리액트 컴포넌트를 만들었음

함수 컴포넌트를 사용하여 만들 수도 있음

```javascript
const Hello = () => <>Hello World</>

<Hello />
```

함수 컴포넌트가 클래스 클래스와의 차이는 state가 없다는 점

내부 상태가 필요없는 컴포넌트라면 함수 컴포넌트를 사용하면 됨

컴포넌트 중 `Header` 컴포넌트는 함수 컴포넌트로 생성하기 적절함

src/components 에 Header.js 생성

```javascript
// src/components/Header.js
import React from 'react';

const Header = () => {
  return (
    <header>
      <h2 className="container">검색</h2>
    </header>
  )
}
export default Header;
```

App.js 수정

``` javascript
// src/App.js
import React from 'react';
import Header from './components/Header.js';

export default class App extends React.Component {
  render() {
    return <Header />;
  }
}
```

### 리액트 개발 도구

리액트 개발 시 개발 도구를 사용하면 편리함

npm의 react-devtools

```bash
# yarn
$ yarn global add react-devtools 
# npm
$ npm install -g react-devtools
```

혹은 파이어폭스나 크롬의 확장프로그램을 설치해서 사용가능

### 재활용 가능한 컴포넌트로 개선 - Props

함수 컴포넌트와 클래스 컴포넌트의 차이는 상태의 유무

상태가 바뀜에 따라 UI가 바뀌는 것이 리액트의 성격인데 그럼 함수 컴포넌트는 무엇?

리액트 컴포넌트에서 UI의 상태로 사용가능한 것은 state 말고도 props 가 있음

state 는 컴포넌트 내부에서 관리하는 상태라면 props 는 컴포넌트 외부에서 들어와 내부 UI에 영향을 줄 수 있음

```javascript
const Hello = ({ name }) => <>Hello { name }</> //1

<Hello name="world" />
```

Props 는 객체 모양으로 컴포넌트에 전달되고, 전달된 값으로 리액트 엘리먼트를 생성함

Header.js 수정

```javascript
// src/components/Header.js
...
const Header = ( props ) => (
  <header>
    <h2 className="container" >{ props.title }</h2>
  </header>
)
export default Header;
```

App.js 수정

```javascript
// src/App.js
...
export default class App extends React.Component {
  render() {
    return <header title="검색">
  }
}
```

### 중간정리

상태와 UI 코드로 이루어진 화면 개발에서도 컴포넌트를 사용해 추상화 하였음

`App` 이라는 컴포넌트를 작은 컴포넌트로 분리하여 가독성과 유지보수성을 높일 수 있음

컴포넌트는 `클래스 컴포넌트`와 `함수 컴포넌트` 두 가지 종류가 있음

스스로 상태 관리가 필요한 경우는 클래스 컴포넌트를 사용하고, 아니면 함수 컴포넌트만으로도 충분함

컴포넌트 내부에 위치한 state가 UI와 밀접하게 반응함, 외부에서 전달받는 props도 UI와 반응함

Header 컴포넌트의 title을 props로 전달해 재활용할 수 있는 형태로 개선하였음

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/15/lecture-react-component.html](https://jeonghwan-kim.github.io/series/2021/04/15/lecture-react-component.html)