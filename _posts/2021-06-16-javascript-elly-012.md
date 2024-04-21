---
layout: single
title:  javascript-012 - 비동기의 꽃 Javascript async 와 await
categories: 
  - learning javascript by ellie
tags: 
  - javascript
  - promise
---

## async & await

promise then 이어진 코드는 가독성이 다소 떨어짐

async 와 await 는 더 사용을 편리하게 하기 위한 syntactic sugar (ex. class)

### async

```js
function fetchUser() {
  // do network request in 10 secs...
  return 'ellie';
}

function user = fetchuser();
console.log(user)
```

프로미스로 처리

```js
function fetchUser() {
  return new Promise((resolve, reject) => {
    // do network request in 10 secs...
    resolve('ellie');
  });
}

const user = fetchUser();
user.then(console.log(user));
console.log(user);
```

async await 적용

```js
async function fetchUser() {
  // do network request in 10 secs...
    resolve('ellie');
  });
}

const user = fetchUser();
user.then(console.log);
console.log(user);
```

### await

```js
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function getApple() {
  await delay(3000);
  return 'Apple';
}

async function getBanana() {
  await delay(3000);
  return 'Banana';
}

// Promise
function pickFruits() {
  return getApple().then(apple => {
    .then(banana => `${apple} + ${banana}`);
  });
}

// async await
async function pickFruits() {
  const apple = await getApple();
  const banana = await getBanana();
  return `${apple} + ${banana}`;
}

pickFruits().then(console.log);
```

## await 병렬 처리

위 케이스는 getApple, getBanana 를 병렬로 처리할 수 있음
await 는 각 데이터를 동시에 가져올 수도 있음

```js
// await 병렬로 처리
async function pickFruits() {
  const applePromise = getApple();
  const bananaPromise = getBanana();
  const apple = await applePromise;
  const banana = await bananaPromise;
  return `${apple} + ${banana}`;
}

pickFruits().then(console.log);
```

## 유용한 Promise APIs

### Promise.all

```js
function pickAllFruits() {
  return Promise.all([getApple(), getBanana()])
    .then(fruits => fruits.join(' + '));
}

pickAllFruits().then(console.log);
```

### Promise.race

```js
function pickOnlyOne() {
  return Promise.race([getApple(), getBanana()]);
}

pickOnlyOne().then(console.log);
```

## 참고
- [https://youtu.be/JB_yU6Oe2eE
