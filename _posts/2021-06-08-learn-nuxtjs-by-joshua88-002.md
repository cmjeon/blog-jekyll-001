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

파일기반의 자동라우팅

pages 폴더에 파일을 만들면 해당 URL 에서 파일명의 경로로 이동가능함

pages/index.vue 가 domain/ 을 표시하는 파일

layouts/default.vue 에 의해서

<Nuxt></Nuxt> 가 <router-view></router-view> 임

따라서 layout/default.vue 의 Nuxt에 의해서 페이지가 이동됨

### 페이지 컴포넌트 빌드 결과 확인

pages 폴더에 파일을 생성하면 라우팅이 되면서 페이지접근이 가능해짐

.nuxt 안의 router.js 확인

```
routes: [{
  path: "/main",
  component: ...
}]
```

에 의해 routes 가 자동생성됨을 알 수 있음

pages/product/index.vue 생성

routes 에 /product 가 생성됨

pages/product.vue 생성을 해도 위의 결과와 동일함

### 에러 페이지 정의 방법

현재 등록된 파일들 외에 다른 파일을 URL 로 호출하면 에러페이지가 발생

layouts/error.vue 생성

```
<template>
  <div>
    <h1>404 페이지</h1>
    <p>페이지를 찾을 수 없습니다</p>
  </div>
</template>
<script>
export default {}
</script>
<style></style>
```

에러페이지가 표시됨

### Nuxt 의 라우팅 관련 컴포넌트 정리

layouts > pages > components 간의 위계를 확인

vue-router 에서 페이지를 이동할 때마다 호출하는 페이지 컴포넌트가 Nuxt 에서 pages 폴더임

### 레이아웃 컴포넌트 소개

도메인/ 호출 시 접근되는 페이지는 홈페이지

도메인/main 호출 시 접근되는 페이지는 메인페이지

헤더의 경우는 pages 보다 layouts 에서 정의하는 것이 좋음

layouts에 정의한 헤더의 값을 어떻게 바꿀 수 있을까?

{% raw %}
```html
<h1>{{ $route.name }} 페이지</h1>
```
{% endraw %}

라우터의 정보를 바로 헤더의 값으로 활용할 수 있음

layouts 로의 분리를 통해 pages 는 본연의 내용에 충실할 수 있음

### nuxt-link 태그 소개

layouts/default.vue 수정

{% raw %}
```html
<template>
  <div>
    <header>
      <h1>{{ $route.name }} 페이지</h1>
      <NuxtLink to="/">홈페이지</NuxtLink>
      <NuxtLink to="/main">메인페이지</NuxtLink>
      <NuxtLink to="/product">상품페이지</NuxtLink>
    </header>
  </div>
</template>
...
```
{% endraw %}

<NuxtLink> 는 <router-link> 와 동일



## 참고

- [https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0](https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)