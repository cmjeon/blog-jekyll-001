---
layout: single
title: m1 macOS에서 spring legacy project 시작하기
categories: 
  - spring
tags: 
  - spring
  - m1 macOS
---

## spring legacy project 목표 개발환경
1. java : jdk 1.8
1. build : maven
1. database : mariadb server 10
1. database tool : dbeaver
1. Web/WAS : apache tomcat 9.0
1. IDE : sts 4 with spring tool 3 add-on
1. template : spring legacy project

## java 설치
- jdk 1.8 다운로드
- zulu-community에서 생성해놓은 jdk 다운로드가 필요
- https://www.azul.com/downloads/zulu-community/?version=java-8-lts&os=macos&architecture=arm-64-bit&package=jdk
- .dmg 클릭

## maven 설치
- 인텔용 brew를 사용하여 설치
- brew install maven

## sts 설치
1. sts 다운로드 - https://spring.io/tools
2. terminal 실행 - rosetta2로 실행

## java-legacy-project를 위한 plugin 설치
- eclipse marketplace 에서
1. sts 검색
2. spring tools 3 add-on for spring tools 4.3.9 > install

## mariadb 설치
1. https://mariadb.com/kb/ko/installing-mariadb-on-macos-using-homebrew/

## dbeaver 설치
1. https://jybaek.tistory.com/858
2. brew install --cask dbeaver-community

## Tomcat 설치
1. 톰켓 웹사이트에서 적당한 버전의 tar.gz의 압축 해제
2. 해당 폴더를 sts에서 연결
