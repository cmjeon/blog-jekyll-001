---
layout: single
title:  javascript-010 - 비동기 처리의 시작 콜백 이해하기, 콜백 지옥 체험
categories: 
  - learning javascript by ellie
tags: 
  - javascript
  - callback
---

## callback

자바스크립트는 동기적임

호이스팅(hoisting)된 이후 순서대로 실행됨

호이스팅(hoisting) : var, funciton 선언이 제일 위로 올라가는 현상

```js
console.log('1'); 
console.log('2');
console.log('3');
// 1, 2, 3

console.log('1');
setTimeout(() => {
  console.log('2')
}, 1000)
console.log('3');
// 1, 3, 2
```

## 동기형 콜백

```js
function printImmediately(print) {
  print();
}
printImmediately((()=> console.log('hello'));
// 호출 즉시 실행
```


## 비동기형 콜백

```js
function printWithDelay(print, timeout) {
  setTimeout(print, timeout);
}
printWithDelay(() => console.log('async callback'), 2000);
// 2초 뒤에 실행
```

## callback 지옥 체험

```js
class UserStorage {
  loginUser(id, password, onSuccess, onError) {
    setTimeout(() => {
      if (
        (id  === 'ellie' && password === 'dream') ||
        (id === 'coder' && password === 'academy')
      ) {
        onSuccess(id);
      } else {
        onError(new Error('not found'));
      }
    },2000)
  };

  getRoles(user, onSuccess, onError) {
    setTimeout() => {
      if(user === 'ellie') {
        onSuccess({ name: 'ellie', role: 'admin' });
      } else {
        onError(new Error('no access'));
      }
    }
  }, 1000)
}
```

위 함수들을 활용하여 진행

```js
const userStorage = new UserStorage();
const id = promt('enter your id');
const password = prompt('enter your password');
userStorage.loginUser(id, password, user => {
  userStorage.getRoles(user, (userWithRole) => {
    alert(`Hello ${userWithRole.name}, you have a${userWithRole.role} role`);
  }, error => {
    console.log(error);
  })
}, error => {
  console.log(error);
})
```

## 참고
- [https://youtu.be/s1vpVCrT8f4]