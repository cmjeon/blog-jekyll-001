---
layout: single
title: "docusaurus v2 pwa 설정하기"
categories:
  - "docusaurus"
tags:
  - "PWA"
  - "정적사이트 생성기(SSG)"
---

## docusaurus v2 pwa 설정하기

docusaurus 에는 pwa 를 지원한다.

PWA 는 Progressive Web App, 웹페이지를 앱처럼 쓸 수 있게 해주는 것이다.

웹페이지를 앱처럼 쓸 수 있게 해주기 위해 제공되는 기능 중 괜찮은 것은 휴대폰의 홈에 아이콘을 만들어 준다는 것이다.

먼저, pwa 플러그인을 설치한다.

### plugin 설정하기

```shell
$ yarn add @docusaurus/plugin-pwa
```

docusaurus.config.js 에 플러그인 설정을 해준다.

```javascript
// ...
const config = {
  // ...
  plugins: [
    // ...
    [
      '@docusaurus/plugin-pwa',
      {
        debug: true,
        offlineModeActivationStrategies: ['appInstalled', 'standalone', 'queryString'],
        pwaHead: [
          {
            tagName: 'link',
            rel: 'icon',
            href: '/img/logo.png',
          },
          {
            tagName: 'link',
            rel: 'manifest',
            href: 'manifest.json', // your PWA manifest
          },
          {
            tagName: 'meta',
            name: 'theme-color',
            content: 'rgb(84, 104, 255)',
          },
          {
            tagName: 'meta',
            name: 'apple-mobile-web-app-capable',
            content: 'yes',
          },
          {
            tagName: 'meta',
            name: 'apple-mobile-web-app-status-bar-style',
            content: '#000',
          },
          {
            tagName: 'link',
            rel: 'apple-touch-icon',
            href: 'img/logo.png',
          },
          {
            tagName: 'link',
            rel: 'mask-icon',
            href: 'img/logo.svg',
            color: 'rgb(255, 255, 255)',
          },
          {
            tagName: 'meta',
            name: 'msapplication-TileImage',
            content: 'img/logo.png',
          },
          {
            tagName: 'meta',
            name: 'msapplication-TileColor',
            content: '#000',
          },
        ],
      },
    ],
  ],
};
```

### manifest.json 설정하기

그리고 static 폴더에 manifest.json 라는 파일이 필요하다.

```shell
$ touch static/manifest.json
```

```javascript
{
  "name": "Today I Learned",
  "short_name": "blog-cmjeon",
  "theme_color": "#5468ff",
  "background_color": "#424242",
  "display": "standalone",
  "scope": "./",
  "start_url": "./index.html",
  "related_applications": [
    {
      "platform": "webapp",
      "url": "https://blog-cmjeon.vercel.app/manifest.json"
    }
  ],
  "icons": [
    {
      "src": "img/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "img/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "img/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "img/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "img/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "img/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "img/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "img/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}

```

### icon 생성하기

사이즈별 파비콘을 만들어야 한다.

https://app.haikei.app 에서 적당한 이미지를 생성한 뒤에 사이즈별로 생성해준다.

### 배포하기

프로덕션에 배포하는 것 외에는 특별히 하는 것이 없다.

chrome 개발자도구 lighthouse 탭에서 확인해볼 수 있다.

### 참고

- [https://docusaurus.io/docs/api/plugins/@docusaurus/plugin-pwa](https://docusaurus.io/docs/api/plugins/@docusaurus/plugin-pwa)
- [https://younho9.dev/docusaurus-manage-docs-2](https://younho9.dev/docusaurus-manage-docs-2)
