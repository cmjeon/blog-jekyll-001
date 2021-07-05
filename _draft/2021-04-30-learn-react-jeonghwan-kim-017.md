---
layout: single
title:  만들면서 학습하는 리액트(react):소개편 - Hello world - 017
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 소개편 

### Hello world

상태변화를 UI에 반영하려면 DOM API 호출이 불가피

상태가 변하는만큼 DOM API 호출이 비례함 > 성능에 영향을 주게 됨

브라우져가 HTML과 css로 화면을 그리는 과정 - `주요렌더링경로`(Critical Render Path)
1. HTML 코드를 파싱해서 DOM 트리를 만든다
1. css 코드를 cssom 트리로 만든다
1. 두 트리를 합쳐 렌더트리를 만든다
1. 레이아웃을 계산한다
1. 픽셀로 화면을 그린다

`주요렌더링경로`는 DOM 구조가 변경되면 레이아웃을 다시 계산하고 픽셀로 화면을 그림

반복되는 작업이기 때문에 렌더링 성능에 영향을 줌

개선방안은? 캐쉬 계층을 만들어 DOM API를 적게 사용하게 함

트리 구조의 돔과 유사한 `가상돔(Virtual DOM)`을 만들어 메모리에서 관리함

가상돔으로 화면을 그리는 과정
1. 어플리케이션에서 변경이 일어나면 가상돔에 요청함
1. 리액트는 렌더링할 때 마다 가상돔을 만들고, 이전 가상돔과의 차이를 찾음
1. 차이가 있는 부분만 실제 돔에 반영하고 차이가 없는 경우 렌더링 요청을 무시함


`가상돔`은 기술이 아니라 패턴임

`가상돔`은 리액트에서는 UI를 말하는 개념으로 사용됨. 리액트 엘리먼트(react element)와도 관련있음

### 중간 정리

순수 자바스크립트로 MVC 패턴을 이용하여 구한 순수JS
- 모델은 어플리케이션의 상태를 관리
- 뷰는 돔으로 UI를 변경
- 컨트롤러는 모델과 뷰를 사용하여 어플리케이션을 관리

상태만 변경면 자동으로 UI를 변경하게 하는 것이 `리액티브`

`리액트`는 리액티브 프로그래밍으로 UI 개발을 할 수 있도록 돕는 라이브러리

`가상돔`은 매번 DOM에 접근하지 않고 꼭 필요할 때만 접근하는 방식으로 성능을 개선시키는 아이디어

리액트는 `컴포넌트`라는 추상화 개념을 사용함

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html](https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html)
- [https://developers.google.com/web/fundamentals/performance/critical-rendering-path?hl=ko](https://developers.google.com/web/fundamentals/performance/critical-rendering-path?hl=ko)