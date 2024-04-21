---
layout: single
title: react native 기초 배우기 005 - Button, ScrollView, TextInput 심화, Picker
categories: 
  - learn react native
tags:
  - react native
---

## Button, ScrollView, TextInput 심화

- 버튼을 누르면 텍스트 입력창으로부터 값을 받아 하단의 목록으로 보여주는 App.js

  ~~~javascript
  import React, { Component } from 'react';
  import { TextInput, Button, View, StyleSheet, ScrollView } from 'react-native';

  class App extends Component {
    state = {
      maTextInput: '',
      alphabet: ['a','b','c','d']
    }

    onChangeInput = (event) => {
      this.setState({
        myTextInput: event
      })
    }

    onAddTextInput = () => {
      this.setState(prevState => {
        return {
          myTextInput: '',
          alphabet: [...prevState.alphabet, prevState.myTextInput]
        }
      }
    }

    render() {
      return (
        <View style={styles.mainView}>
        <TextInput
          value={this.state.myTextInput}
          style={styles.input}
          onChangeText={this.onChangeInput}
          multiline={true}
          maxLength={100}
          autoCapitalize={'none'}
          editable={true}
        >
        <Button
          title="Add Text Input"
          onPress={this.onAddTextInput}
        />
        {% raw %}
        <ScrollView style={{width: '100%'}}>
        {% endraw %}
          {
            this.state.alphabet.map((item, idx) => (
              <Text
                styles={styles.main}
                key={idx}
              >
                {item}
              </Text>
            ))
          }
      )
    }
  }

  const style = StyleSheet.create({
    mainView: {
      flex: 1,
      backgroundColor:'white',
      paddingTop: 50,
      alignItems: 'center',
    },
    subView: {
      backgroundColor: 'yellow',
      marginBottom: 10,
    }
    anotherSubView: {
      flex: 2,
      backgroundColor: 'yellow',
      marginBottom: 10,
      width: '100%',
      alignItems: 'center',
      justifyContent: 'center'
    },
    mainText: {
      fontSize: 20,
      fontWeight: 'normal',
      color: 'red',
      padding: 20,
      margin: 20,
      backgroundColor: 'pink'
    }
    input: {
      width: '100%',
      backgroundColor: '#cecece',
      marginTop: 20,
      fontSize: 25,
      padding: 10
    } 
  })
  ~~~

## Picker

- 콤보박스 같은 여러가지 옵션이 있고 그 중에 선택할 수 있는 컴포넌트

### picker 설치

#### picker 다운로드

~~~bash
# install picker
$ npm install @react-native-community/picker --save  
~~~

#### pod install

~~~bash
$ cd ios
# pod install
$ npx pod-install
$ cd ..
~~~

### picker 파일 생성

  ~~~bash
  # pwd = react_native_01
  $ mkdir src/picker.js
  ~~~

  ~~~javascript
  // picker.js

  import React, { Component } from 'react';
  import { View, Text, StyleSheet } from 'react-native';
  import { Picker } from '@react-native-community/picker';

  class PickerComponent extends Component {
    state = {
      country: 'korea'
    }
    render() {
      return (
        <View style={styles.container}>
          {% raw %}
          <Picker
            style={{height:50, width: 250)}}
            selectedValue={this.country}
            onValueChange={(val, idx) => 
              this.setState({country: val})
            }
          >
          {% endraw %}
            <Picker.Item label="Korea" value="korea">
            <Picker.Item label="Canada" value="canada">
            <Picker.Item label="China" value="china">
          </Picker>
        </View>
      )
    }
  }

  const styles = StyleSheet.create({
    container: {
      flex: 1,
      paddingTop: 20,
      marginBottom: 200,
      alignItems: 'center'
    }
  })

  export default PickerComponent
  ~~~

### App.js 에 추가

  ~~~javascript
  ...
  import { TextInput, Button, View, Text, StyleSheet, ScrollView} from 'react-native';
  import Picker from './src/picker';
  ...
  render() {
    return (
      <View style={styles.mainView}>
      <Picker/>
      ...
    )
  }
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnative.dev/docs/components-and-apis](https://reactnative.dev/docs/components-and-apis)
- [https://reactnative.dev/docs/scrollview](https://reactnative.dev/docs/scrollview)
- [https://reactnative.dev/docs/textinput](https://reactnative.dev/docs/textinput)
- [https://github.com/react-native-picker/picker](https://github.com/react-native-picker/picker)
