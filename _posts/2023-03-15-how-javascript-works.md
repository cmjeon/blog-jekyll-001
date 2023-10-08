---
layout: single
title: "javascript 가 동작하는 방식"
categories:
  - "javascript"
---

javascript 가 동작하는 방식을 알아보자.

브라우저에서 서버로부터 데이터를 조회하는 로직을 만든다고 한다면 javascript 에서 대충 이런 코드로 만들어진다.

```javascript
function greeting() {
    sayHi();
}

function sayHi() {
    // 서버에서 데이터 조회하기
    return getData('<https://somewhere.com/get>...');
}

greeting();
```

javascript 는 '싱글 쓰레드 방식'이기 때문에 하나의 콜스택을 가지고 작업을 한다.

하나의 콜스택에서 위 소스코드의 실행을 살펴본다.

1. greeting 을 콜스택에 올림
2. sayHi 를 콜스택에 올림
3. [*] getData 를 콜스택에 올림
4. getData 가 서버로부터 받은 데이터를 반환함
5. 콜스택에서 getData 를 제거
6. 콜스택에서 sayHi 를 제거
7. 콜스택에서 greeting 을 제거

> 쓰레드(Thread)? 콜스택(Call stack)?
> - 쓰레드는 스택 영역이 고유한 실행 단위이다.
> - 콜스택은 위에 쌓인 일이 처리되어야 아래의 일이 처리가 가능한 자료구조 (ex. 쌓아놓은 접시), First in, last out

만약 3. getData 가 서버로부터 데이터를 조회해오는 작업이 오랜기간 기다려야 하는 작업이라면 어떻게 될까?

심지어 끝날지 안끝날지도 모른다면?

그렇게 되면 getData 가 콜스택에서 제거되지 않았기 때문에 javascript 는 이후 작업을 할 수 없게 된다.

javascript 는 서버로부터 데이터를 조회해오는 것 같은 언제 끝날지 모르는 작업을 할 때는 쓰레드 외부로 위임하고 작업이 끝나면 결과를 받아서 처리하도록 만들었다.

![Event Loop](https://miro.medium.com/v2/resize:fit:1400/1*iHhUyO4DliDwa6x_cO5E3A.gif)

이제 javascript 는 언제 끝날지 모르는 작업 을 해야하면 Web API 에 작업을 위임한다.

Web API 는 작업이 끝나면 Callback Queue 에 완료된 작업을 올려놓는다.

Event Loop 는 콜스택이 비어있을 때, Callback Queue 에 있는 작업을 콜스택으로 옮겨놓는다.

이런 구조에서 소스코드의 실행단계를 보면

1. greeting 을 콜스택에 올림
2. sayHi 를 콜스택에 올림
3. getData 를 콜스택에 올림
4. [*] getData 가 할 일을 Web API 한테 시킴
5. 콜스택에서 getData 를 제거
6. 콜스택에서 sayHi 를 제거
7. 콜스택에서 greeting 을 제거
8. (5 ~ 7 단계 중 어디쯤에서) getData 가 작업을 완료하면 조회결과를 Task Queue 에 올림
9. (콜스택을 비어있다면) Event Loop 가 Task Queue 에 있는 조회결과를 콜스택에 올림
10. 콜스택에 올라온 조회결과를 수행함
11. 콜스택에서 조회결과를 제거

이제 javascript 는 싱글 쓰레드여도 `언제 끝날지 모르는 작업`을 할 수 있게 된다.

우리는 이렇게 언제 끝날지 모르는 작업을 `비동기 작업` 이라고 한다.
