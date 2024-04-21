---
layout: single
title:  nuxt.js 시작하기 - 001
categories: 
  - learn nuxtjs by joshua88
tags: 
  - vuejs
  - nuxtjs
---

## Nuxt.js 소개

### Nuxt.js 강의 소개

- Server Side Rendering 을 구현할 때 가장 많이 사용되는 프레임워크
- 빠른 페이지 로딩 속도와 성능 최적화에 대한 해답
- SEO
- 쇼핑몰 사이트를 제작하며 nuxt.js 기본 개념과 활용 방법

### 개발환경 및 수업 코드 리포지토리 안내

- 수업 코드 리포지토리
[https://github.com/joshua1988/learn-nuxt](https://github.com/joshua1988/learn-nuxt)

### Nuxt.js 소개

#### 공식사이트

- Nuxt.js 공식사이트
[https://nuxtjs.org/](https://nuxtjs.org/)

- Vue Framework 의 Framework

#### 강의교안

- Cracking Vue.js
[https://joshua1988.github.io/vue-camp/nuxt/intro.html](https://joshua1988.github.io/vue-camp/nuxt/intro.html)

#### 참고

- 타입스크립트 로드맵
[https://www.inflearn.com/roadmaps/387](https://www.inflearn.com/roadmaps/387)

#### Nuxt 란?

- Vue.js 로 빠르게 웹을 제작할 수 있게 도와주는 프레임워크
- 장점
    - 검색 엔진 최적화
    - 초기 프로젝트 설정 비용 감소
    - 페이지 로딩 속도 향상
- Nuxt 특징
    - 서버 사이드 렌더링
    - 규격화된 폴더 구조
    - pages 폴더 기반의 자동 라우팅 설정
    - 코드 스플리팅
    - 비동기 데이터 요청 속성
    - ES6/ES6+ 변환
    - 웹팩을 비롯한 기타 설정
- Nuxt 시작하기

    ```
    $ npm init nuxt-app {프로젝트명}
    $ cd {프로젝트명}
    $ npm run dev
    ```

### 서버 사이드 렌더링이란?

[https://joshua1988.github.io/vue-camp/nuxt/ssr.html](https://joshua1988.github.io/vue-camp/nuxt/ssr.html)

- 서버에서 페이지를 그려 클라이언트로 보낸 후 화면에 표시하는 기법

### 클라이언트 사이드 렌더링과 서버 사이드 렌더링의 차이점

- CSR, SSR 차이 페이지의 내용을 브라우저에서 그려주는가, 서버에서 그려주는가 차이
- 서버 사이드 렌더링은 검색 엔진 최적화와 빠른 페이지 렌더링을 목적으로 쓰임
- 서버 빌드에 대한 이해 필요
- 페이지를 서버에서 그려서 던져주면 서버 사이드 렌더링 SSR
- 자바스크립트 라이브러이에 의해 비어져 있는 태그영역이 채워지면 클라이언트 사이드 렌더링 CSR

### 클라이언트 사이드 렌더링과 서버 사이드 렌더링 결과 비교

#### CSR

carers.google.com

- vue.js CSR 로 개발되어있음
- 개발자 도구 진입 -> 새로고침
- 네트워크탭의 필터에 Doc 를 선택하고 리로드하면 최초의 문서를 확인할 수 있음
- SPA 의 경우 브라우저에서 렌더링을 진행 함

#### SSR

joshua1988.github.io

- vue.js + nuxt.js 로 개발되어있음
- 개발자 도구 진입 -> 새로고침 -> Doc 필터
- 화면을 구성하는 요소들을 서버에서 불러옴

### 서버 사이드 렌더링을 왜 쓸까

#### 검색 엔진 최적화

[SEO 기본가이드](https://developers.google.com/search/docs/beginner/seo-starter-guide?hl=ko)

- SPA 에는 별도의 조치없이는 og 태그가 구분되어 표시되지 못함

#### 빠른 페이지 렌더링

- 빈 페이지를 받아서 브라우저에서 그려주는 방식보다 서버에서 미리 그려서 브라우저에서 보여주는 방식이 빠름

#### SSR의 단점

- 서버쪽 지식이 필요
- node.js 웹 어플리케이션 실행 방법을 알아야 함
- node.js 환경에서 실행되기 때문에 브라우저 관련 API 를 사용할 때 주의하여야 함

### Nuxt의 장점과 특징

#### Nuxt 장점

- 검색 엔진 최적화
- 초기 프로젝트 설정 비용 감소와 생산성 향상
    - ESLint, Prettier
    - 라우터, 스토어 등의 라이브러리 설치 및 설정 파일 불필요
    - 파일 기반의 라우팅 방식, 설정 파일 자동 생성
- 페이지 로딩 속도와 사용자 경험 향상
    - 브라우저가 하는 일을 서버에 나눌 수 있음
    - 코드 스플리팅 기본 설정

#### Nuxt 특징

- 서버 사이드 렌더링
- 규격화된 폴더 구조
- pages 폴더 기반의 자동 라우팅 설정
- 코드 스플리팅
- 비동기 데이터 요청 속성
- ES6/ES6+ 변환

## 참고

- [https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0](https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)
