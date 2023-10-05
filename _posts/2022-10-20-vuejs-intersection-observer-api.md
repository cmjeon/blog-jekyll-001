---
layout: single
title: "vue.js 에서 Intersection Observer API 이용하기"
categories:
  - "vuejs"
tags:
  - "front-end"
---

## vue.js 에서 Intersection Observer API 이용하기

[https://velog.io/@kbpark9898/Vue-intersection-observer%EB%A1%9C-%EC%8A%A4%ED%81%AC%EB%A1%A4-%ED%83%90%EC%A7%80%ED%95%98%EA%B8%B0](https://velog.io/@kbpark9898/Vue-intersection-observer%EB%A1%9C-%EC%8A%A4%ED%81%AC%EB%A1%A4-%ED%83%90%EC%A7%80%ED%95%98%EA%B8%B0)

vue.js 에서 Intersection Observer API 이용해서 pagination 처리를 해보겠습니다.

화면 스크롤이 하단까지 내려오는 것을 감지해서 새로운 게시물 목록을 조회해서 붙여주는 방식입니다.

Container.vue 와 Observer.vue 를 이용해서 쉽게 만들 수 있습니다.

```javascript
// Container.vue
<template>
  <div class="hello">
    <h1>게시물 목록</h1>
    <ul>
      <li class="box" v-for="item in myList" v-bind:key="item.no">
        <span>{{ item.no }} || {{ item.name }}</span>
      </li>
    </ul>
    <Observer v-on:nextPage="getNextPage"></Observer>
  </div>
</template>

<script>
import list from "../list.js";
import Observer from "./Observer";

export default {
  name: "HelloWorld",
  components: {
    Observer,
  },
  props: {},
  data() {
    return {
      index: 0,
      perPage: 3,
      myList: [],
      originList: list,
    };
  },
  mounted() {
    this.myList = this.originList.splice(this.index, this.perPage);
  },
  methods: {
    getNextPage() {
      this.myList = this.myList.concat(
        this.originList.splice(this.index, this.perPage)
      );
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.box {
  padding: 100px;
  list-style: none;
  border: solid #000 1px;
}
</style>
```

---

```javascript
// Observer.vue
<template>
  <div class="aa" ref="trigger">_</div>
</template>
<script>
export default {
  data() {
    return {
      observer: {},
    };
  },
  methods: {
    handleIntersect(target) {
      if (target.isIntersecting) {
        this.$emit("nextPage");
      }
    },
  },
  mounted() {
    const options = {
      root: null,
      threshold: 1,
    };
    this.observer = new IntersectionObserver((entries) => {
      this.handleIntersect(entries[0]);
    }, options);
    
    this.observer.observe(this.$refs.trigger);
  },
};
</script>

<style>
.aa {
  background-color: red;
}
</style>
```

Observer 컴포넌트가 화면에 보일 때를 감지하여 myList 에 항목을 추가해줍니다.

위 예제를 Sandbox 에서 구현해보았습니다.

[https://yw3vf3.csb.app/](https://yw3vf3.csb.app/)
