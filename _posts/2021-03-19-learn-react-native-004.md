---
layout: single
title: react native 기초 배우기 004 - ScrollView, TextInput
categories: 
  - learn react native
tags:
  - react native
---

## ScrollView

- 화면을 스크롤할 수 있는 컴포넌트
- scrollView

  ~~~javascript
  ...
  import React, { Component } from 'react';
  import { View, Text, StyleSheet, ScrollView } from 'react-native';
  import Header form './src/header';
  ...
  <Generator add={this.onAddRandomNum}/>
  <ScrollView
    style={{width: '100%'}}
  >
    <NumList
      num={this.state.random}
      delete={this.onNumDelete}
    />
  </ScrollView>
  ...
  ~~~

### ScrollView의 Properties

#### onMomentumScrollBegin

- 스크롤이 움직이기 시작했을 때 트리거되는 함수

  ~~~javascript
  ...
  <ScrollView
    style={{width: '100%'}}
    onMomentumScrollBegin={()=>alert('begin')}
  >
  ...
  ~~~

#### onMomentumScrollEnd

- 스크롤의 움직임이 끝나면 트리거되는 함수

  ~~~javascript
  ...
  <ScrollView
    style={{width: '100%'}}
    onMomentumScrollEnd={()=>alert('end')}
  >
  ...
  ~~~

#### onScroll

- 스크롤이 움직이면 트리거되는 함수

  ~~~javascript
  ...
  <ScrollView
    style={{width: '100%'}}
    onScroll={()=>alert('Scrolling')}
  >
  ...
  ~~~

#### onContentSizeChange

- View의 크리가 바뀌면 트리거되는 함수

  ~~~javascript
  ...
  <ScrollView
    style={{width: '100%'}}
    onContentSizeChange={(width, height)=>alert(height)}
  >
  ...
  ~~~

#### bounces

- 스크롤이 바운스 되는 것을 허용/방지

  ~~~javascript
  ...
  <ScrollView
    style={{width: '100%'}}
    bounces={true}
  >
  ...
  ~~~

## TextInput

- 텍스트를 입력할 수 있는 컴포넌트
- src/input.js 생성

  ~~~javascript
  // input.js

  import React, { Component } from 'react';
  import { View, Text, StyleSheet, TextInput } from 'react-native';

  class Input extends Component {

    state = {
      myTextInput: 'text input'
    }

    onChangeInput = (event) => {
      this.setState({
        myTextInput: event
      })
    }

    render() {
      return (
        <View style={styles.mainView}>
          <TextInput
            value={this.state.myTextInput}
            style={styles.input}
            onChangeText={this.onChangeInput}
          />
        </View>
      )
    }
  }

  const styles = StyleSheet.create({
    mainView: {
      width: '100%'
    },
    input: {
      width: '100%',
      backgroundColor: '#cecece',
      marginTop: 20,
      fontSize: 25,
      padding: 10
    }
  })

  export default Input;
  ~~~

- App.js 에 추가

  ~~~javascript
  import React, { Component } from 'react';
  import { View, Text, }
  ~~~

### TextInput의 Properties

#### multiline

- 개행을 허용/방지

  ~~~javascript
  <TextInput
    multiline={true}
  >
  ~~~

#### maxLength

- 입력값의 길이를 제한

  ~~~javascript
  <TextInput
    maxLength={100}
  >
  ~~~

#### autoCapitalize

- 대문자 자동수정 허용/방지

  ~~~javascript
  <TextInput
    autoCapitalize={'none'}
  >
  ~~~

#### editable

- 입력을 허용/방지

  ~~~javascript
  <TextInput
    editable={true}
  >
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnative.dev/docs/components-and-apis](https://reactnative.dev/docs/components-and-apis)
- [https://reactnative.dev/docs/scrollview](https://reactnative.dev/docs/scrollview)
- [https://reactnative.dev/docs/textinput](https://reactnative.dev/docs/textinput)