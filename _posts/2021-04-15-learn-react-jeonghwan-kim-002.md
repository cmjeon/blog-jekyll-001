---
layout: single
title:  만들면서 학습하는 리액트(react):준비편 - 순수JS - 002
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 준비편 - 순수JS

비교와 대조를 통해 리액트에 대한 이해력을 높일 수 있음

리액트 라이브러리가 없으면 어떻게 개발할까 > 순수 자바스크립트, 바닐라 자바스트립트로 개발함

따라서 결과물을 순수 자바스크립트로 개발해봄으로 리액트의 역할과 이점을 더 확실하게 배워보려고 함

### [순수JS 1] MVC 패턴

프로그래밍 세상에서는 문제를 해결하는 접근법을 `디자인 패턴` 이라고 함

화면 개발에서 많이 쓰이는 디자인 패턴 중 하나가 `MVC 패턴` 임

모델(Model), 뷰(View), 컨트롤러(Controller) 라는 3개의 역할이 서로 협력해 문제를 해결하는 방식

모델(Model)은 데이터를 관리(CRUD), 뷰(View)는 사용자가 보는 화면을 관리(데이터의 출력, 이벤트 처리), 컨트롤러(Controller)는 모델과 뷰를 연결하고 움직이는 주체

### [순수JS 1] 폴더 구조

1. 저장소로부터 코드 가져오기

    ```bash
    $ git clone https://github.com/jeonghwan-kim/lecture-react.git
    ```

1. 브랜치 이동

    ```bash
    $ git checkout -f ready/scaffoding
    ```


#### index.html

- html 구조
- ./style.css
    - 학습할 때 css는 수정하지 않음
- ./js/main.js
- 본문에 `<header>`, `<div>` 있음

#### storage.js

- 데이터가 있는 곳
- Model 역할

#### store.js

- class 로 정의되어 있음
- 생성 시 storage 받아서 저장

#### views/View.js

- class 로 정의되어 있음
- Views는 많아서 폴더로 관리
- 생성 시 element 받아서 저장
- View 역할
- 화면을 숨기고, 보여주는 역할(hide, show)
- element.style.display를 originalDisplay로 저장
- 이벤트를 수신하고, 발행하는 역할(on, emit)

#### helper.js

- 돔 api를 wrapping 해놓은 것
- qs는 document.querySelector()를 호출
- qsAll은 document.querySelectorAll()을 호출해서 배열로 반환
- on은 target.addEventListener()를 호출
- delegate은 on과 유사, 특정 엘리먼트의 하위 엘리멘트의 이벤트를 처리할 때 사용
- emit은 target.dispatchEvent()를 호출
- formatRelativeDate은 상대날짜 생성 포매터
- createPastDate은 하루전, 이틀전 날짜 객체 생성
- createNextId은 아이디값을 +1

#### Controller.js

- class로 정의
- store와 view를 받음

#### js/main.js

- DOM 로딩이 완료되면 main 함수 호출
- storage 객체를 사용해서 store 객체 생성
- view 객체 생성
- store, view 객체를 사용해서 Controller 객체 생성

#### 개발서버 동작확인

```bash
$ npx lite-server --baseDir 1-vanilla/
```

- tag 추가

    ```javascript
    // main.js
    ...
    import storage from "./storage.js";

    const tag = "[main]"; // add
    ducument.addEventListener("DomContentLoaded", main);
    function main() {
        console.log(tag, "main"); // add
        const store = new Store(storage);
        ...
    }
    ```

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0](https://jeonghwan-kim.github.io/series/2021/04/05/lecture-react-ready.html#%EA%B2%B0%EA%B3%BC%EB%AC%BC-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0)