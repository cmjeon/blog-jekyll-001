---
layout: single
title: "Naver 서치어드바이저 활용하기"
categories:
  - "SEO"
---

## Naver 서치어드바이저 활용하기

Naver 에 서치어드바이저를 활용해서 사이트의 검색 최적화에 도움을 받을 수 있다.

먼저 사이트의 소유권을 확인해야 한다.

Naver 에서 제공하는 값을 docusaurus.config.js 의 metadata 에 등록한다.

```javascript
module.exports = {
  // ...
  themeConfig: {
    metadata: [{name: 'naver-site-verification', content: 'xxxxxxx'}],
    // ...
  },
};
```

네이버 서치어드바이저 들어온 김에 사이트 간단 체크를 해보았다.

[https://searchadvisor.naver.com/tools/sitecheck](https://searchadvisor.naver.com/tools/sitecheck)

### sitemap.xml 추가하기

[https://docusaurus.io/ko/docs/next/api/plugins/@docusaurus/plugin-sitemap](https://docusaurus.io/ko/docs/next/api/plugins/@docusaurus/plugin-sitemap)

플러그인으로 해결 할 수 있다.

플러그인을 설치한 후에는 config 파일을 수정한다.

```javascript
module.exports = {
  // ...
  presets: [
    // ...
    [
      '@docusaurus/preset-classic',
      {
        sitemap: {
          changefreq: 'weekly',
          priority: 0.5,
          ignorePatterns: ['/tags/**'],
          filename: 'sitemap.xml',
        },
      },
    ],
  ],
};
```

[https://www.xml-sitemaps.com/](https://www.xml-sitemaps.com/) 여기에서 만들기도 할 수 있다.

### docusaurus 태그 목록 확인하기

블로그 포스팅할 때 추가한 태그 목록들은 {{domain}}/blog/tags 에서 확인가능하다.
