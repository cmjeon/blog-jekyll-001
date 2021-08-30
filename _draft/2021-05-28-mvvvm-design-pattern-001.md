---
layout: single
title:  MVVM 디자인패턴 - 001
categories: 
  - front-end
tags: 
  - design pattern
  - mvvm
---

## MVVM 디자인패턴

MVVM 디자인패턴은 Model-View-ViewModel 으로 구성된 디자인패턴임

디자인패턴이란 어떤 상황의 문제를 해결하기 위해 고수들이 고민해놓은 `Best Practice`

MVVM (디자인)패턴은 마이크로소프트의 WPF 와 실버라이트 아키텍트인, 존 구스먼(John Gossman)의 [블로그](https://docs.microsoft.com/ko-kr/archive/blogs/johngossman/introduction-to-modelviewviewmodel-pattern-for-building-wpf-apps)에서 발표되었음

이름이 MVVM 이라서 Model <-> View <-> View Model 관계로 보이지만 실제 관계는 View <-> View Model <-> Model 임

View Model 이 View 와 Model 의 중간에서 결합도를 낮추는 역할을 함

왜 View 와 Model 의 결합도를 낮추어야 했을까?

MVVM 패턴의 원조격인 Martin Fowler 의 `프리젠테이션 모델 디자인 패턴` 문서에서 그 고민을 어느정도 찾을 수 있음

> GUIs consist of widgets that contain the state of the GUI screen. Leaving the state of the GUI in widgets makes it harder to get at this state, since that involves manipulating widget APIs, and also encourages putting presentation behavior in the view class.

GUI의 상태를 위젯안에 두면 API 호출, 화면변화 로직 모두 위젯에 있게 되어 상태를 알기 어렵게 된다는 뜻

MVVM 패턴을 사용하지 않은 코드와 사용한 코드를 비교하기 위해 Martin Fowler 의 프리젠테이션 모델 디자인 패턴 페이지에 있는 앨범의 일부를 구현해보았음

요구사항은 아래와 같음

- 폼화면
- 앨범제목 inputbox
- 클래식여부 checkbox
- 클래식여부를 체크 시 작곡가 inputbox가 활성화됨



## 참고

- [https://ko.wikipedia.org/wiki/모델-뷰-뷰모델](https://ko.wikipedia.org/wiki/모델-뷰-뷰모델)
- [https://martinfowler.com/eaaDev/PresentationModel.html](https://martinfowler.com/eaaDev/PresentationModel.html)