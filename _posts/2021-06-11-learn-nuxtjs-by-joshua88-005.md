---
layout: single
title:  nuxt.js 시작하기 - 005
categories: 
  - learn nuxtjs by joshua88
tags: 
  - vuejs
  - nuxtjs
---

## 쇼핑 상품 검색 UI 개발

### 검색 UI 컴포넌트화

components/SearchInput.vue 생성

```vue
<template>
  <div>
    <input type="text" />
    <button>Search</button>
  </div>
</template>

<script>
export default {}
</script>

<style>
</style>
```

pages/index.vue 에 SearchInput Component 추가

```vue
<template>
  <div class="app">
    <main>
      <SearchInput></SearchInput>
      <ul>
        ...
</template>
<script>
...
import SearchInput from '~/components/SearchInput.vue'

export default {
  component: {
    SearchInput
  }
  ...
}
</script>
...
```

nuxtjs 에서 '~/` 은 '@/' 같은 뜻

### 검색 UI 컴포넌트 데이터, 이벤트 처리

components/SearchInput.vue 수정

```vue
<template>
  <div>
    <input
      type="text"
      :value="searchKeyword"
      @input="$emit('input', $event.target.value)"
    />
    <button>Search</button>
  </div>
</template>

<script>
export default {
  props: {
    searchKeyword: {
      type: String,
      default: () => '',
    },
  },
}
</script>
...
```

'vpr' 로 props 기본형식 생성가능

[https://marketplace.visualstudio.com/items?itemName=sdras.vue-vscode-snippets](https://marketplace.visualstudio.com/items?itemName=sdras.vue-vscode-snippets)

pages/index.vue 의 SearchInput 수정

```vue
<template>
  <div class="app">
    <main>
      <!-- @ 를 이용한 방법
        <SearchInput
          :search-keyword="searchKeyword"
          @input="updateSearchKeyword"
        ></SearchInput>
      @ 를 이용한 방법 -->

      <!-- v-model 을 이용한 방법 -->
      <SearchInput
        v-model="searchKeyword"
        :search-keyword="searchKeyword"
      ></SearchInput>
      <!-- v-model 을 이용한 방법 -->
      ...
    </main>
  </div>
</template>

<script>
...

export default {
  ...
  methods: {
    moveToDetailPage(id) {
      this.$router.push(`detail/${id}`)
    },
    /* @ 를 이용한 방법
    updateSearchKeyword(keyword) {
      this.searchKeyword = keyword
    },
    @ 를 이용한 방법 */
  },
}
</script>
...
```

### 검색을 위힌 API 함수 설계 및 구현

components/SearchInput.vue 수정

```vue
<template>
  <div>
    <input
      type="text"
      :value="searchKeyword"
      @input="$emit('input', $event.target.value)"
    />
    <button type="button" @click="$emit('search')">Search</button>
  </div>
</template>
...
```

이벤트에서 바로 $emit 을 호출하는 것 보다 methods 를 호출하여 $emit 을 호출하는 방식이 더 테스터에 용이한 방식임

json server 백엔드의 operators 를 이용하여 검색함수 생성

[https://github.com/typicode/json-server#operators](https://github.com/typicode/json-server#operators)

api/index.js 수정

```js
...
function fetchProductsByKeyword(keyword) {
  return instance.get(`/products`, {
    params: {
      name_like: keyword,
    }
  })
}

export { fetchProductById, fetchProductsByKeyword }
```

pages/index.js 에서 searchProduct 함수로 데이터 조회

```vue
<template>
  <div class="app">
    <main>
      <!-- ... -->
      <SearchInput
        v-model="searchKeyword"
        :search-keyword="searchKeyword"
        @search="searchProducts"
      ></SearchInput>
      <!-- ... -->
    </main>
  </div>
</template>

<script>
...

export default {
  // ...
  methods: {
    // ...
    async searchProducts() {
      const response = await fetchProductsByKeyword(this.searchKeyword)
      console.log(response)
    },
  },
}
</script>
<style scoped>
/* ... */
</style>
```

### 검색 기능 마무리

pages/index.js 수정

```vue
<template>
  <div class="app">
    <main>
      <!-- ... -->
      <SearchInput
        v-model="searchKeyword"
        :search-keyword="searchKeyword"
        @search="searchProducts"
      ></SearchInput>
      <!-- ... -->
    </main>
  </div>
</template>

<script>
...

export default {
  // ...
  methods: {
    // ...
    async searchProducts() {
      const response = await fetchProductsByKeyword(this.searchKeyword)
      console.log(response.data)
      this.products = response.data.map((item) => {
        return {
          ...item,
          imageUrl: `${item.imageUrl}?random=${Math.random()}`,
        }
      })
    },
  },
}
</script>
<style scoped>
/* ... */
</style>
```

### 헤더와 검색 UI 스타일 정리

components/SearchInput.vue, components/AppHeader.vue 를 참고하여 스타일 정리

[components/SearchInput.vue](https://github.com/joshua1988/learn-nuxt/blob/master/components/SearchInput.vue)

[components/AppHeader.vue](https://github.com/joshua1988/learn-nuxt/blob/master/components/AppHeader.vue)

## 참고

- [https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0](https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)
