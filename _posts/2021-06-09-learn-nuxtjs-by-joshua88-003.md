---
layout: single
title:  nuxt.js 시작하기 - 003
categories: 
  - learn nuxtjs by joshua88
tags: 
  - vuejs
  - nuxtjs
---

## Nuxt.js 의 데이터 호출 방식과 API 연동

### 백엔드 API 서버 구성

https://github.com/joshua1988/learn-nuxt/tree/master/backend 접속

```bash
$ git clone https://github.com/joshua1988/learn-nuxt.git

$ cd learn-nuxt
```

backend 폴더를 진행중인 프로젝트 폴더에 복사

json 서버 실행

```bash
$ cd backend

$ npm i

$ npm run dev
```

localhost:3000 에서 접속

localhost:3000 에서 json 서버 접속 확인

### JSON Server 소개

https://jsonplaceholder.typicode.com/

https://github.com/typicode/json-server

json Server는 mock API 를 제공

구글에서 json server github 검색 > github repo 확인 시 파일기반의 REST API 서버를 확인할 수 있음

### axios 설치 및 API 호출

nuxtjs 를 중지한 후 json 서버가 3000 에 실행시키고 nuxtjs 를 재실행할 것

```bash
$ cd backend

$ npm run dev
# port 3000 에 json server 가 실행됨

$ cd ..

$ npm run dev
# 임의의 port 에 nuxtjs 가 실행됨 ex) 51542
```

axios 설치

```bash
$ npm i axios
```

pages/main.vue 에 axios 를 통해 호출

```js
...
<scirpt>
import axios from 'axios';

export default {
  async created() {
    const response = await axios.get('http://localhost:3000/products');
    console.log(response);
  }
}
</script>
```

개발자도구 console 창에 products 목록확인이 출력 

### 서버 포트 변경 및 받아온 데이터 표시

nuxtjs 호스트와 포트 변경

[https://nuxtjs.org/docs/features/configuration/#edit-host-and-port](https://nuxtjs.org/docs/features/configuration/#edit-host-and-port)

nuxt.config.js 파일에 포트를 지정

```js
{
  ...
  server : {
    port:5000,
  }
}
```

nuxtjs 재실행

```bash
$ npm run dev
```

nuxtjs 가 포트 5000 으로 실행되는 것을 확인

code - preference - keyboard shortcuts 로 설정

pages/main.vue 에 있는 console.log 로 보이는 products 를 화면에 표시

```vue
<template>
  <div>
    ...
    <div>
      {{ products }}
    </div>
  </div>
</template>
...
<script>
...
export default {
  data() {
    return {
      products: [],
    }
  },
  ...
}
</script>
```

HMR(Hot Module Replacement) 에 의해 화면이 변경되면서 표시됨

CSR 은 데이터가 표시되는 시간이 무조건 필요하고, 이는 네트워크 반응속도에 따라 콘텐츠가 느리게 표현될 수 있음

이를 SSR 인 nuxtjs 을 통해 해결할 수 있음

[https://joshua1988.github.io/webpack-guide/devtools/hot-module-replacement.html](https://joshua1988.github.io/webpack-guide/devtools/hot-module-replacement.html)

### Nuxt 데이터 호출 방식 안내

[https://joshua1988.github.io/vue-camp/nuxt/data-fetching.html](https://joshua1988.github.io/vue-camp/nuxt/data-fetching.html)

#### asyncData 속성

asyncData 속성은 페이지컴포넌트(pages 폴더 아래에 있는 컴포넌트)에만 제공되는 속성임

```vue
...
<script>
import axios from 'axios';

export default {
  // params의 id가 1이라고 가정
  async asyncData({ params, $http }) {
    const response = await axios.get(`/users/${params.id}`);
    const user = response.data;
    return { user }
  }
}
</script>
...
```

### asyncData 적용 및 ESLint 규칙 확인

asyncData 를 추가

eslint-plugin-vue 에 의해 asyncData 를 추가하면 export default 의 상단으로 이동됨

[https://eslint.vuejs.org/rules/order-in-components.html](https://eslint.vuejs.org/rules/order-in-components.html)

```vue
...
<script>
...
export default {
  async asyncData() {
    const response = await axios.get('http://localhost:3000/products');
    console.log(response);
    this.products = response.data;
  }
}
...
```

this.products 의 this 에서 오류가 발생

### asyncData 속성 안내 및 코드 수정

[https://joshua1988.github.io/vue-camp/es6/enhanced-object-literals.html#축약-문법-1-속성과-값이-같으면-1개만-기입](https://joshua1988.github.io/vue-camp/es6/enhanced-object-literals.html#축약-문법-1-속성과-값이-같으면-1개만-기입)

asyncData 의 예제코드를 보면 this 에서 오류가 발생하는 이유를 알 수 있음

```vue
...
<script>
export default {
  async asyncData({ params, $http }) {
    const response = await axios.get(`/users/${params.id}`);
    const user = response.data;
    return { user }
  }
}
</script>
...
```

asyncData 는 비동기데이터를 호출하는 인스턴스 속성임

asyncData 는 기본적으로 페이지를 진입하기 전에 호출하여 뷰 인스턴스 데이터 객체로 리턴함

따라서 asnycData 에 문제가 있으면 페이지가 표시되지 않음

### asyncData 속성 주의사항

```vue
<template>
  <div>
    <p>메인 페이지입니다</p>
    <ProductList><ProductList/>
  </div>
</template>
<script>
import axios from 'axios';
import ProductList form '~/components/ProductList.vue';
// ...
</script>
```

components/ProductList.vue 생성

```vue
<template>
  <div>
    {{ products }}
  </div>
</template>
<script>
// ...
export default {
  async asyncData() {
    const response = await axios.get('http://localhost:3000/products');
    console.log(response);
    return { product }
  }
}  
</script>
<style>
</style>
```

오류가 발생함. 이유는 ProductList.vue 가 페이지 컴포넌트가 아니기 때문임

### asyncData 속성 정리

asyncData 의 params, $http 는 무엇일까?

asyncData 는 페이지에 진입하기 전에 호출되고, return 으로 뷰 인스턴스 데이터 객체를 반환


## 참고

- [https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0](https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)