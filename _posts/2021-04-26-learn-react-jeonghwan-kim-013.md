---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 013
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

### [순수JS 1] 최근 검색어 3

#### 요구사항

> 검색시마다 최근 검색어 목록에 추가된다

#### 구현

- Store.js 수정

```javascript
...
import { createNextId } from "./helpers.js";
...
export default class Store {
  ...
  search(keyword) {
    ...
    );
    this.addHistory(keyword);
  }
  ...
  addHistory(keyword) {
    keyword = keyword.trim();
    if(!keyword) {
      return;
    }
    const hasHistory = this.storage.historyData.some(history => history.keyword === keyword);
    if(hasHistory) {
      this.removeHistory(keyword);
    }
    const id = createNextId(this.storage.historyData);
    const date = new Date();
    this.storage.historyData.push({ id, keyword, date });
    this.storage.historyData = this.storage.historyData.sort(this._sortHistory);
  }
}
```

### 중간정리

- 탭, 추천 검색어, 최근 검색어에 대한 요구사항을 구현하였음
- 뷰는 외부에서 데이터가 들어온다고 가정하고 화면을 그림
- 모델은 데이터만 관리함
- 모델과 뷰를 이용하여 어플리케이션이 돌아가게끔 하는 것이 컨트롤러

### 최종정리

- 백문이 불여일타
- 리액트를 접하기 앞서 순수 자바스크립트로 구현해봄
- MVC 패턴으로 결과물을 구현, UI 라이브러리의 필요성과 리액트의 장점을 강조하려는 의도

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0)