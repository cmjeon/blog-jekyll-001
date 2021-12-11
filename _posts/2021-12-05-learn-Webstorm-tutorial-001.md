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

1. 파라미터가 필요한 곳에 커서를 올리고 `Cmd + p` 을 누르면 어떤 파라미터가 들어와야 하는지 제안해줌

1. 객체나 함수 등에 커서를 올리고 `F1` 을 누르면 관련 문서가 표시되고 이동할 수 있음
  
[![Code Completion](/assets/images/20211205-01.gif)](/assets/images/20211205-01.gif)

### The Power of Code Inspections

1. 소스코드에서 `F2`을 누르면 개선가능한 항목을 선택하고 개선방법을 제안해줌

1. 변수에 커서를 올리고 `Opt + Enter` 을 누르면 개선가능한 방법 목록이 표시됨

1. 목록중에 하나를 선택하고 `Enter` 을 선택함

1. 우측 상단에 녹색 체크마크가 있다면 개선가능한 항목은 모두 수정된 상태임

1. function 에 커서를 올리고 `Opt + Enter` 을 누르고 `Convert to arrow function` 을 선택하면 화살표 함수로 변경가능

[![Code Completion](/assets/images/20211205-02.gif)](/assets/images/20211205-02.gif)

### Refactorings in a Nutshell

1. `Ctrl + t` 을 누르고 `Rename...` 을 선택하여 리팩터링을 진행할 수 있음

1. 특정 요소의 프로퍼티를 선택하고 `Cmd + Opt + v` 를 누르면 새로운 이름의 변수로 생성가능

[![Code Completion](/assets/images/20211205-03.gif)](/assets/images/20211205-03.gif)

### Code Editing Tops and Tricks



### Efficient Navigation



## 참고 
- []()