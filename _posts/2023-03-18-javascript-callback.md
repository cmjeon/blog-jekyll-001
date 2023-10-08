---
layout: single
title: "javascript callback 함수"
categories:
  - "javascript"
tags:
  - "callback"
---

> "I will call back later!"A callback is a function passed as an argument to another functionThis technique allows a function to call another functionA callback function can run after another function has finished - w3schools.com

javascript 에서 비동기 작업을 처리하는 방법 중에 하나가 callback 함수이다.
callback 함수는 작업의 순서를 보장해준다는 점에서 유용하다.

## 1급 시민

변수에 들어갈 수 있는 것은 1급 시민

javascript 에서 숫자, 문자, 함수는 1급 시민이다.

함수라 다른 함수의 리턴값이 될 수 있다면 그 언어는 함수를 1급 시민으로 대우하고 있는 것이다.

callback 함수는 함수에 변수로 전달되어 함수내에서 필요에 따라 호출된다.

> 이름이 왜 콜백함수일까?
> 무엇을 할지는 미리 정해두고, 때가 오면 실제로 행하는 것이니까 비단주머니 함수, 호리병 함수라고 하는게 더 낫겠다.

## 콜백함수 예시

array.filter() 함수가 좋은 예시이다.

[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

```javascript
const words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

const result = words.filter(word => word.length > 6);

console.log(result);
// Expected output: Array ["exuberant", "destruction", "present"]
```

filter() 메소드를 직접 만들어보면 파라미터로 전달받은 함수를 이용해 처리하는 것을 확인해볼 수 있다.

```javascript
const words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

const filter = (callback, iter) => {
    let res = [];
    for (const a of iter) {
        if (callback(a)) {
            res.push(a);
        }
    }
    return res;
};

const result = filter(word => word.length > 6, words);

console.log(result);
```

## 참고

- [https://www.w3schools.com/js/js_callback.asp](https://www.w3schools.com/js/js_callback.asp)
- [https://www.youtube.com/watch?v=TAyLeIj1hMc](https://www.youtube.com/watch?v=TAyLeIj1hMc)
- [https://www.inflearn.com/course/functional-es6/dashboard](https://www.inflearn.com/course/functional-es6/dashboard)
