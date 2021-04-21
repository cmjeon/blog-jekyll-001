---
layout: single
title: react navigation 기초 배우기 002 - Stack Navigator 1
categories: 
  - learn react navigation
tags:
  - react navigation
---

## [Stack] 화면 Linking

- App.js 수정

  ~~~javascript
  import 'react-native-gesture-handler';
  import React, { Component } from 'react';
  import { StyleSheet, View, Text } from 'react-native';
  import { NavigationContainer } from '@react-navigation/native';
  import { createStackNavigator } from '@react-navigation/stack';
  import { HomeScreen } from './src/home';
  import { UserScreen } from './src/user';

  const Stack = createStackNavigator();

  class App extends Component {
    render() {
      return (
        <NavigationContainer>
          <Stack.Navigatior>
            <Stack.Screen name="Home" component="HomeScreen"/>
            <Stack.Screen name="User" component="UserScreen"/>
          </Stack.Navigator>
        </NavigationContainer>
      )
    }
  }

  const styles = StyleSheet.create({

  });
  export default App;
  ~~~

- /src/home.js 생성

  ~~~javascript
  import 'react-native-gesture-handler';
  import React, { Component } from 'react';
  import { View, Text, Button } from 'react-native';

  class HomeScreen extends Component {
    render() {
      return (
        {% raw %}
        <View style={{
          flex: 1,
          alignItems: 'center',
          justifyContent: 'center'
        }}>
        {% endraw %}
          <Text>Home Screen</Text>
          <Button
            title="To User Screen"
            onPress{()=>{
              this.props.navigation.navigate("User")
            }}
          >
        </View>
      )
    }
  }
  ~~~

- /src/user.js 생성

  ~~~javascript
  import 'react-native-gesture-handler';
  import React, { Component } from 'react';
  import { View, Text, Button } from 'react-native';

  class UserScreen extends Component {
    render() {
      return (
        {% raw %}
        <View style={{
          flex: 1,
          alignItems: 'center',
          justifyContent: 'center'
        }}>
        {% endraw %}
          <Text>User Screen</Text>
          <Button
            title="To Home Screen"
            onPress{()=>{
              this.props.navigation.navigate("Home")
            }}
          >
        </View>
      )
    }
  }
  ~~~

## [Stack] Navigation Params

- initialRouteName 으로 초기화면 설정 가능
- initialParams 으로 파라메터 초기화 가능
- App.js 수정

  ~~~javascript
  // App.js
  ...
  <NavigationContainer>
    <Stack.Navigator initialRouteName="User">
      {% raw %}
      <Stack.Screen 
        name="Home"
        component="HomeScreen"
        initialParams={{
          userIdx: 50,
          userName: 'Gildong',
          userLastName: 'Go'
        }}
      />
      {% endraw %}
      ...
  ~~~

- navigate로 파라메터 전달 가능
- /src/home.js 수정

  ~~~javascript
  // /src/home.js

  ...
  <Text>Home Screen</Text>
  <Button
    title="To User Screen"
    onPress() => {
      this.props.navigation.navigate('User', {
        userIdx: 100,
        userName: 'Gildong',
        userLastName: 'Hong'
      })
    }
    ...
  >
  
  ~~~

- this.props.route 으로 파라메터 수신
- /src/user.js 수정
  
  ~~~javascript
  // /src/user.js 
  ...
  render() {
    const { params } = this.props.route;
    const userIdx = params ? params.userIdx : null;
    const userName = params ? params.userName : null;
    const userLastName = params ? params.userLastName : null;
    return (
    ...
      </Button>
      <Text>User Idx: {JSON.stringfy(userIdx)}</Text>
      <Text>User Name: {JSON.stringfy(userName)}</Text>
      <Text>User LastName: {JSON.stringfy(userLastName)}</Text>
    </View>
      ...
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnavigation.org/](https://reactnavigation.org/)