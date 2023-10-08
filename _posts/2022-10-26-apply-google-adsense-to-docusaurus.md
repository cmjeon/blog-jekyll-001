---
layout: single
title: "docusaurus navbar 에 github 아이콘 설정하기"
categories:
  - "docusaurus"
tags:
  - "google adsense"
  - "analytics"
  - "logo color"
---

## docusaurus Google Adsense 적용하기

먼저 Google Adsense 에 접속해서 clientId 를 얻는다.

사이트를 등록하면 아래와 같은 스크립트를 제공해준다.

```html
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-000000000" crossorigin="anonymous"></script>
```

<!--truncate-->

스크립트의 `client` 값을 플러그인을 이용하여 적용할 수 있다.

```shell
$ yarn add -D docusaurus-plugin-google-adsense
```

[https://www.npmjs.com/package/docusaurus-plugin-google-adsense](https://www.npmjs.com/package/docusaurus-plugin-google-adsense)

docusaurus.config.js 파일을 수정한다.

```js {5-7,10} title="docusaurus.config.js"
module.exports = {
  // ...
  themeConfig: {
    // ...
    googleAdsense: {
      dataAdClient: 'ca-pub-000000000',
    },
  },
  // ...
  plugins: ['docusaurus-plugin-google-adsense'],
};
```

광고가 나오는지 일단 기다려보자

## docusaurus Google Analytics 적용하기

애널리틱스 사이트에서 Google Analytics 4 측정ID(`G-xxxxxxx`) 를 발급받는다.

플러그인을 설치한다.

```shell
$ yarn add @docusaurus/plugin-google-analytics
```

docusaurus.config.js 파일을 수정한다.

```js {7-10} title="docusaurus.config.js"
module.exports = {
  // ...
  presets: [
    [
      '@docusaurus/preset-classic',
      {
        googleAnalytics: {
          trackingID: 'UA-xxxxxxx',
          anonymizeIP: true,
        },
      },
    ],
  ],
};
```

Theme Configuration 의 Metadata 로도 등록이 가능하다고 한다.

[https://docusaurus.io/docs/api/themes/configuration#metadata](https://docusaurus.io/docs/api/themes/configuration#metadata)

잠시 후에 구글 애널리틱스 사이트에서 수집사항을 확인하면 된다.

꽤나 간단하다.

구글 애널리틱스에 등록을 해두면 [Google Search Console](https://search.google.com/search-console) 에서도 자동으로 인식된다.

## White 과 Black 에 잘 어울리는 컬러?

블로그 아이콘을 만드려는데 색상을 고르기가 어려웠다.

다크모드에 어울리는 색상은 화이트모드에서는 잘 보이지 않았다.

반대로도 마찬가지였다.

화이트모드, 다크모드 에 둘 다 잘 보이는 색상은 없는 것일까 궁금해졌다.

뜬금없지만 인테리어 사이트에서 어느정도 답을 찾을 수 있었다.

[https://www.hunker.com/13772448/colors-that-go-with-black-and-white](https://www.hunker.com/13772448/colors-that-go-with-black-and-white)

15가지 색상을 제안하고 있었다.

- Pink
- Light teal
- Lime green
- Brown
- Turquoise
- Gold
- Beige
- Sage green
- Hunter green
- Navy blue
- Yellow
- Tan
- Natural wood
- Orange
- Gray

나는 그 중에 오렌지를 선택하였다.

만족스럽다.
