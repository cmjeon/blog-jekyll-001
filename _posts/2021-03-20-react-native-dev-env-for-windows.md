---
layout: single
title: react native 개발 환경 for windows
categories: 
  - env/config
tags: 
  - react native
  - development environment
---

## chocolatey 설치

- chocolatey 윈도우용 패키지 관리자
- cmd를 관리자권한으로 실행

  ~~~bash
  # chocolatey 설치
  > @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
  
  # 설치확인
  > choco -version
  0.10.15
  ~~~

## nodejs, npm 설치

- cmd를 관리자권한으로 실행

  ~~~bash
  # nodejs 설치
  > choco install -y nodejs.install

  # 설치확인
  > node -v
  v14.15.1

  # npm 설치확인
  > npm -v
  6.14.8
  ~~~

## python 설치

- cmd를 관리자권한으로 실행

  ~~~bash
  # python 설치
  > choco install -y python

  # 설치확인
  > python --version
  Python 3.8.1
  ~~~

## React Native CLI 설치

- cmd를 관리자권한으로 실행

  ~~~bash
  # react native CLI 설치
  > npm install -g react-native-cli

  # 설치확인
  > react-native --version
  react-native-cli: 2.0.1
  react-native: n/a - not inside a React Native project directory
  ~~~

## jdk 설치

- cmd를 관리자권한으로 실행

  ~~~bash
  # jdk 설치
  > choco install -y jdk8

  # 설치확인
  > java -version
  java version "1.8.0_211"
  Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
  Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
  > javac -version
  javac 1.8.0_211
  ~~~

## Android Studio 설치

- https://developer.android.com/studio?hl=ko 접속
- Android Studio 다운로드 > 실행

### Android Studio 설정

- Welcome : Next
- Install Type : Custom > Next
- Select UI Theme : Darcula > Next
- Verify Settings : Finish
- Android Studio 실행

#### SDK Manager 설정

- Configure > SDK Manager
- 최신 버전 Android 설치 > 현재 11.0(R) 선택 
- Show Package Detail 체크 > 최신 버전 Android 하단 체크 > Apply > 설치

### AVD Manager 설정

- 시뮬레이터를 실행할 가상디바이스
- Create Virutal Device 선택
- Select Hardware > 모델 하나 선택 > Next
- System Image > 최신 버전 Android 선택 > Next
- Android Virtual Device > Finish

### Android Studio 환경 설정

- 내 PC 우클릭 > 속성
- 고급 시스템 설정 > 환경변수
- 사용자 변수의 새로 만들기 > 변수 이름 : ANDROID_HOME, 변수 값 : {Android SDK Location}

---

**NOTE**

- Android SDK Location SDK Manager 화면에서 확인 가능

---

- adb 설치확인

  ~~~bash
  # 설치확인
  > adb --version
  Android Debug Bridge version 1.0.41
  Version 31.0.2-7242960
  Installed as C:\Users\cm083\AppData\Local\Android\Sdk\platform-tools\adb.exe
  ~~~

## react native 프로젝트 생성 & 실행

- react native 프로젝트 생성

  ~~~bash
  > npm install set save-exact=true
  > react-native init sampleApp
  ~~~

- AVD에서 가상 디바이스 실행

- 프로젝트 실행

  ~~~bash
  > cd sampleApp
  > react-native run-android
  ~~~


## 참고
- [https://dev-yakuza.posstree.com/ko/react-native/install-on-windows/](https://dev-yakuza.posstree.com/ko/react-native/install-on-windows/)