---
layout: single
title: Webstorm 배우기 - 001
categories: 
  - learn Webstorm
tags:
  - IDE
  - Webstorm
---

## Webstorm 구매

Webstorm 은 Jetbrains 사의 javascript 용 IDE 임

Webstorm 유료툴이라 지금까지는 Visual Studio Code 를 썼기때문에 딱히 관심두지 않았음

최근 이직한 회사에서 Webstorm 이 지원되어, 회사에서는 Webstorm, 집에서는 vscode 를 사용해야 하는 상황이 됨

이 참에 새로운 툴을 배워보자는 생각이 들어 이번에 개인용도로 구매하게 되었음

가격은 개인용으로 첫 1년이 8만원 정도, 2년, 3년까지 점점 저렴해짐

[Webstorm 가격](https://www.jetbrains.com/ko-kr/webstorm/buy/){:target="_blank"}

꽤 괜찮은 가격이라는 생각이 듬

IDE 내에 튜토리얼이 있어서 이를 해보고 내용을 기록하려고 함

## Editor Basic

### Code Completion

1. 키를 입력할 때마다 뭔가를 제안해줌. 그리고 엔터나 탭을 누르면 제안중에 가장 위의 제안이 선택됨

1. 파라미터가 필요한 곳을 선택하고 `Cmd + p` 을 누르면 어떤 파라미터가 들어와야 하는지 제안해줌

1. 객체나 함수 등을 선택하고 `F1` 을 누르면 관련 문서가 표시되고 이동할 수 있음
  
[![Code Completion](/assets/images/20211205-01.gif)](/assets/images/20211205-01.gif)

### The Power of Code Inspections

1. 소스코드에서 `F2`을 누르면 개선가능한 항목을 선택하고 개선방법을 제안해줌

1. 변수를 선택하고 `Opt + Enter` 을 누르면 개선가능한 방법 목록이 표시됨

1. 목록중에 하나를 선택하고 `Enter` 을 선택함

1. 우측 상단에 녹색 체크마크가 있다면 개선가능한 항목은 모두 수정된 상태임

1. function 으ㄹ 선택하고 `Opt + Enter` 을 누르고 `Convert to arrow function` 을 선택하면 화살표 함수로 변경가능

[![Code Completion](/assets/images/20211205-02.gif)](/assets/images/20211205-02.gif)

### Refactorings in a Nutshell

1. `Ctrl + t` 을 누르고 `Rename...` 을 선택하여 리팩터링을 진행할 수 있음

1. 특정 요소의 프로퍼티를 선택하고 `Cmd + Opt + v` 를 누르면 새로운 이름의 변수로 생성가능

[![Code Completion](/assets/images/20211205-03.gif)](/assets/images/20211205-03.gif)

### Code Editing Tops and Tricks

1. `Cmd + Opt + l' 을 누르면 들여쓰기 조정(reformat)

1. `Opt + ↑/↓` 을 누르면 누를 때마다 블럭지정이 커짐/작아짐

1. 블럭지정 후 `Cmd + Opt + /` 을 누르면 주석처리

1. 블럭지정 후 `Opt + ⌫` 을 누르면 삭제

1. 블럭지정 후 `Cmd + Shift + ↑/↓` 을 누르면 블럭이 위/아래로 이동

1. HTML 에서 특정 태그를 선택하고 `Ctrl + g` 를 누르면 누를 때마다 같은 태그를 더 선택함

1. `Cmd + d` 을 누르면 라인 복제

1. `Cmd + ⌫` 을 누르면 라인 삭제

[![Code Completion](/assets/images/20211205-04.gif)](/assets/images/20211205-04.gif)

### Efficient Navigation

1. `Cmd + e` 을 누르면 최근파일 열기 팝업

1. 최근파일 열기 팝업에서 `Structure` 탭 선택 or `Cmd + 7` 을 선택하면 `Structure` 탭 표시

1. `Structure` 탭에서 문자 입력하여 요소탐색 가능

1. `Structure` 탭에서 요소를 선택한 후 `Opt + F7` 을 선택하면 `Find` 탭 표시

1. `Shift + ESC` 을 누르면 하단에 표시된 탭 닫기

1. `Shift` 을 두번 눌러 `모든 곳을 검색` 창 열기

1. `Cmd + b` 을 누르면 해당 변수가 정의된 곳으로 이동

[![Code Completion](/assets/images/20211205-05.gif)](/assets/images/20211205-05.gif)

## 참고 
- []()
