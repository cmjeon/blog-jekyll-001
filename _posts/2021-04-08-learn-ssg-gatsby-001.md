---
layout: single
title:  정적 사이트 생성기 gatsby 배우기 - 001
categories: 
  - learning ssg gatsby
tags: 
  - static site generator
  - gatsby
---

## 정적 사이트 생성기 gatsby 배우기 001

- 준비사항 
  - v12.13 버전 이상의 node.js
  - git
  - Visual Studio Code
  - Gatsby CLI

### Gatsby CLI 설치

```bash
$ npm install -g gatsby-cli
$ gatsby --version # 버전확인
```

### Gatsby site 생성

```bash
$ gatsby new
> What would you like to call your site? # my-gatsby-site
> What would you like to name the folder where your site will be created? # Enter
> Will you be using a CMS? # No
> Would you like to install a styling system? # No
> Would you like to install additional features with other plugins? # Build and host for free on Gatsby Cloud > Done > Yes

$ cd my-gatsby-site
$ npm run develop
```

### Gatsby site 접속하기

- Browser에서 `http://localhost:8000` 접속

### github 저장소 만들기

- Repository name : my-gatsby-site

### github 저장소에 Push

```bash
$ git remote add origin git@github.com:cmjeon/my-gatsby-site.git
$ git branch -M main
$ git push -u origin main
```

### Gatsby Cloud 

- https://www.gatsbyjs.com/dashboard

### gatsby cloud에서 사이트 생성하기

1. Add a site
1. Git provider : Github 선택
1. github 접근 허용
1. Repository : `Select destination` 선택
1. 새창이 뜨면 `Only select repositories` 선택
1. `my-gatsby-site` 등록
1. `Install` 선택
1. Gatsby Cloud 페이지에서 `Select a repository`에서 `my-gatsby-site` 선택
1. `Next` 선택
1. `Skip this step` 선택
1. `Create site` 선택

### 사이트 접속하기

- `HOSTED ON GATSBY CLOUD` 하단 URL 복사
- Browser에서 URL 접속

## 참고
- [https://www.gatsbyjs.com/docs/tutorial/](https://www.gatsbyjs.com/docs/tutorial/)