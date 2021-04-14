---
layout: single
title: react native 기초 배우기 003 - Button
categories: 
  - learn react native
tags:
  - react native
---

## Button

- generator.js 파일생성

  ~~~bash
  # pwd = react_native_01
  $ mkdir src/generator.js
  ~~~

  ~~~javascript
  // generator.js

  import React from 'react';
  import { View, Text, StyleSheet, Button } from 'react-native';

  const Generator = () => {
    return (
      <View>
        <Button
          title="Add Number" // title : 필수 property
        />
      </View>
    )
  }

  const styles = StyleSheet.create({
    
  })

  export default Generator;
  ~~~

- App.js 에서 Generator 추가

  ~~~javascript
  ...
  import Header from './src/header';
  import Generator from './src/generator';
  ...
      return (
        <View>
          <Header name={this.state.appName}/>
          <View>
            <Text
              style={styles.mainText}
              onPress={()=>alert('text touch event')}
            >Hello World</Text>
          </View>
          <Generator/>
        </View>
      )
  ...
  ~~~

- ios, aos은 기본 버튼 모양이 다르므로, 스타일 정의가 필요

- generator.js 스타일 정의

  ~~~javascript
  // generator.js

  ...
    return (
      <View style={style.generator}>
        <Button
          title="Add Number"
          onPress={()=>alert('Button Pressed!!!')}
        />
  ...
  const styles = StyleSheet.create({
    generator: {
      backgroundColor: '#D197CF',
      alignItme: 'center',
      padding: 5,
      width: '100%'
    }
  })
  ...
  ~~~

- App.js 에 변수 추가

  ~~~javascript
  // App.js

  ...
    state = {
      appName: 'My First App',
      random: [36, 999]
    }

    onAddRandomNum = () => {
      alert('add random number!!!')
    }
  ...
        </View>
        <Generator add={this.onAddRandomNum}/>
      </View>
  ...
  ~~~

- generator에 props, props.add() 추가

  ~~~javascript
  // generator.js

  ...
  const Generator = (props) => {
    return (
  ...
          title="Add Number"
          onPress={()=>props.add()}
        />
  ...
  ~~~

- numlist.js 파일생성

  ~~~bash
  # pwd = react_native_01
  $ mkdir src/numlist.js
  ~~~

  ~~~javascript
  // numlist.js

  import React from 'react';
  import { View, Text, StyleSheet } from 'react-native';

  const NumList = (props) => {
    return (
      <View>
        <Text>Hello Again~~</Text>
      </View>
    )
  }

  const styles = StyleSheet.create({
    numList: {
      backgroundColor: '#cecece',
      alginItems: 'center',
      padding: 5,
      width: '100%',
      marginTop: 5
    }
  })

  export default NumList;
  ~~~

- App.js 에서 NumList 추가

  ~~~javascript
  ...
  import Generator from './src/generator';
  import NumList from './src/numlist';
  ...
      return (
        <View>
          <Header name={this.state.appName}/>
  ...
          <Generator add={this.onAddRandomNum}/>
          <NumList num={this.state.random}/>
        </View>
      )
  ...
  ~~~

- numlist에 props 추가

  ~~~javascript
  // numlist.js

  ...
  const NumLisst = (props) => {
    return (
      props.num.map((item, idx)=>(
        <View style={styles.numList} key={idx}>
          <Text>{item}</Text>
        </View>
      ))
  ...
  ~~~

- app.js 에서 style 수정

  ~~~javascript
  ...
    const style = StyleSheet.create({
      mainView: {
        ...
        // justifyContent: 'top' 삭제
      }
    })
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnative.dev/docs/components-and-apis](https://reactnative.dev/docs/components-and-apis)