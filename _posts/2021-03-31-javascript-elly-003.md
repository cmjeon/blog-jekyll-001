---
layout: single
title:  javascript-003 - 코딩의 기본, operator, if, for loop 코드리뷰 팁
categories: 
  - learning javascript by ellie
tags: 
  - javascript
---

## String concatenation

```javascript
console.log('my' + ' cat'); // my cat
console.log('1' + 2); // 12
console.log(`string literals: 1 + 2 = ${1 + 2}`); // string literals: 1 + 2 = 3
```

## Numeric operators

```javascript

console.log(1 + 1); // 2 add
console.log(1 - 1); // 0 substract
console.log(1 / 1); // 1 divide
console.log(1 * 1); // 1 multiply
console.log(5 % 2); // 1 remainder
console.log(2 ** 3); // 8 exponentiation
```

## Increment and decrement operators

```javascript
let counter = 2;
const preIncrement = ++counter;
// count = counter + 1;
// preIncrement = counter;
console.log(`preIncrement: ${preIncrement}, counter: ${counter}`); // preIncrement: 3, counter: 3
const postIncrement = counter++;
// postIncrement = counter;
// counter = counter + 1;
console.log(`postIncrement: ${postIncrement}, counter: ${counter}`); // preIncrement: 3, counter: 4

const preDecrement = --counter;
console.log(`preDecrement: ${preDecrement}, counter: ${counter}`); // preIncrement: 3, counter: 3
const postDecrement = counter--;
console.log(`postDecrement: ${postDecrement}, counter: ${counter}`); // preIncrement: 3, counter: 2
```

## Assignment operators

```javascript
let x = 3;
let y = 6;
x += y; // x = x + y;
x -= y; // x = x - y;
x *= y; // x = x * y;
x /= y; // x = x / y;
```

## Comparision operators

```javascript
console.log(10 < 6); // less than
console.log(10 <= 6); // less than or equal
console.log(10 > 6); // greater than
console.log(10 >= 6); // greater than or equal
```

## Logical operators: || (or), && (and), ! (not)

### || (or), finds the first truthy value

- 복합적인 논리연산 시에 복잡한 연산일수록 뒤에 배치

```javascript
const value1 = true;
const value2 = 4 < 2;
console.log(`or: ${value1 || value2 || check()}`);

function check() {
  for(let i=0; i<10; i++) {
    // wasting time
    console.log('ttt');
  }
  return true;
}
```

### && (and), finds the first falsy value

- 복합적인 논리연산 시에 복잡한 연산일수록 뒤에 배치
- often used to compress long if-statement

```javascript
const value1 = true;
const value2 = 4 < 2;
console.log(`and: ${value1 && value2 && check()}`);

function check() {
  for(let i=0; i<10; i++) {
    // wasting time
    console.log('ttt');
  }
  return true;
}
```

- nullableObject && nullableObject.something

```javascript
if (nullableObject != null) {
  nullableObject.something;
}
```

### ! (not)

```javascript
const value1 = true;
console.log(!value1); // false
```

## Equality

```javascript
const stringFive = '5';
const numberFive = 5;

// loose equality, with type conversion
console.log(stringFive == numberFive); // true
console.log(stringFive != numberFive); // false

// strict equality, no type conversion
console.log(stringFive === numberFive); // false
console.log(stringFive !== numberFive); // false

// object equality by reference
const ellie1 = { name: 'ellie' };
const ellie2 = { name: 'ellie' };
const ellie3 = ellie1;
console.log(ellie1 == ellie2); // false
console.log(ellie1 === ellie2); // false
console.log(ellie1 === ellie3); // true

// equality - puzzler
console.log(0 == false); // true
console.log(0 === false); // false
console.log('' == false); // true
console.log('' === false); // false
console.log(null == undefined); // true
console.log(null === undefined); // false
```

## Conditional operators: if

- if, else if, else

```javascript
const name = 'coder';
if(name === 'ellie') {
  console.log('Welcome, Ellie');
} else if(name === 'code') {
  console.log('You are amazing coder');
} else {
  console.log('unknown');
}
```

## Ternary operator: ?

- condition ? value : value2;

```javascript
console.log(name === 'ellie' ? 'yes' : 'no'); // no
```

## Switch statement

- use for muliple if checks
- use for enum-like value check
- use for multiple type checks in TS

```javascript
const browser = 'IE';
switch (browser) {
  case 'IE':
    console.log('go away');
    break;
  case 'Chrome':
  case 'Firefox':
    console.log('love you!');
    break;
  default:
    console.log('same all!');
    break;
}
```

## Loops

```javascript
// while loop, while the condition is truthy,
// body code is executed.
let i = 3;
while (i>0) {
  console.log(`while: ${i}`);
  i--;
}
// while: 3
// while: 2
// while: 1

// do while loop, body code is executed first,
// then check the condition
do {
  console.log(`do while: ${i}`);
  i--;
} while ( i > 0);
// do while 3
// do while 2
// do while 1
// do while 0

// for loop, for(begin; condition; step)
for (i=3; i>0; i--) {
  console.log(`for: ${i}`);
}
// for: 3
// for: 2
// for: 1
for (let i=3; i>0; i=i-2) {
  // inline variable declaration
  console.log(`inline variable for: ${i}`);
}
// inline variable for: 3
// inline variable for: 1

// nested loops
for (let i=0; i<10; i++) {
  for(let j=0; j<10; j++) {
    console.log(`i: ${i}, j:${j}`);
  }
}

// break, continue
// Q1. iterate from 0 to 10 and print only even numbers (use continue)
for (let i=0; i<=10; i++) {
  if(i % 2 !== 0) {
    continue;
  }
  console.log(`q1. ${i}`);
}

// Q2. iterate from 0 to 10 and print numbers until reaching 8 (use break)
for (let i=0; i<=10; i++) {
  if(i > 8) {
    break;
  }
  console.log(`q2. ${i}`);
}
```


## 참고
- [https://youtu.be/YBjufjBaxHo]