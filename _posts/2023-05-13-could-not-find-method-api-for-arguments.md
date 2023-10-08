---
layout: single
title: "jsonwebtoken 사용을 위해 gradle 에 api 추가하다가 만난 오류"
categories:
  - "오류"
---

## 오류 상황

jsonwebtoken 을 사용하기 위해 gradle dependency 에 api 를 추가하려고 하였다.

빌드 하였더니 아래 오류를 만났다.

> AbstractDynamicObject$CustomMessageMissingMethodException: Could not find method api() for arguments ...

## 해결방법

build.gradle 파일에 plugins { ... } 영역이 있다.

이 영역 내에 id 'java' 를 id 'java-library' 로 변경한다.

```
// ASIS
plugins {
	id 'java'
}

...
```

```
// TOBE
plugins {
	id 'java-library'
}

...
```


## 참고

- [https://stackoverflow.com/questions/49423330/could-not-find-method-api-for-arguments-directory-libs](https://stackoverflow.com/questions/49423330/could-not-find-method-api-for-arguments-directory-libs)
