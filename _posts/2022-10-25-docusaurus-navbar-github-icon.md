---
layout: single
title: "docusaurus navbar 에 github 아이콘 설정하기"
categories:
  - "docusaurus"
tags:
  - "PWA"
  - "정적사이트 생성기(SSG)"
---

## docusaurus navbar 에 github 아이콘 설정하기

상단 내비게이션바에서 github 링크를 만드려면 docusaurus.config.js 에서 설정을 하면 된다.

docusaurus.config.js 에서 navbar 설정을 한다.

```javascript
// ...
const config = {
  // ...
  themeConfig:
    /** @type {import('@docusaurus/preset-classic').ThemeConfig} */
    ({
      navbar: {
        title: 'Today I Learned',
        logo: {
          alt: 'Today I Learned Logo',
          src: 'img/logo.png',
          href: '/',
          // target: '_self',
        },
        items: [
          // ...
          {
            href: 'https://github.com/cmjeon',
            position: 'right',
            className: 'header-github-link',
            'aria-label': 'GitHub repository',
          },
        ],
      }
      // ...
    })
};

```

그런데 github 아이콘이 보이지 않았다.

docusaurus.io 페이지를 이리저리 살피다가 custom.css 를 알게되었고 거기서 github icon 관련 css 로 보이는 것을 찾을 수 있었다.

docusaurus.config.js 의 github 요소에 className 이 header-github-link 인데, custom.css 에 header-github-link 관련 요소가 있다.

얼른 가져다가 내 블로그의 src/css/custom.css 에 붙여넣기 했다.

```css
.header-github-link:hover {
  opacity: 0.6;
}

.header-github-link::before {
  content: '';
  width: 24px;
  height: 24px;
  display: flex;
  background: url("data:image/svg+xml,%3Csvg viewBox='0 0 24 24' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12'/%3E%3C/svg%3E")
  no-repeat;
}

[data-theme='dark'] .header-github-link::before {
  background: url("data:image/svg+xml,%3Csvg viewBox='0 0 24 24' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath fill='white' d='M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12'/%3E%3C/svg%3E")
  no-repeat;
}
```

이제 github 아이콘이 잘 보인다.
