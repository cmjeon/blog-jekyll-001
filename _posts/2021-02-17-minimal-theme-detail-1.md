---
layout: posts
title: minimal-mistakes theme 더 자세히 살펴보기 1
categories: 
  - jekyll
tags: 
  - jekyll
  - theme
  - minimal-mistakes
---
# Quick-Start Guide

## Installing the theme
- minimal mistakes는 gem-based theme임
- jekyll v3.7+이고 self-hosting을 한다면, theme를 ruby gem으로 설치할 수 있음
- `/docs`와 `/test`를 삭제해라
- _config.yml의 plugins에 jekyll-include-cache를 반드시 추가해야 함

## Gem-based method
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