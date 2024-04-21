---
layout: single
title: react native 기초 배우기 001 - intro, View & Text
categories: 
  - learn react native
tags:
  - react native
---

## Intro

- 터미널 실행

  ~~~bash
  # 새 프로젝트 생성
  $ react-native init --version 0.61.5 react_native_01
  # 프로젝트로 이동
  $ cd react_native_01
  ~~~

- 시뮬레이터 실행
  ~~~bash
  $ react-native run-ios
  ~~~

- App.js의 내용이 화면에 출력됨. 어떻게?
- 앱의 입구인 index.js와 app.json 을 살펴보자

  ~~~javascript
  // index.js

  // react-native 으로부터 AppRegistry 을 사용
  import {AppRegistry} from'react-native';
  // App 으로부터 App 을 사용
  import App from './App';
  // app.json 으로부터 name 을 appName으로 사용
  import {name as appName} from './app.json';
  // appName 와 App을 AppRegistry에 등록
  AppRegistry.registerComponent(appName, () => App);
  ~~~

  ~~~javascript
  // app.json

  {
    "name": "react_native_01",
    "displayName": "react_native_01"
  }
  ~~~

- App.js를 알맞게 수정

  ~~~javascript
  import React, { Component } from 'react';
  import { View, Text } from 'react-native';
  class App extends Component {
    render() {
      return (
        <View>
          <Text>hello world</Text>
        </View>
      )
    }
  }

  export default App;
  ~~~

## View, Text

  ~~~javascript
  // react 모듈로부터 Component 클래스를 import
  import React, { Component } from 'react';
  // react-native 모듈로부터 View, Text 클래스를 improt  
  import { View, Text } from 'react-native';
  // Component 을 상속하는 App 클래스
  class App extends Component {
    render() {
      return (
        <View>
          <Text>hello world</Text>
        </View>
      )
    }
  }

  export default App;
  ~~~ 

- react-native를 통해 필요에 따라 다양한 클래스를 import 할 수 있음 (ex, Button, Image)

### View

- `<View>`는 다른 컴포넌트를 감싸는 컴포넌트
- UI를 만드는 기본적인 컴포넌트
- `<View>`안에 또 `<Veiw>`가 있을 수 있음
  
  ~~~javascript
  ...
  return (
    <View>
      <View>
        <Text>hello world</Text>
      </View>
      <View>
        <Text>hello world</Text>
      </View>
      <View>
        <Text>hello world</Text>
      </View>
    </View>
  )
  ...
  ~~~

### Text

- `<Text>`는 텍스트를 표시하는 컴포넌트

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnative.dev/docs/components-and-apis](https://reactnative.dev/docs/components-and-apis)
