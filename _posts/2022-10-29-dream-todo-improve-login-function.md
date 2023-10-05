---
layout: single
title: "docusaurus 에서 draft 문서로 남기는 법"
categories:
  - "드림투두"
tags:
  - "SEO"
  - "정적사이트 생성기(SSG)"
---

## docusaurus 에서 draft 문서로 남기는 법

문서를 쓰다가 잠시 저장하고 싶을 때는 frontmatter 에 draft: true 라고 남기면 된다.

```markdown
---
date: 2022-10-29
title: '2022년 10월 29일'
authors: [cmjeon]
tags: ['docusaurus']
draft: true
---

// ...
```

이렇게 해두면 개발환경에서는 보이지만 운영환경에서는 보이지 않는 문서가 된다.

### docusaurus 생성 유튜브 영상

[https://www.youtube.com/watch?v=Sey7jymZU1Q](https://www.youtube.com/watch?v=Sey7jymZU1Q)

docusaurus 로 문서 작성하기 유튜브 영상 기록이다.

따라한 순서로 기록

아래 명령어로 바로 설치 가능하다.

```shell
$ npx create-docusaurus@latest my-website classic
```

node 16.14 버전 이상이 필요하다.

nvm 을 이용해서 설치해주자

실행하기

```shell
$ cd my-website
$ npx docusaurus start
```

실행하면 http://localhost:3000 으로 접속 가능

참고할만한 사이트 찾아보자
- [https://docusaurus.io/ko/showcase](https://docusaurus.io/ko/showcase)
- [https://github.com/mikro-orm/mikro-orm/tree/master/docs](https://github.com/mikro-orm/mikro-orm/tree/master/docs)
- [https://github.com/facebookincubator/profilo/tree/main/website](https://github.com/facebookincubator/profilo/tree/main/website)
- [https://github.com/livekit/livekit-docs](https://github.com/livekit/livekit-docs)
- [https://github.com/reduxjs/redux-toolkit](https://github.com/reduxjs/redux-toolkit)
- [https://github.com/realtime-apps-iap/realtime-apps-iap.github.io](https://github.com/realtime-apps-iap/realtime-apps-iap.github.io)

프리티어 적용

```json
{
  "singQuote":true,
  "semi": false,
}
```

docusaurus.config.js 의 themeConfig.navbar 가 상단바의 구성을 담당한다.

내 사이트의 navbar 참고

```javascript
const config = {
  // ...
  themeConfig: {
    // ...
    navbar: {
      title: 'Today I Learned',
      logo: {
        alt: 'Today I Learned Logo',
        src: 'img/logo-2.svg',
        href: '/',
        // target: '_self',
      },
      items: [
        {
          label: 'Posts',
          type: 'docSidebar',
          sidebarId: 'posts',
          position: 'left',
        },
        {
          label: '日日新 又日新',
          to: '/blog',
          position: 'left',
        },
        {
          label: 'RESUME',
          type: 'doc',
          docId: 'resume',
          position: 'left',
        },
        {
          href: 'https://github.com/cmjeon',
          position: 'right',
          className: 'header-github-link',
          'aria-label': 'GitHub repository',
        },
      ],
    },
  }
}

```

이 영상은 도큐사우루스 영상이기 보다는 문서작성 영상에 가까워 보인다.
