---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 021
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 사용편

### Input 요소 사용하기

#### 요구사항

> 검색 상품명을 입력할 수 있는 폼이 위치한다

#### main.js

```javascript
const element = (
  <>
    <header>
      <h2 className="container">검색</h2>
    </header>
    <div className="container">
      <form>
        <input type="text" placeholder="검색어를 입력하세요" autoFocus />
        <button type="reset" className="btn-reset"></button>
      </form>
    </div>
  </>
);
...
```

- className, autoFocus를 CamelCase로 사용하자
- `<>`, `</>` 는 fragment

### 리액트 컴포넌트

> React는 ... "컴포넌트"라고 불리는 작고 고립된 코드의 파편을 이용하여 복잡한 UI를 구성하도록 돕습니다.

리액트는 리액트 방식의 상태가 필요함

컴포넌트는 UI를 나타내는 엘리먼트와 어플리케이션 로직을 포함한 상위 개념

입력값의 상태를 기억하기 위해 컴포넌트가 필요함

#### main.js

```javascript
class App extends React.Component {
  render() {
    return (
      <>
        <header>
          <h2 className="container">검색</h2>
        </header>
        <div className="container">
          <form>
            <input type="text" placeholder="검색어를 입력하세요" autoFocus />
            <button type="reset" className="btn-reset"></button>
          </form>
        </div>
      </>
    );
  }
}
ReactDOM.render(<App />, document.querySelector("#app"));
```

- render() 함수를 통해 리액트 엘리먼트를 반환
- React.Component를 상속한 class의 render()를 만들면 컴포넌트가 됨
- ReactDOM.render에 JSX 문법으로 `<App />` 을 넣으면 리액트 엘리먼트를 반환하여 실제돔에 반영

### 입력 값을 기억하기 - State

컴포넌트에서 UI를 나타내는 부분이 render라면, 어플리케이션 로직이 State임

#### main.js

```javascript
class App extends React.Component {
  constructor() {
    super();
    this.state = {
      searchKeyword: '',
    };
  }

  render() {
    return (
      <>
        ...
        <div className="container">
          <form>
            <input type="text" placeholder="검색어를 입력하세요" autoFocus value={this.state.searchKeyword}>
            ...
          </form>
        </div>
      </>
    );
  }
}
```

input 박스에서 입력을 해도 반응하지 않음. 브라우저에 input 태그에서 change 이벤트를 관리하고 있기 때문

onChange 이벤트도 리액트에서 관리하게끔 해줘야 함

### 상태를 갱신하기 - 이벤트 처리

input 태그에 값을 입력하면 change 이벤트가 발생함

#### main.js

```javascript
class App extends React.Component {
  constructor() {
    ...
  }

  handleChangeInput(event) {
    this.state.searchKeyword = event.target.value;
    this.forceUpdate();
  }

  render() {
    return (
      <>
        ...
        <div className="container">
          <form>
            <input 
              ...
              value={this.state.searchKeyword}
              onChange={event => this.handleChangeInput(event)}
            >
            ...
          </form>
        </div>
      </>
    );
  }
}
```

값을 입력해도 여전히 안됨.

리액트 컴포넌트는 state를 변경할 때 주의할 점이 있음

state가 변경되도 컴포넌트의 render 함수가 실행되지 않음

this.forceUpdate() 함수를 호출하면 render() 함수를 호출해서 화면을 갱신해줌

그런데 this.forceUpdate() 함수를 호출하기 보다 리액트 컴포넌트가 스스로 render() 함수를 호출하게 하기 위해 변경해보자

#### main.js

```javascript
class App extends React.Component {
  constructor() {
    ...
  }

  handleChangeInput(event) {
    this.setState({
      searchKeyword: event.target.value,
    });
  }

  render() {
    ...
  }
}
```

### 중간정리

브라우저가 기본적으로 input 엘리먼트의 입력을 관리함. 리액트에서 input을 관리하려면 리액트 컴포넌트를 사용해야 함

리액트 컴포넌트는 상태 관리를 위한 내부 변수 state를 가지고 있음

컴포넌트 클래스의 setState() 메서드를 사용하면 컴포넌트는 상태의 변화를 인지하고 UI에 영향을 줄 수 있음

input 엘리먼트 자체의 상태 관리를 사용하지 않고 리액트 컴포넌트가 관리하는 것을 제어 컴포넌트(Controlled Component)라고 함

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html](https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html)
