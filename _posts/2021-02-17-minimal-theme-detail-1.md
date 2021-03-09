---
layout: single
title: jekyll minimal-mistakes theme 실행가이드
categories: 
  - jekyll
tags: 
  - jekyll
  - theme
  - minimal-mistakes
---

## Quick-Start Guide
- [Quick-Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/){:target="_blank"}
### Installing the theme
- minimal mistakes는 gem-based theme임
- jekyll v3.7+이고 self-hosting을 한다면, theme를 ruby gem으로 설치할 수 있음
- `/docs`와 `/test`를 삭제해라
- _config.yml의 plugins에 jekyll-include-cache를 반드시 추가해야 함

### Gem-based method
- Gem-based theme를 사용하면 assets, _layouts, _includes, _sass 같은 디렉토리가 gem 테마에 저장됨
- Gem-based theme 설치하기
  1. Gemfile에 추가하기
  ```yml
  gem "minimal-mistakes-jekyll"
  ```
  1. bundle 업데이트
  ```
  $ bundle
  ```
  1. _config.yml에 추가
  ```
  theme: minimal-mistakes-jekyll
  ```
  1. 실행
  ```
  $ bundle update
  ```

### Remote theme method
- Remote theme은 Gem-baased theme와 비슷하지만 Gemfile 변경이나 화이트리스트(?)가 필요하지 않음
- [Remote theme 설치하기](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#remote-theme-method){:target="_blank"}

### remove the Unnecessary
- 테마를 받은 후 불필요한 파일이나 디렉토리는 삭제해야 함
```
# 불필요한 파일, 디렉토리 목록
.editorconfig
.gitattributes
.github
/docs
/test
CHANGELOG.md
minimal-mistakes-jekyll.gemspec
README.md
screenshot-layouts.png
screenshot.png
```
