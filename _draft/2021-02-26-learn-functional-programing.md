---
layout: single
title: 함수형 프로그래밍 배우기
categories: 
  - learning
tags: 
  - functional programming
---

## 함수형 프로그래밍에서 함수의 특징
- 인풋과 아웃풋이 있다.
- 외부환경으로부터 독립적이다.
- 같은 인풋으로는 같은 아웃풋을 만들어내고, 다른 함수에 결과물 외에 다른 영향을 주지 않는다. (순수함수이다)
- 함수형 프로그래밍은 '선언형'이다.
  - '명령형' 프로그래밍 > 너는 이것을, 너는 저것을 해서 그것을 만들어내라.
  - '선언형' 프로그래밍 > 이걸하고, 저걸하면 그것이 된다.
- 함수는 '행위'가 아닌 '값'이다
  - 함수를 함수형이나 변수형으로 선언가능
  ```javascript
  //함수형
  function say_it(given) {
    console.log(given);
  }
  //변수형
  var say_it = function(given) {
    console.log(given);
  }
  //es6 변수
  const say_it = (given) => console.log(given);
  ```
  - 단순 호출 가능, 콜백 함수로 호출 가능
  ```javascript
  say_it('안녕');
  // 안녕
  ['aaa','bbb','cccc'].foreach(say_it);
  // aaa
  // bbb
  // ccc
  ```

## 함수형 프로그래밍의 장점
- 부작용에 의한 문제로부터 보다 자유롭다.

## 서로 다른 쓰레드가 한 변수를 고칠 때 발생하는 문제
- 변수에 lock 을 걸거나, synchronized 방법으로 변수를 동기화한다. > 아주 주의를 기울여 코딩해야함

## 함수형 프로그래밍은?
- 함수의 동작에 의한 변수의 부수적인 값 변경을 원천 배제한다.
- 외부의 변수를 이용하더라도 사본을 복사해서 작업한다.

## 값이 변하지 않는 프로그램이 무슨 소용이 있나?
- 값을 아예 바꾸지 않는 것이 아니고 특정 범위에서라도 부수적인 효과없이 프로그램을 짜는 것이 함수형 프로그램이다.

## 

## 참고
- https://www.youtube.com/watch?v=jVG5jvOzu9Y