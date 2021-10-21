---
layout: single
title:  javascript-011 - 프로미스 개념부터 활용까지 Javascript Promise
categories: 
  - learning javascript by ellie
tags: 
  - javascript
  - promise
---

## Promise

Promise is a Javascript object for asynchronous operation

### State : pending -> fulfilled or rejected

Promise 의 상태가 수행중(pending)인지, 수행완료(fulfilled)인지, 거절(rejected)인지

### Producer vs Consumer

Producer 원하는 데이터를 만드는 Promise Object

Consumer 원하는 데이터를 소비하는 Promise Object

### Producer

```js
const promise = new Promise((resolve, reject) => {
  // do some heavy work (network, read files)
  console.log('doing something...');
  setTimeout(() => {
    // 성공하면 resolve
    resolve('ellie');
    // 실패하면 reject
    reject(new Error('no network'));
  }, 2000)
});
```

프로미스가 생성될 때 executor 가 자동으로 실행됨

executor? new Promise 내부의 함수

```js
// excutor
(resolve, reject) => {
  // do some heavy work (network, read files)
  console.log('doing something...');
  setTimeout(() => {
    // 성공하면 resolve
    resolve('ellie');
    // 실패하면 reject
    reject(new Error('no network'));
  }, 2000)
}
```

### Consumer

then, catch, finally 에서 사용가능

then 의 value 는 Producer 의 resolve 의 값과 연결

catch 의 error 은 Producer 의 reject 의 값과 연결

```js
promise.then((value) => {
  console.log(value);
}).catch(error => {
  console.log(error);
}).finally(()=> {
  console.log('finally')
})
```

## Promise Chaining

```js
const fetchNumber = new Promise((resolve, reject) => {
  setTimeout(()=> {
    resolve(1);
  }, 1000);
});

fetchNumber
  .then(num => num * 2)
  .then(num => num * 3)
  .then(num => {
    return new Promise((resolve), reject) => {
      setTimeout(() => {
        resolve(num - 1), 1000);
      });
    }
  })
  .then(num => console.log(num))
```

위 코드의 출력값과 소요시간은?

## 오류를 잘 처리 하자

```js
const getHen = () => {
  new Promise((resolve, reject) => {
    setTimeout(()=> resolve('닭'), 1000)
  });
}
const getEgg = hen => {
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(`${hen} => '달걀'`), 1000);
  })
}
const cook = egg => {
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(`${egg} => '계란 후라이'`), 1000);
  })
}
getHen()
  .then(hen => getEgg(hen))
  .then(egg => cook(egg))
  .then(meal => console.log(meal));
```





## 참고
- [https://youtu.be/s1vpVCrT8f4]