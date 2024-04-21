---
layout: single
title: react navigation 기초 배우기 001 - 소개 및 설치
categories: 
  - learn react navigation
tags:
  - react navigation
---

## React Navigation 소개 및 설치

- 강좌는 react navigation v5로 진행되었음
- 새로운 프로젝트 생성

  ~~~bash
  $ react-native init --version 0.61.5 react_navigation_01
  $ cd react_navigation_01
  ~~~

- react navigation 설치

  ~~~bash
  $ npm install react-native-reanimated react-native-gesture-handler react-native-screens react-native-safe-area-context @react-native-community/masked-view
  ~~~

- ios 개발자라면 pod 설치가 필요하다

  ~~~bash
  $ cd ios
  $ npx pod-install ios
  $ cd ..
  # pwd = react_navigation_01
  ~~~

- App.js 최상단에 등록

  ~~~javascript
  // App.js 

  import 'react-native-gesture-handler';
  import * as React from 'react';
  ...
  ~~~

- stack navigator library 설치

  ~~~bash
  $ npm install @react-navigation/stack
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnavigation.org/](https://reactnavigation.org/)
