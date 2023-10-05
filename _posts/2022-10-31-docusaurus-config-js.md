---
layout: single
title: "docusaurus.config.js 설정"
categories:
  - "docusaurus"
tags:
  - "mermaid"
  - "정적사이트생성기(SSG)"
---

## docusaurus.config.js 설정

script 와 stylesheet 추가하는 방법

```javascript
module.exports = {
  // ...
  scripts: [
    {
      src: 'https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js',
      defer: true
    }
  ],
  stylesheets: [
    {
      href: "https://cdnjs.cloudflare.com/ajax/libs/mermaid/6.0.0/mermaid.css"
    }
  ],
}
```

기본 컬러모드를 설정, 스크롤 시 navbar 감추는 설정

```javascript
module.exports = {
  // ...
  themeConfig: {
    colorMode: {
      defaultMode: 'dark', // light, dark
    },
    navbar: {
      hideOnScroll: true,
      style: 'dark', // primary, dark
    }
    // ...
  }
  // ...
}
```

## mermaid.js 구동시켜보기

설명되어 있는 mermaid.js 를 구동하는 법은 꽤 간단하다.

```html
<html>
  <body>
    <script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
    <script>mermaid.initialize({ startOnLoad: true });</script>

    Here is one mermaid diagram:
    <div class="mermaid">
        graph TD
        A[Client] --> B[Load Balancer]
        B --> C[Server1]
        B --> D[Server2]
    </div>

    And here is another:
    <div class="mermaid">
        graph TD
        A[Client] -->|tcp_123| B
        B(Load Balancer)
        B -->|tcp_456| C[Server1]
        B -->|tcp_456| D[Server2]
    </div>
  </body>
</html>
```

문제가 되는 부분은 `mermaid.initialize({ startOnLoad: true });` 인데, 'mermaid' 라고 선언된 tag 영역을 읽어서 랜더링해주는 부분이다.
`
이 부분이 docusaurus.config.js 의 scripts 요소나 인라인으로 처리해봤더니 작동하지 않았다.

mermaid 를 구동할 방법을 찾아야 한다.

docusaurus.config.js 에 `ssrTemplate` 설정으로는 안된다.
