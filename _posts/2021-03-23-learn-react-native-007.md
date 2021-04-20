---
layout: single
title: react native 기초 배우기 007 - Image, Modal
categories: 
  - learn react native
tags:
  - react native
---

## Image

- 폴더 생성

  ~~~bash
  # pwd = react_native_01
  $ mkdir assets/images
  ~~~

### 로컬에서 파일 가져오기

- 폴더에 이미지 추가(파일명 steak.jpg)

- App.js 수정

  ~~~javascript
  import React, { Component } from 'react';
  import { TextInput, Button, View, Text, StyleSheet, ScrollView, Image } from 'react-native';
  ...
  import Steak from './assets/images/steak.jpg';
  ...
    render() {
      return(
        <View style={styles.mainView}>
          <Image
            style={styles.image}
            resizeMode='contain'
            source={Steak}
          />
      )
    }
  ...
  const style = StyleSheet.create({
    ...
    image: {
      width: '100%',
      height: 700
    }
  ~~~

### 원격에서 파일 가져오기

- picsum.photos 접속하여 이미지주소 가져오기

- App.js 수정

  ~~~javascript
  ...
    render() {
      return(
        <View style={styles.mainView}>
          <Image
            style={styles.image}
            source={{uri: `이미지 주소`}}
            resizeMode="contain"
            onLoadEnd={()=>alert('Image Loaded!')}
          >
          ...
      )
    }
  ...
  ~~~

## Modal

- 화면 가장 위에 표시되는 레이어로 사용자의 다른 작업을 막는 컴포넌트

### modal 파일 생성

~~~bash
# pwd = react_native_01
$ mkdir src/modal.js
~~~

~~~javascript
import React, { Component } from 'react';
import { View, Text, Button, Modal } from 'react-native';

class ModalComponent extends Component {
  state = {
    modal: false
  }

  handleModal = () => {
    this.setState({
      modal: this.stae.modal ? false : true
    })
  }

  render() {
    return (
      <View style={{width: '100%'}}>
        <Button
          title="Open Modal"
          onPress={this.handleModal}
        />
        <Modal
          visible={this.state.modal}
          animationType={'slide'} // none, slide, fade
          onShow={()=>alert('Warning!')}
        >
          <View style={{
            marginTop:60,
            backgroundColor: 'red'
          }}>
            <Text>This is modal content</Text>
          </View>
          <Button
            title="Go Back"
            onPress={this.handleModal}
          />
        </Modal>
      </View>
    )
  }
}
export default ModalComponent;
~~~

### App.js 에 추가

~~~javascript
...
import Steak from './asset/images/steak.jpg'
import Modal from './src/modal'
...
  render() {
    return(
      <View>
        <Modal/>
      </Veiw>
    )
  }
~~~

## 참고
- [https://www.inflearn.com/course/리액트-네이티브-기초](https://www.inflearn.com/course/리액트-네이티브-기초)
- [https://reactnative.dev/docs/components-and-apis](https://reactnative.dev/docs/components-and-apis)
- [https://reactnative.dev/docs/image](https://reactnative.dev/docs/image)
- [https://reactnative.dev/docs/modal](https://reactnative.dev/docs/modal)