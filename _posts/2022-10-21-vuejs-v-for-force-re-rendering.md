---
layout: single
title: "vue.js 에서 v-for force re-rendering 하기"
categories:
  - "vuejs"
tags:
  - "front-end"
---

## vue.js 에서 v-for force re-rendering 하기

v-for 로 만든 자식 컴포넌트에서 사용자가 입력한 값을 부모 컴포넌트에서 확인하였더니 값을 잘못 입력한 경우, 원래의 값을 다시 표시해 줄 수 있는 방법이다.

처음에는 컴포넌트에 초기값을 보관한 뒤에 reset 함수를 통해 다시 복구하는 방법을 생각했다.

이렇게 만들었더니 자식 컴포넌트에서 prop 으로 받은 값을 다른 곳에 복사해서 보관하도록 구현해야하고, 부모 컴포넌트가 어느 자식 컴포넌트의 reset 을 찾아야 하는지 등 문제가 좀 있었다.

그러던 중 :key 를 통해 손쉽게 컴포넌트를 re-rendering 할 수 있는 방법을 찾아서 기록해둔다.

위의 참고한 페이지의 예시를 일부 사용하였다.

```vue
<template>
  <q-list :key="renderKey">
    <div class="row no-wrap" v-bind:key="index" v-for="(item, index) in items">
      <q-formc 
          :item="item" 
          @changed="patchItem" 
          @save="saveChanges" 
          @reset="reset">
      </q-formc>
    </div>
  </q-list>
</template>
<scrip>
export default {
  data () {
    return {
      items: [],
      schema: {},
      renderKey: 0, // force re-rendering 시에 값을 변경한다.
    }
  },
  methods: {
    reset() {
      this.renderKey += 1
    },
    // ...
  }
}
</scrip>
```

q-list 에 :key 를 선언하고 renderKey 변수를 선언한다.

자식 컴포넌트에서 reset 을 호출하면 부모컴포넌트에서 renderKey 의 값을 변경한다.

q-list 가 갱신된다.

검색을 해보니 forceUpdate() 하는 방식도 있었는데, 위 방법이 더 마음에 든다.
