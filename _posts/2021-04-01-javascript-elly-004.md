---
layout: single
title:  javascript-004 - Arrow Function은 무엇인가? 함수의 선언과 표현
categories: 
  - learning javascript by ellie
tags: 
  - javascript
---

## Function

- fundamenta lbuilding block in the program
- subprogram can be used multiple times
- performs a task or calculates a value

### Function declaration

- function name(param1, param2) { body... return; }
- one functon === one thing
- naming : doSomething, command, verb
- e.g. createCardAndPoint -> createCard, createPoint
- function is a object in javascript

```javascript
function printHello() {
  console.log('Hello');
}
printHello();

function log(message) {
  console.log(message);
}
log('Hello@');
```

- typescript playground : https://www.typescriptlang.org/play


```typescript
function(message: string): number {
    console.log(message);
    return 0;
}
```

## Parameters

- primitive parameters: passed by value
- object parameters: passed by reference

```javascript
function changeName(obj) {
  obj.name = 'coder';
}
const ellie = { name: 'ellie' };
changName(ellie);
console.log(ellie); // { name: 'coder' }
```

## Default parameters ( added in ES6 )

```javascript
function showMessage(message, from) {
  console.log(`${message} by ${from}`);
}
showMessage('Hi!'); // Hi! by undefined

function showMessage(message, from = 'unknwon') {
  console.log(`${message} by ${from}`);
}
showMessage('Hi!'); // Hi! by unknown
```

## Rest parameters ( added in ES6 )

```javascript
function printAll(...args) {
  for(let i=0; i<args.length; i++) {
    console.log(args[i]);
    // dream
    // coding
    // ellie
  }
  
  for(const arg of args) {
    console.log(arg);
  }
}
printAll('dream', 'coding', 'ellie');
```

## Local scope

```javascript
let globalMessage = 'global'; // global variable
function printMessage() {
  let message = 'hello';
  console.log(message); // local variable
  console.log(globalMessage); 
  function printAnother() {
    console.log(message);
    let childMessage = 'hello';
  }
}
printMessage();
console.log (message); // Uncaught ReferenceError
```

## Return a value

```javascript
functoin sum(a, b) {
  return a + b;
}
const result = sum(1, 2);
console.log (`sum: ${sum(1, 2)}`); // sum: 3
```

## Early return, early exit

```javascript
// bad case
function upgradeUser(user) {
  if(user.point > 10) {
    // long upgrade logic...
  }
}

// good case
function upgradeUser(user) {
  if(user.point <= 10) {
    return;
  }
  // long upgrade logic...
}
```

---

**NOTE**

## First-class function
- functions are treated like any other variable
- can be assigned as a value to variable
- can be passed as an argument to other functions
- can be returned by another function

---

## Function expression

- a function declaration can be called earlier than it is defined. (hoisted)
- a function expression is created when the execution reaches it

```javascript
const print = function() {
  console.log('print');
}
print(); // print
const printAgain = print;
printAgain(); // print
const sumAgain = sum;
console.log(sumAgain(1, 3));
```

## Callback function using function expression

```javascript
function randomQuiz(answer, printYes, printNo) {
  if(answer === 'love you') {
    printYes();
  } else {
    printNo();
  }
}

const printYes = function() {
  console.log('yes!');
};

const printNo = function print() {
  console.log('no!');
};

randomQuiz('wrong', printYes, printNo);
randomQuiz('love you', printYes, printNo);
```

## Arrow function

- always anonymous

```javascript
const simplePrint = function() {
  console.log('simplePrint');
};

const simplePrint = () => console.log('simplePrint');
const add = (a, b) => a + b;
```

## IIFE : Immediately Invoked Function Expression

```javascript
(function hello() {
  console.log('IIFE');
})();
```

## 참고
- [https://youtu.be/e_lU39U-5bQ](https://youtu.be/e_lU39U-5bQ)