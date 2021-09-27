---
layout: single
title:  nuxt.js 시작하기 - 004
categories: 
  - learn nuxtjs by joshua88
tags: 
  - vuejs
  - nuxtjs
---

## 쇼핑 상품 목록 페이지와 상세 페이지 개발

### 상품 목록 표시 기능 구현

pages/main.vue

```vue
<template>
  <div>
    <p>상품 페이지입니다</p>
    <div>
      <ul>
        <li v-for="product in products" :key="product.id">
          <img :src="product.imageUrl" :alt="product.name" />
          <p>{{ product.name }}</p>
          <p>{{ product.price }}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  async asyncData() {
    const response = await axios.get('http://localhost:3000/products')
    const products = response.data.map((item) => {
      return {
        ...item,
        imageUrl: `${item.imageUrl}?random=${Math.random()}`,
      }
    })
    return { products }
  },
  data() {
    return {
      products: [],
    }
  },
  async created() {},
}
</script>

<style>
</style>
```

template 에 상품을 보기좋게 표시하도록 ul 로 정리함

각 항목별로 다른 이미지가 나오도록 수정하기 위해 map 을 이용하여 상품데이터의 imageUrl 을 변형

### 라우팅 및 스타일링 정리

pages/main.vue 의 내용을 pages/index.vue 로 복사하고 main.vue 는 삭제

pages/main.vue 수정

```vue
<template>
  <div class="app">
    <main>
      <div>
        <input type="text" />
      </div>
      <ul>
        <li class="item flex" v-for="product in products" :key="product.id">
          <img
            class="product-image"
            :src="product.imageUrl"
            :alt="product.name"
          />
          <p>{{ product.name }}</p>
          <span>{{ product.price }}</span>
        </li>
      </ul>
    </main>
  </div>
</template>
...
<style scoped>
.flex {
  display: flex;
  justify-content: center;
}
.item {
  display: inline-block;
  width: 400px;
  height: 300px;
  text-align: center;
  margin: 0 0.5rem;
  cursor: pointer;
}
.product-image {
  width: 400px;
  height: 250px;
}
.app {
  position: relative;
}
.cart-wrapper {
  position: sticky;
  float: right;
  bottom: 50px;
  right: 50px;
}
.cart-wrapper .btn {s
  display: inline-block;
  height: 40px;
  font-size: 1rem;
  font-weight: 500;
}
</style>
```

layouts/default.vue 수정

```vue
<template>
  <div>
    <header>
      <h1>Nust Shopping</h1>
    </header>
    <Nuxt />
  </div>
</template>
...
```

### 상품 상세 페이지 구현을 위한 사전 작업

pages/index.js 의 상품리스트에 클릭이벤트, 메소드 추가

```vue
<template>
...
<ul>
  <li
    v-for="product in products"
    :key="product.id"
    class="item flex"
    @click="moveToDetailPage(product.id)"
  >
  ...
</template>
...
<script>
...
  methods: {
    moveToDetailPage(id) {
      console.log(id)
    },
  },
</script>
```

상품 li 선택 시 상품 id 가 콘솔에 표시됨

### Nuxt 의 동적 라우팅

pages/index.vue 수정

```vue
<script>
...
methods: {
  moveToDetailPage(id) {
    console.log(id)
    this.$router.push(`detail/${id}`)
  },
}
...
</script>
```

detail/${id} 페이지는 pages/detail/_id.vue 로 라우팅 할 수 있음

pages/detail/_id.vue 생성

```vue
<template>
  <div><h1>상세 페이지</h1></div>
</template>

<script>
export default {
  created() {
    console.log(this.$route.params)
  },
}
</script>

<style>
</style>
```

### 상품 상세 정보 조회 기능 개발

api 함수 모듈화 api 파일 생성

api/index.js 생성

```js
import axios from 'axios'

const instance = axios.create({
  baseURL: 'http://localhost:3000/',
})

function fetchProductById(id) {
  return instance.get(`/products/${id}`)
}

export { fetchProductById }
```

pages/detail/_id.vue 에 fetchProductById 를 이용하여 데이터 가져와서 표시

```vue
<template>
  <div>
    <h1>상세 페이지</h1>
    <div>
      <img :src="product.imageUrl" :alt="product.name" />
      <p>name : {{ product.name }}</p>
      <span>price : {{ product.price }}</span>
    </div>
  </div>
</template>

<script>
import { fetchProductById } from '@/api/index'

export default {
  async asyncData({ params }) {
    const response = await fetchProductById(params.id)
    const product = response.data
    return { product }
  },
}
</script>

<style>
</style>
```

### context 속성 안내

asyncData 의 context 속성

[https://joshua1988.github.io/vue-camp/nuxt/data-fetching.html#asyncData의 파라미터](https://joshua1988.github.io/vue-camp/nuxt/data-fetching.html#asyncdata%EC%9D%98-%ED%8C%8C%EB%9D%BC%EB%AF%B8%ED%84%B0)

[https://nuxtjs.org/docs/internals-glossary/context/](https://nuxtjs.org/docs/internals-glossary/context/)

### 상품 상세 페이지 레이아웃 정리 및 전역 스타일시트 설정

전역 스타일시트 파일의 위치는 nuxt.config.js 에서 설정가능함

nuxt.config.js 수정

```js
...
 // Global CSS: https://go.nuxtjs.dev/config-css
  css: [
    '@/assets/css/reset.css'
  ],
...
```

learn-nuxt 의 assets/css/reset.css 파일을 복사하여 프로젝트 폴더로 붙여넣기

[https://github.com/joshua1988/learn-nuxt/blob/master/assets/css/reset.css](https://github.com/joshua1988/learn-nuxt/blob/master/assets/css/reset.css)

상세페이지 레이아웃 정리

learn-nuxt 의 상세페이지 파일 참고하여 작성

```vue
// pages/detail/_id.vue
<template>
<div>
    <div class="container">
      <div class="main-panel">
        <img
          class="product-image"
          :src="product.imageUrl"
          :alt="product.name"
        />
      </div>
      <div class="side-panel">
        <p class="name">{{ product.name }}</p>
        <p class="price">{{ product.price }}</p>
        <button type="button" @click="addToCart">Add to Cart</button>
      </div>
    </div>
  </div>
</template>

<script>
import { fetchProductById } from '@/api/index'

export default {
  async asyncData({ params }) {
    const response = await fetchProductById(params.id)
    const product = response.data
    return { product }
  },
}
</script>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  margin: 2rem 0;
}
.product-image {
  width: 500px;
  height: 375px;
}
.side-panel {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 220px;
  text-align: center;
  padding: 0 1rem;
}
</style>
```

## 참고

- [https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0](https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)