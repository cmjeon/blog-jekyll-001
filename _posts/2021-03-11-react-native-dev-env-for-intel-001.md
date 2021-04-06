---
layout: single
title: react native 개발 환경 for intel 001

categories: 
  - learning
tags: 
  - react native basic
---

## 리액트 네이티브란

- 페이스북의 오픈소스 모바일 응용 프로그램
- 네이티브 앱 개발을 위한 자바스크립트 프레임워크
- 크로스플랫폼, 안드로이드 ios 동작가능
- 기본언어 자바스크립트, 쉽다
- 선수지식 html, css, js es6, 리액트

## 리액트 네이티브 기본원리

- 자바스크립트 > 리액트 네이티브 > js bundle > js thread > bridge > native Threads

## js bundle 만드는 방법 2가지

### expo CLI

- 개발 환경 구축 용이하고 실제 개발이 쉽다
- OS Layer와 직접 상호작용 불가능
- Expo에서 제공해주는 모듈만 사용 가능
- 개발 관점에서 자유도가 낮음

### React Native CLI > 추천되는 방법

- Expo로 접근하지 못하는 Native 기능에 접근 가능
- 원하는 언어로 추가 작성 가능
- OS Layer와 직접적인 상호작용 가능
- 초기 개발 환경 구축 및 개발 시간 다소 시간 소요
- Mac의 경우에만 iOS/Android 지원, iOS 시뮬레이터 사용가능

## 개발환경 설치

- intel mac 기반 react native 개발환경 구축 방법임

### xcode-select

~~~bash
# xcode-select 설치
$ xcode-select --install
~~~

### nvm(Node Version Manager)

- (https://gist.github.com/falsy/8aa42ae311a9adb50e2ca7d8702c9af1) 접속

~~~bash
# nvm 복사
$ sudo curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

# profile에 설정 입력
$ vi ~/.bash_profile

# 설정 입력
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm

# profile 재설정
$ source ~/.bash_profile

# 버전 확인
$ nvm --version
0.33.11
~~~

#### zsh 사용자라면?

- 설정을 .zshrc에 nvm 설정을 입력

~~~bash
# profile에 설정 입력
$ vi ~/.zshrc

# 설정 입력
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm

# profile 재설정
$ source ~/.zshrc

# 버전 확인
$ nvm --version
0.33.11
~~~

#### nvm 사용하기

~~~bash
# 설치한 node 버전확인
$ nvm ls

# node 설치
$ nvm install 00.00.00

# node 특정 버전 사용
$ nvm use 00.00.00
~~~

### node.js

~~~bash
# node 설치
$ nvm install 10.15.1

# 버전 확인
$ node -v
v.10.15.1
~~~

### npm(Node Package Manager)

- npm은 node.js를 설치하면 자동으로 설치되고, 버전만 확인

~~~bash
# 버전 확인
$ npm --version
6.4.1
~~~

### Android Studio

- https://developer.android.com/studio?hl=ko 접속
- Android Studio 다운로드 > 실행
- Application에 드래그
- Launchpad에서 Android Studio 실행

#### Android Studio 설정

- Welcome > Next
- Install Type > Next
- Select UI Theme > Darcula > Next
- Verify Settings > Finish
- Android Studio 실행

##### SDK Manager 설정

- Configure > SDK Manager
- 최신 버전 Android 설치 > 현재 11.0(R) 선택 
- Show Package Detail 체크 > 최신 버전 Android 하단 체크 > Apply > 설치

#### AVD Manager 설정

- 시뮬레이터를 실행할 가상디바이스
- Create Virutal Device 선택
- Select Hardware > 모델 하나 선택 > Next
- System Image > 최신 버전 Android 선택 > Next
- Android Virtual Device > Finish

### 환경설정

~~~bash
$ vi ~/.bash_profile

# 설정 입력
export ANDROID_HOME={Android SDK Location} # Check Android SDK Location in your SDK Manager
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# 설정 확인
$ adb
Android Debug Bridge version 1.0.41
Version ...
~~~

### java

- java 11 설치

~~~bash
# 버전 확인
$ java --version
~~~

### Xcode

- Appstore에서 Xcode 검색 후 설치
- xcode preference > Locations > Command Line Tools의 설정 확인

### Visual Studio Code

- visual studio code download 검색 > 설치

### CocoaPod

- Open Library들을 내 프로젝트에 확장시킬 수 있게 해주는 iOS용 프로그램

~~~bash
$ sudo gem install cocoapods {버전}

# 버전 확인
$ pod --version
1.10.1
~~~

#### brew에서 CocoaPod 설치

~~~bash
$ sudo brew install cocoapods
# xcod licence 문제
You have not agreed to the Xcode license. Please resolve this by running:
  sudo xcodebuild -license accept
$ sudo xcodebuild -license accept
~~~

### React Native CLI

~~~
$ npm install -g react-native-cli

# 버전 확인
$ react-native --version
react-native-cli: 2.0.1
react-native: n/a - not inside a React Native project directory
~~~

## 참고
-(https://www.inflearn.com/course/리액트-네이티브-기초)