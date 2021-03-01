---
layout: posts
title: m1 macOS에서 spring legacy project 시작하기
categories: 
  - spring
tags: 
  - spring
  - m1 macOS
---
1. 설치하는 방법 2가지
    1. brew
    2. dmg 다운로드
2. 개발환경
    1. java : jdk 1.8
    2. build : maven
    3. database : mariadb server 10
    4. database tool : dbeaver
    5. Web Server : apache
    6. WAS : apache tomcat 9.0
    7. IDE : sts 4 with spring tool 3 add-on)
    8. template : spring legacy project
3. 응용프로그램 > 유틸리티 > 터미널 > 우클릭 > 정보가져오기 > 로제타 사용
4. $ xcode-select —install
5. brew 설치
    1. https://brew.sh/index_ko
    2. https://velog.io/@mordred/Apple-M1-Mac%EC%97%90%EC%84%9C-HomeBrew-%EC%84%A4%EC%B9%98
    3. .zshrc 에 alias brew='arch -x86_64 /usr/local/bin/brew' 추가
    4. https://www.44bits.io/ko/post/setup-apple-silicon-m1-for-developers 참고
6. jdk 1.8 다운로드
    1. https://www.azul.com/downloads/zulu-community/?version=java-8-lts&os=macos&architecture=arm-64-bit&package=jdk
    2. .dmg 클릭
7. gradle or maven 설치
    1. brew install gradle
    2. brew install maven
8. sts 설치
    1. sts 다운로드
        1. https://spring.io/tools
    2. terminal 실행 - rosetta2로 실행
    3. $ hdiutil
9. 취소 - 테스트 소스 실행
    1. https://spring.io/guides/gs/spring-boot/
10. 취소 - new spring starter dependencies
    1. spring boot devtools
    2. lombok
    3. mariadb driver
    4. spring web
11. eclipse marketplace
    1. sts 검색
    2. spring tools 3 add-on for spring tools 4.3.9 > install
12. mariadb 설치
    1. https://mariadb.com/kb/ko/installing-mariadb-on-macos-using-homebrew/
13. Mysql “Access denied for user ‘root’@’localhost'” 오류 해결하기
    1. https://kimtaekju-study.tistory.com/257 > pw : 사용자비번
    2. use mysql;
    3. grant all on *.* to ‘root’@‘localhost’ identified by ‘root’ with grant option;
    4. flush privileges;
    5. exit;
14. dbeaver 설치
    1. https://jybaek.tistory.com/858
    2. brew install --cask dbeaver-community
15. Tomcat 설치
    1. 톰켓 웹사이트에서 적당한 버전의 tar.gz의 압축 해제
    2. 해당 폴더를 sts에서 연결