---
layout: single
title:  만들면서 학습하는 리액트(react):사용편 - 024
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 사용편

### 폼 초기화

#### 요구사항

> x 버튼을 클릭하거나 검색어를 삭제하면 검색 결과를 삭제한다

#### 구현

```javascript
// main.js
class App extends React.Component {
  ...
  handleReset() {
    this.setState(() => {
      return { searchKeyword: ""}
    }, () => {
      console.log('handleReset', this.state.searchKeyword);
    });
  }

  handleChangeInput(event) {
    const searchKeyword = event.target.value;

    if(searchKeyword.length <= 0) {
      return this.handleReset();
    }
    this.setState({ searchKeyword });
  }

  render() {
    return (
      <>
      <div className="container">
        <form
          onSubmit={ (event) => this.handleSubmit(event)}
          onReset={ ()) => this.handleReset()}
        >
          ...
        </form>
      </div>
      </>
    );
  }
}
```

setState는 항상 비동기로 동작한다

setState를 실행해도 바로 다음 코드 전에 실행된다는 보장이 없다

setState() 함수가 완료되는 시점을 찾아야 함

setState() 함수는 값을 전달 할 수도 있지만 함수를 전달할 수도 있음

```javascript
setState(() => {}, () => {})
```

setState()) 함수는 앞의 함수로 상태를 변경하고 리턴을 하고, 뒤의 함수가 상태가 변경된 이후에 실행되게 할 수 있음

### 중간정리

검색어 입력에 따라 보이고 안보이는 것을 통해 조건부 렌더링을 학습함

조건부 렌더링 하는 여러 방법이 있음

폼 제출 시점에 onSubmit 속성이 있음

폼 초기화, 검색어 삭제 시, x 버튼을 숨기는 기능을 만들었음

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html](https://jeonghwan-kim.github.io/series/2021/04/12/lecture-react-usage.html)
- [https://ko.reactjs.org/docs/faq-state.html](https://ko.reactjs.org/docs/faq-state.html)
