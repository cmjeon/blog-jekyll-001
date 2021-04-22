---
layout: single
title: react navigation 기초 배우기 005 - Drawer Navigator 2
categories: 
  - learn react navigation
tags:
  - react navigation
---

## [Drawer] Custom Component

- /src/my_drawer.js 생성

  ~~~javascript
  import React, { Component } from 'react';
  import { StyleSheet, ScrollView, Image, View, Text, Button } from 'react-native';
  import Logo from './assets/pics/home_icon.png';
  import { CommonActions } from '@react-navigation/native';

  class SideDrawer extends Component {
    navigateToScreen = (route) => () => {
      this.props.navigation.dispatch(
        CommonActions.navigate({
          name: route,
          params: {},
        })
      )
    }
    render() {
      return (
        <View style={styles.container}>
          <ScrollView>
            <View>
              <View style={styles.imageContainer}>
                {% raw %}
                <Image
                  source={ Logo }
                  style={{ width: 40, height: 40 }}
                />
                {% endraw %}
              </View>
              <Text style={styles.sectionHeading}>Section 1</Text>
              <View style={styles.navSectionStyle}>
                <Text
                  style={styles.navItemStyle}
                  onPress={this.navigateToScreen('Home')}
                >Home</Text>
                <Text
                  style={styles.navItemStyle}
                  onPress={this.navigateToScreen('User')}
                >User</Text>
                <Text
                  style={styles.navItemStyle}
                  onPress={()=>alert('Help Window')}
                >Help</Text>
                <Text
                  style={styles.navItemStyle}
                  onPress={()=>alert('Info Window')}
                >Info</Text>
              </View>
            </View>>
          </ScrollView>
          {% raw %}
          <View style={{paddingLeft: 10, paddingBottom: 30}}>
          {% endraw %}
            <Text>Copyright @ Wintho, 2020.</Text>
          </View>
        </View>
      )
    }
  }

  const styles = StyleSheet.create({
    container: {
      flex: 1,
      paddingTop: 80
    },
    imageContainer: {
      alignItems: 'center',
      padding: 50,
    },
    sectionHeading: {
      color: '#fff',
      backgroundColor: '#ef9de4',
      paddingVertical: 10,
      paddingHorizontal: 15,
      fontWeight: 'bold'
    },
    navSectionStyle: {
      backgroundColor: '#04b6ff'
    },
    navItemStyle: {
      padding: 10,
      color: '#fff'
    }

  });
  export default SideDrawer;
  ~~~

- App.js 수정

  ~~~javascript
  ...
  import PictogramHome from './src/assets/pics/home_icon.png';
  import SideDrawer from './src/my_drawer'
  ...
  {% raw %}
  <Drawer.Navigator
    initialRouteName="Home"
      drawerType="front"
      drawerPosition="right"
      drawerStyle={{
        backgroundColor: '#c6cbef',
        width: 200
      }}
      drawerContentOptions={{
        activeTintColor: 'red',
        activeBackgroundColor: 'skyblue'
      }}
      drawerContent={
        props => <SideDrawer {...props} /> // CustomDrawerContent 대체
      }
  >
  {% endraw %}
  ~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnavigation.org/](https://reactnavigation.org/)