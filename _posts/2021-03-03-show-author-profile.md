---
layout: single
title: minimal-mistakes에서 저자 프로필을 보이게 하기
categories: 
  - jekyll
tags: 
  - jekyll
  - minimal-mistakes
  - author profile
---

## 저자 프로필
- 지킬에서 좌측 사이드바에 보이는 내용은 저자 프로필(author_profile)이다.

## 해결
- _config.yml에서 defaults 설정으로 보이게 할 수 있다.
- 또한 각 포스트나 페이지에서 author_profile 설정으로 정의할 수 있다.

```
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true # 읽는 시간
      comments: # true
      share: true
      related: true
      show_date: true # 발행 일자 표시
  # _pages
  - scope:
      path: ""
      type: pages
    values:
      layout: single
      author_profile: true
```

## 참고
- https://devinlife.com/howto%20github%20pages/blog-config/#9-_posts-_pages-기본-설정