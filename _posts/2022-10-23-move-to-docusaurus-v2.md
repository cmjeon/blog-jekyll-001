---
layout: single
title: "docusaurus v2 로 블로그 이전"
categories:
  - "docusaurus"
tags:
  - "정적사이트 생성기(SSG)"
---

## docusaurus v2 로 블로그 이전

원래 github.io 에서 jekyll 을 사용해서 블로그를 쓰고 있었다.

jekyll 은 간편하게 개인 블로그 생성, 내용을 작성할 수 있었고, github.io 에 배포되는 등 특별히 불편함을 찾을 수 없는 훌륭한 도구었다.

하지만 jekyll 은 나에게는 어색한 ruby 라는 언어로 만들어져 있었기 때문에...

나는 javascript 로 만들어진 블로그 도구를 찾으려고 하게 되었다.  (지금 생각해보면 왜였을까)

그래서 javascript 로 만들어진 SSG 들 중에 어떤 것들이 있는지 찾아보게 되었다.

vuepress, gatsby, docusaurus 들이 좀 유명했는데 이것들의 tutorial 을 따라해보았다.

제일 먼저 gatsby 의 문서를 보면서 tutorial 을 따라해보았다.

gatsby 는 일단 jekyll 보다 훨씬 강력하고, 내가 이해하기엔 복잡한 SSG 였다.

수많은 플러그인, 테마들, react component 들..

vuepress 는 적절하게 템플릿화되어 있어서 마치 jekyll 과 비슷한 수준으로 복잡하였다.

괜찮은 듯 보였는데, 쓰다보니까 태그기능 등 jekyll 에서 당연히 있었던 기능들이 없었다.

v2 베타버전이어서 그런거였을까?

그래서 최근에 v2 버전을 발표한 docusaurus 로 다시 방향을 잡았다.

docusaurus 은 javascript 로 만들어진 jekyll 만큼 간단한 SSG 로 보였다.

튜토리얼 좀 해보고 맘에들어서 얼른 이사하였다.

그리고 배포채널도 좀 바꿔보았다.

docusaurus 도 github.io 으로 잘 배포되었지만, 왠지 요즘 유행이라는 vercel 에 배포해보고 싶었다.

그래서...

새로운 블로그는 https://blog-cmjeon.vercel.app/blog 가 되었다.

이후로는 잔잔한 수정을 하면서 과거에 썼던 블로그들을 옮겨 올 생각이다.

docusaurus 에 블로그(blog) 작성 시 쓸 수 있는 front matter 정보를 여기에 기록해둔다.

https://docusaurus.io/docs/api/plugins/@docusaurus/plugin-content-blog#markdown-front-matter

문서(docs) 작성 시 쓸 수 있는 front matter 도 따로 있다.

https://docusaurus.io/docs/api/plugins/@docusaurus/plugin-content-docs#markdown-front-matter

이제 매일 새롭게 알게 된 것을 꼭 기록해서 하루하루 쬐끔이라도 나아지는 개발자가 되자.
