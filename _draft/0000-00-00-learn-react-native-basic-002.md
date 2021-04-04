---
layout: single
title: react native 기초 배우기 002
categories: 
  - learning
tags: 
  - react native basic
---

## 새 프로젝트 생성

- Visual Studio 실행
- View > Terminal 실행

~~~bash
# 새 프로젝트 생성
$ react-native init --ver sion 0.61.5 my_first_app
# 폴더 이동
$ cd my_first_app
~~~

> - react-native 가 실행되지 않는다면?
> - 설정을 bash_profile에 했는데, vscode에서 zsh bash가 오픈되서 생긴 문제   
> - 해결방법   
>> ~~~bash
>> $ source ~/.bash_profile
>> ~~~

- File > Add Foler to Workspace > 폴더 선택 > Add

## iOS Simulator 구동

~~~bash
$ npm start
~~~

- 새 터미널
~~~bash
$ react-native run-ios
~~~

~~~bash
# 다른 iPhone에서 실행
$ react-native run-ios --simulator="iPhone 8 Plus"
~~~

- App.js에서 생성된 코드에 대해서 iOS Simu
- command + r : 새로고침
- command + d : 시뮬레이터 메뉴열기


## Android Simulator 구동

- ADV Manager에서 Android Simulator 실행

~~~bash
$ react-native run-android
~~~

- rr : 새로고침
- command + n : 메뉴열기
