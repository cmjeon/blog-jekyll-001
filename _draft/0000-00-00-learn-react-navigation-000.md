---
layout: single
title: react navigation 기초 배우기 000 - 
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

## [Stack] Header Bar 설정

- Header Bar의 title은 Screen의 name이 기본값임
- options 으로 속성 수정 가능
- App.js 수정

  ~~~javascript
  // App.js

  ...
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
      options={{
        title: 'Home Screen',
        headerStyle: {
          backgroundColor: 'pink'
        },
        headerTintColor: 'red',
        headerTitleStyle: {
          fontWeight: 'bold',
          color: 'purple'
        }
      }}
    />
    {% endraw %}
  </Stack.Navigator>
  ...
  ~~~

- props.navigation.setOptions 으로 수정가능
- /src/home.js 수정

  ~~~javascript
  ...
  <Button
    title="To User Screen"
    onPress() => {
      this.props.navigation.navigate('User', {
        userIdx: 100,
        userName: 'Gildong',
        userLastName: 'Hong'
      })
    }
  >
  <Button
    title="Change the title"
    onPress={()=>{
      this.props.navigation.setOptions({
        title:'Changed!',
        headerStyle: {
          backgroundColor: 'pink'
        },
        headerTintColor: 'red'
      })
    }}
  >
  </Button>
  ...
  ~~~

- style을 각 화면에서 선언하고 설정가능
- App.js 에 선언된 options 주석처리
- /src/user.js 수정

  ~~~javascript
  ...
  class UserScreen extends Component {
    headerStyle = () => {
      this.props.navigation.setOptions({
        title: 'Customizing',
        headerStyle: {
          backgroundColor: 'blue'
        },
        headerTintColor: 'yellow',
        headerTitleStyle: {
          fontWeight: 'bold',
          color: 'green'
        }
      })
    }
    render() {
      this.headerStyle();
    }
  }
  ~~~

## [Stack] 여러 화면에 공통 Style 적용

- App.js에 Stack.Navigator 의 screenOptions 로 공통 Style 적용 가능
- App.js 수정

  ~~~javascript
  ...
  {% raw %}
  <Stack.Navigator
    initialRouteName="Home"
    screenOptions={{
      headerStyle: {
        backgroundColor: '#a4511e'
      },
      headerTintColor: '#fff',
      headerTitleStyle: {
        fontWeight: 'bold',
        color: '#f3d612'
      }
    }}
  >
  {% endraw %}
  ...
  ~~~

- /src/user.js 에 선언된 headerStyle 주석처리

- Stack.Screen 의 options 로 화면별 Style 적용 가능

## [Stack] Header Bar 커스터마이징

- Header Bar 에 title 대신 그림 삽입
- home icon 등록
- /src/assets/pics에 파일 업로드(파일명 home_icon.png)
- App.js 수정

  ~~~javascript
  ...
  class App extends Component {
    logoTitle = () => {
      return (
        {% raw %}
        <Image
          style={{width: 40, height: 40}}
          source={require('./src/assets/pics/home_icon.png')}
        >
        {% endraw %}
      )
    }
    ...
    {% raw %}
    <Stack.Screen
      name="Home"
      component={HomeScreen}
      options={{
        title: 'Home Screen',
        headerTitle: <this.logoTitle/>
      }}
    >
    {% endraw %}
  }
  ~~~

- logoTitle을 별도의 파일로 분리
- /src/logo.js 생성

  ~~~javascript
  import React, { Component } from 'react';
  import { Image } from 'react-native';
  import Logo from './assets/pics/home_icon.png';

  class LogoTitle extends Component {
    render() {
      return (
        {% raw %}
        <Image
          style={{width:40, height:40}}
          source={Logo}
        >
        {% endraw %}
      )
    }
  }
  export default LogoTitle;
  ~~~

- App.js 수정

  ~~~javascript
  ...
  import UserScreen from './src/user';
  import Logo from './src/logo';
  ...
    {% raw %}
    <Stack.Screen
      name="Home"
      component={HomeScreen}
      options={{
        title: 'Home Screen',
        headerTitle: <LogoTitle/>
      }}
    />
    {% endraw %}
    ...
  ~~~

- header bar이 title은 logo 로 유지하지만 back button 은 text로 하고 싶다면 setOptions 을 수정
- /src/user.js 수정

  ~~~javascript
  ...
  headerStyle = () => {
    this.props.navigation.setOptions({
      ...
      headerBackTitle: 'BACK'
    })
  }
  ...
  ~~~

## [Stack] Header Bar 에 버튼 추가

- Home Screen에 버튼 추가
- App.js 수정

  ~~~javascript
  ...
  import React, { Component } from 'react';
  import { StyleSheet, View, Text, Image, Button } from 'react-native';
  import { NavigationContainer } from '@react-navigation/native';
  ...
    {% raw %}
    <Stack.Screen
      name="Home"
      component={HomeScreen}
      options={{
        ...
        headerRight: () => (
          <Button
            title="Info"
            onPress={()=>alert('I am a button!')}
          >
        )
      }}
    {% endraw %}
  ~~~

- User Screen에 버튼 추가
- /src/user.js 수정

  ~~~javascript
  ...
  headerStyle = () => {
    this.props.navigation.setOptions({
      ...
      headerBackTitle: 'BACK'
      headerRight: () => (
        <Button
          title="Go Back"
          onPress={() => {
            this.props.navigation.navigate('Home')
          }}
          color='orange'
        >
      )
    })
  }
  ...
  ~~~

## [Drawer] 설치 및 화면 Linking

## [Drawer] Style 설정

## [Drawer] Custom Component

## [Tab] 설치 및 화면 Linking

## React Native Vector Icons 설치

## React Native Vector Icons 활용

## Nesting Navigators

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnavigation.org/](https://reactnavigation.org/)