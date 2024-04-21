---
layout: single
title:  javascript-007 - 배열 제대로 알고 쓰자. 자바스크립트 배열 개념과 APIs 총정리
categories: 
  - learning javascript by ellie
tags: 
  - javascript
---

## Array

### 자료구조

- javascript === dynamically typed language 이지만 권장하지 않음
- 검색, 정렬, 삽입, 삭제의 속도를 고려

### Declaration

```javascript
const arr1 = new Array();
const arr2 = [1, 2];
```

### Index position

```javascript
const fruits = ['apple', 'banana'];
console.log(fruits); // ['apple', 'banana']
console.log(fruits.length); // 2
console.log(fruits[0]); // apple
console.log(fruits[1]); // banana
console.log(fruits[2]); // undefined
console.log(fruits[fruits.length - 1]); // banana
```

### Looping over an array

```javascript
const fruits = ['apple', 'banana'];

for (let i=0; i<fruits.length; i++) {
  console.log(fruits[i]);
}

for(let value of fruits) {
  console.log(value);
}

fruits.forEach((fruit) => console.log(fruit));
```

### Addtion, deletion, copy, push, pop

```javascript
const fruits = ['apple', 'banana'];

// push
fruits.push('strawberry', 'peach');
console.log(fruits); // ['apple', 'banana', 'strawberry', 'peach'];

// pop
fruits.pop();
fruits.pop();
console.log(fruits); // ['apple', 'banana'];

// unshift : like push
fruits.unfhift('strawberry', 'lemon']);
console.log(); // ['strawberry', 'lemon', 'apple', 'banana'];

// shift : like pop
fruits.shift();
fruits.shift();
console.log(); // ['apple', 'banana'];

// shift, unshift is slow than pop, push;
// shift 와 unshift 는 값을 넣고 빼기 위해 기존의 위치를 이동시키는 방식으로 작업하기 때문

// splice
fruits.push('strawberry', 'peach', 'lemon');
console.log(fruits); // ['apple', 'banana', 'strawberry', 'peach', 'lemon']

fruits.splice(1);
console.log(fruits); // ['apple']

fruits.splice(1, 1);
console.log(fruits); // ['apple', 'strawberry', 'peach', 'lemon']

fruits.splice(1, 1, 'greenapple', 'watermelon');
console.log(fruits); // ['apple', 'greenapple', 'watermelon', 'peach', 'lemon']

// concat
const fruits2 = ['pear','coconut'];
const newFruits = fruits.concat(fruits2);
console.log(newFruits);
```

### Searching

```javascript
const fruits  = ['apple', 'greenapple', 'watermelon', 'peach', 'lemon'];
console.log(fruits); // ['apple', 'greenapple', 'watermelon', 'peach', 'lemon']

// indexOF
console.log(fruits.indexOf('apple')); // 0
console.log(fruits.indexOf('watermelon')); // 2
console.log(fruits.indexOf('coconut'); // -1

// includes
console.log(fruits.includes('watermelon')); // true
console.log(fruits.includes('coconut')); // false

// lastIndexOf
fruits.push('apple');
console.log(fruits.indexOf('apple')); // -1
console.log(fruits.lastIndexOf('apple')) // 5
```

## 참고
- [https://youtu.be/yOdAVDuHUKQ](https://youtu.be/yOdAVDuHUKQ)
