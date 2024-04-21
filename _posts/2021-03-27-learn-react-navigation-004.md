---
layout: single
title: react navigation 기초 배우기 004 - Drawer Navigator 1
categories: 
  - learn react navigation
tags:
  - react navigation
---

## [Drawer] 설치 및 화면 Linking

- 화면 좌측이나 우측에서 나오는 새로운 스크린
- 햄버거 메뉴나 쇼핑몰 상품목록의 필터로 쓰임
- drawer navigator library 설치

  ~~~bash
  $ npm install @react-navigation/drawer
  ~~~

- App.js 수정

  ~~~javascript
  ...
  import { createStackNavigator } from '@react-navigation/stack';
  import { createDrawerNavigator } from '@react-navigation/drawer';
  import DrawHomeScreen from './src/home_drawer';
  import DrawUserScreen from './src/user_drawer';
  const Stack = createStackNavigator();
  const Drawer = createDrawerNavigator();

  class App extends Component {
    render() {
      return (
        <NavigatoinContainer>
          <Drawer.Navigator>
            <Drawer.Screen name="Home" component={DrawerHomeScreen}>
            <Drawer.Screen name="User" component={DrawUserScreen}>
          </Drawer.Navigator>
        </NavigatoinContainer>
      )
    }
  }
  ~~~

- /src/home_drawer.js 생성

  ~~~javascript
  import 'react-native-gesture-handler';
  import React, { Component } from 'react';
  import { View, Text, Button } from 'react-native';

  class DrawHomeScreen extends Component {
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

- /src/user_drawer.js 생성

  ~~~javascript
  import 'react-native-gesture-handler';
  import React, { Component } from 'react';
  import { View, Text, Button } from 'react-native';

  class DrawerUserScreen extends Component {
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

- drawer navigator를 사용하면 header bar 가 사라짐

## [Drawer] Style 설정

- Drawer Navigator의 경우 모든 화면에서 여는 공통의 화면이므로 Style을 Drawer.Navigator 안에 설정하게 됨
- DrawerContent 을 이용해서 커스터마이징 가능

  ~~~javascript
  // App.js

  ...
  import { createStackNavigator } from '@react-navigatoin/stack';
  import {
    createDrawerNavigator,
    DrawerContentScrollView,
    DrawerItemList,
    DrawerItem
  } from '@react-navigatoin/drawer';
  ...
  CustomDrawerContent = (props) => {
    return (
      <DrawerContentScrollView {...props}>
        <DrawerItemList {...props}/>
        <DrawerItem
          label="Help"
          onPress={()=>Linking.openURL('http://www.google.com')}
        />
        <DrawerItem
          label="Info"
          onPress={()=>alert('Infor Windows')}
        />
    )
  }
  ...
  class App extends Component {
  ...
  <NavigationContainer>
    {% raw %}
    <Drawer.Navigator
      initialRouteName="Home"
      drawerType="front" // front slide permanent
      drawerPosition="right" // right left
      drawerStyle={{
        backgroundColor: '#c6cbef',
        width: 200
      }}
      drawerContentOptions={{
        activeTintColor: 'red', // 선택된 메뉴의 폰트 컬러
        activeBackgroundColor: 'skyblue' // 선택된 메뉴의 배경 컬러
      }}
      drawerContent={
        props => <CustomDrawerContent {...props} />
      }
    >
    {% endraw %}
      <Drawer.Screen name="Home" component={DrawerHomeScreen}/>
  </NavigationContainer>
  ...
  ~~~

- Drawer Navigator 에 이미지 삽입하는 방법
- 첫번째, DrawerContent 에 쓰이는 CustomDrawerContent를 활용

  ~~~javascript
  // App.js

  ...
  CustomDrawerContent = (props) => {
    return (
      <DrawerContentScrollView {...props}>
        <DrawerItemList {...props}/>
        <DrawerItem
          label="Help"
          onPress={()=>Linking.openURL('http://www.google.com')}
          icon={()=><LogoTitle/>} // LogoTitme 재사용
        />
        <DrawerItem
          label="Info"
          onPress={()=>alert('Infor Windows')}
        />
    )
  }
  ...
  ~~~

- 두번째, Drawer.Screen 의 options 프로퍼티로 선언

  ~~~javascript
  ...
  import DrawerUserScreen from './src/user_drawer';
  import PictogramHome from './src/assets/pics/home_icon.png';
  ...
  {% raw %}
  <Drawer.Screen
    name="Home"
    component={DrawerHomeScreen}
    options={{
      drawerIcon: ()=>(
        <Image
          source={PicktogramHome}
          style={{width: 40, height: 40}}
        >
      )
    }}
  >
  {% endraw %}
  ...
  ~~~

- 세번째, Drawer.Screen에서 사용되는 각 Component에서 선언

  ~~~javascript
  // user_drawer.js

  ...
  import { View, Text, Button, Image } from 'react-native';
  import Logo from './assets/pics/home_icon.png';
  
  class DrawerUserScreen extends Component {
    drawStyle = () => {
      this.props.navigation.setOptions({
        drawerIcon: () => (
          {% raw %}
          <Image
            source={Logo}
            style={{width: 40, height: 40}}
          >
          {% endraw %}
        )
      })
    }

    render() {
      this.drawerStyle();
      return (
        ...
      )
    }
  }
  ...
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnavigation.org/](https://reactnavigation.org/)
