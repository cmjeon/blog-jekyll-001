---
layout: single
title: react native 기초 배우기 002 - Style, TouchEvent
categories: 
  - learn react native
tags:
  - react native
---

## Style

- inline 와 StyleSheet 을 사용하는 방법이 있음

### inline 방식

  ~~~javascript
  ...
  return (
    <View style=({
      backgroundColor: 'green',
      height: '100%',
      paddingTop: 50
    })>
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

### StyleSheet 방식

  ~~~javascript
  ...
  import { View, Text, StyleSheet } from 'react-native';
  ...
  return (
    <View style={styles.mainView}></View>
  ...
  const styles = StyleSheet.create({
    mainView: {
      flex: 1,
      backgroundColor: 'green',
      paddingTop: 50,
      alignItems: 'center',
      justifyContent: 'center',
    }
  })
  ...
  ~~~

### style 꾸며보기

  ~~~javascript
  ...
  return (
    <View style={styles.mainView}>
      <View style={styles.subView}>
        <Text style={styles.mainText}>hello world</Text>
      </View>
      <View style={styles.subView}>
        <Text>hello world</Text>
      </View>
      <View style={styles.anotherSubView}>
        <Text style={styles.mainText}>hello world</Text>
      </View>
    </View>
  )
  ...
  const styles = StyleSheet.create({
    mainView: {
      flex: 1,
      backgroundColor: 'green',
      paddingTop: 50,
      alignItems: 'center',
      justifyContent: 'center',
    },
    subView: {
      flex: 1,
      backgroundColor: 'yellow',
      marginBottom: 10,
      width: '50%'
    },
    anotherSubView: {
      flex: 2,
      backgroundColor: 'yellow',
      marginBottom: 10,
      width: '100%',
      alignItems: 'center',
      justifyContent: 'center'
    },
    mainText: {
      fontSize: 50,
      fontWeight: 'bold', 
      color: 'red',
      padding: 20
    }
  })
  ...
  ~~~

## TouchEvent

- TouchEvent 는 컴포넌트를 선택하면 발생하는 이벤트

  ~~~javascript
  ...
  return (
    <View style={styles.mainView}>
      <View style={styles.subView}>
        <Text style={styles.mainText}>hello world</Text>
      </View>
    </View>
  )
  ...
  const styles = StyleSheet.create({
    mainView: {
      flex: 1,
      backgroundColor: 'white', // green -> white
      paddingTop: 50,
      alignItems: 'center',
      justifyContent: 'center',
    },
    subView: {
      // flex: 1,
      backgroundColor: 'yellow',
      marginBottom: 10,
      // width: '50%'
    },
    anotherSubView: {
      flex: 2,
      backgroundColor: 'yellow',
      marginBottom: 10,
      width: '100%',
      alignItems: 'center',
      justifyContent: 'center'
    },
    mainText: {
      fontSize: 20, // 50 -> 20
      fontWeight: 'normal', // 'bold' -> 'normal'
      color: 'red',
      padding: 20
    }
  })
  ...
  ~~~

- header.js 파일생성

  ~~~bash
  # pwd = react_native_01
  $ mkdir src/header.js
  ~~~

  ~~~javascript
  // header.js

  import React from 'react';
  import { View, Text, StyleSheet } from 'react-native';

  const Header = () => (
    <View style={styles.header}>
      <Text>This is header</Text>
    </View>
  )

  const styles = StyleSheet.create({
    header: {
      backgroundColor: 'pink',
      alignItems: 'center',
      padding: 5,
      width: '100%'
    }
  })

  export default Header;
  ~~~

- App.js 수정

  ~~~javascript
  ...
  import { View, Text, StyleSheet } from 'react-native';
  import Header from './src/header';
  ...
      return (
        <View>
          <!-- header.js 의 내용이 표시 -->
          <Header/>
        </View>
      )
  ...
  ~~~

---

**NOTE**

### {} 와 () 의 차이
 
~~~javascript
eFunc = () => {
  // return 되는 JSX 컴포넌트가 없음
}

eFunc = () => (
  // JSX 컴포넌트를 return 할 수 있음
)
~~~
 
### JSX(javascript xml)
 
~~~javascript
const element = <h1>Hello, world!</h1>;
~~~

### 참고

- [https://ko.reactjs.org/docs/introducing-jsx.html](https://ko.reactjs.org/docs/introducing-jsx.html)

---

- state를 통해 header의 메시지 변경

  ~~~javascript
  // App.js

  ...
  class App extends Component {
    state = {
      appName: 'My First App'
    }
  ...
      <View style={styles.mainView}>
        <Header name={this.state.appName}/>
      </View>
  ...
  }
  ~~~

  ~~~javascript
  // header.js

  ...
  const Header = (props) => (
    <View style={styles.header}>
      <Text>{props.name}</Text>
    </View>
  )
  ...
  ~~~

- touchableOpacity 을 통해 touchEvent 생성

  ~~~javascript
  // header.js
  
  ...
  import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
  ...
  const Header = (props) => (
    <TouchableOpacity
      style={styles.header}
      // onPress 로 alert 띄우기
      onPress={()=alert('hello world')}
      // onLongPress 로 alert 띄우기
      onLongPress={()=alert('hello world')}
      // onPressIn 로 alert 띄우기
      onPressIn={()=alert('hello world')}
      // onPressOut 로 alert 띄우기
      onPressOut={()=alert('hello world')}
    >
      <View>
        <Text>{props.name}</Text>
      </View>
    </TouchableOpacity>
  )
  ...
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnative.dev/docs/components-and-apis](https://reactnative.dev/docs/components-and-apis)
- [https://reactnative.dev/docs/touchableopacity](https://reactnative.dev/docs/touchableopacity)