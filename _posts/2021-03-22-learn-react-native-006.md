---
layout: single
title: react native 기초 배우기 006 - Slider, ActivityIndicator
categories: 
  - learn react native
tags:
  - react native
---

## Slider

- 가로선이 있고 좌우로 값을 바꿀 수 있는 컴포넌트

### Slider 설치

#### Slider 다운로드

~~~bash
# install slider
$ npm install @react-native-community/slider --save  
~~~

#### pod install

~~~bash
$ cd ios
# pod install
$ npx pod-install
$ cd ..
~~~

#### picker 파일 수정

~~~javascript
...
import { Picker } from '@react-native-community/picker';
import Slider from '@react-native-community/slider';
...
  state = {
    country: 'korea',
    value: 50
  }
  sliderValueChange=(value) => {
    this.setState({
      value: value
    })
  }
  ...
  render() {
    return (
      <View style={styles.container}>
        {% raw %}
        <Slider
          style={{height:40, width:300}}
          value={this.state.value}
          mininumValue={0}
          maximunValue={100}
          onValueChange={this.sliderValueChange}
          maximumTrackTintColor='red'
          minimumTrackTintColor='blue'
          step={10}
        />
        {% endraw %}
        <Text
          style={styles.input}
        >
        {this.staet.value}
        </Text>
        ...

const style = StyleSheet.create({
  ...
  input:{
    width:'100%'
  }
})
~~~

## ActivityIndicator

- 화면전환 시 표시되는 화면

- picker 파일 수정

  ~~~javascript
  import React, { Component } from 'react';
  import { View, Text, StyleSheet, ActivityIndicator } from 'react-native';
  import { Picker } from '@react-native-community/picker';
  ...
    render() {
      return (
        <View style={styles.container}>
          ...
          <Text
            style={styles.input}
          >{this.state.value}</Text>
          {% raw %}
          <ActivityIndicator
            style={{paddingTop: 200}}
            size="large"
            color="green"
            animating={true} // false 로 변경
          />
          {% endraw %}
          <Picker>
          ...
      )
    }
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnative.dev/docs/components-and-apis](https://reactnative.dev/docs/components-and-apis)
- [https://github.com/callstack/react-native-slider](https://github.com/callstack/react-native-slider)
- [https://reactnative.dev/docs/activityindicator](https://reactnative.dev/docs/activityindicator)