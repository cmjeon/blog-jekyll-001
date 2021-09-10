---
layout: single
title:  nuxt.js 시작하기 - 002
categories: 
  - learn nuxtjs by joshua88
tags: 
  - vuejs
  - nuxtjs
---

## Nuxt.js 프로젝트 시작하기

### Nuxt.js 프로젝트 생성

프로젝트 생성

```bash
$ npm init nuxt-app learn-nuxt
```

1. Project name : Enter
1. Programming language : Javascript
1. Package manager :Npm
1. UI framework : None
1. Nuxt.js module : Enter
1. Linting tools : ESLint, Prettier
1. Testing framework: None
1. Rendering mode : Universal (SSR / SSG)
1. Deployment target : Server (Node.js hosting)
1. Development tools : jsconfig.json
1. Continuous integration : None
1. Version control sytem : Git


```bash
$ cd learn-nuxt

# to get startd
$ npm run dev

# to build & start
$ npm run build
$ npm run start
```

### 프로젝트 폴더 구조 설명

터미널에서 `$ code .` 으로 vscode 실행하기

1. vscode 실행
1. cmd + shift + p
1. install code 입력 > 엔터

```bash
# vs code 실행
$ code .
```

nuxt 프로젝트의 폴더구조

```
|- .nuxt : 빌드 결과물 폴더
|- assets : 이미지, css 등 리소스
|- components : vue 컴포넌트
|- layouts : 라우터 기준으로 특정 페이지 레이아웃 컴포넌트
|- middleware : 서버사이드렌더링 진행 시 서버에서 브라우저로 파일을 넘기기 전에 항상 실행하는 함수의 모음
|- pages : 파일기반의 라우팅
|- plugins : vue 플러그인, 인스턴스
|- static : 빌드했을 때 서버의 루트주소에 이동되는 정적 자원
|- store : vuex 의 스토어
```

### 메인 페이지 생성 및 결과 확인




### 페이지 컴포넌트 빌드 결과 확인



### 에러 페이지 정의 방법



### Nuxt 의 라우팅 관련 컴포넌트 정리



### 레이아웃 컴포넌트 소개



### nuxt-link 태그 소개





## 참고

- [https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0](https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)