---
layout: single
title:  javascript-002 - 데이터타입, data types, let vs var, hoisting
categories: 
  - learning javascript by ellie
tags: 
  - javascript
---

## Varilable, rw(read/write)

- let : ES6에서 추가됨
- mutable

```javascript
let name = 'ellie';
console.log(name); // ellie
name = 'hello';
console.log(name); // hello
```

## block scope

- let은 block scope를 가짐

```javascript
let globalName = 'global name';
{
  let name = 'ellie';
  console.log(name); // ellie
  name = 'hello';
  console.log(name); // hello
}
console.log(name); // 
console.log(globalName); // global name
```

## var

```javascript
console.log(age); // undefined
age = 4;
console.log(age); // 4
var age;
```

- hoisting : 호이스팅은 선언한 위치에 상관없이 선언을 가장 위로 올려주는 것
- block scope : block scope를 따지지 않고 어디에서나 사용가능

## Constant, r(readonly)

- 한 번 선언하면 변경할 수 없음
- immutable
- favor immutable data type always for a few reasons:
  - security
  - thread safety
  - reduce human mistakes

```javascript
const daysInWeek = 7;
const maxNumber = 5;
```

---

**NOTE**

- Immutable data types: primitive types, frozen objects (i.e. object.freeze())
- Mutable data types: all objects by default are mutable in JS

---

## Variable types

- primitive, single item : number, string, boolean, null, undefined, symbol
- object, box container
- function, first-class function : 함수를 변수에 할당, 인자로 전달, 리턴타입으로 선언 가능

## number

```javascript
const count = 17; // interger
const size = 17.1; // decimal number
console.log(`value: ${count}, type: ${typeof count}`); // value: 17, type number
console.log(`value: ${size}, type: ${typeof size}`); // value: 17.1, type number

// special numeric values
const infinity = 1 / 0;
const nagativeInfinity = -1 / 0;
const nAn = 'not a number' / 2; 
console.log(infinity); // Infinity
console.log(nagativeInfinity); // -Infinity
console.log(nAn); // NaN

// bigInt
const bigInt = 1234567890123456789012345678901234567890n;
console.log(`value: ${bigInt}, type: ${typeof bigInt}`); // value: 1234567890123456789012345678901234567890, type bigint
```

## string

```javascript
const char = 'c';
const brendan = 'brendan';
const greeting = 'hello ' + brendan;
console.log(`value: ${greeting}, type: ${typeof greeting}`); // value: hello brendan, type string
const helloBob = `hi ${brendan}!`; // template literals (string)
console.log(`value: ${helloBob}, type: ${typeof helloBob}`); // value: hi brendan, type string
console.log('value: ' + helloBob + ' type: ' + typeof helloBob);
```

## boolean

- false : 0, null, undefined, NaN, ''
- true : any other value

```javascript
const canRead = true;
const test = 3 < 1; // false
console.log(`value: ${canRead}, type: ${typeof canRead}`); // value: true, type boolean
console.log(`value: ${test}, type: ${typeof test}`); // value: false, type boolean
```

## null

- null로 할당된 경우

```javascript
let nothing = null;
console.log(`value: ${nothing}, type: ${typeof nothing}`);
```

## undefined

- 값이 할당되지 않은 경우

```javascript
let x;
console.log(`value: ${x}, type: ${typeof x}`);
```

## symbol

- create unique identifiers for objects

```javascript
const symbol1 = Symbol('id');
const symbol2 = Symbol('id');
console.log(symbol1 === symbol2); // false

const gSymbol1 = Symbol.for('id');
const gSymbol2 = Symbol.for('id');
console.log(gSymbol1 === gSymbol2); // true
console.log(`value: ${symbol1}, type: ${typeof symbol1}`); // Uncaught TypeError
console.log(`value: ${symbol1.description}, type: ${typeof symbol1}`); // value: id, type: symbol
```

## object, real-life object, data structure

```javascript
const ellie = { name: 'ellie', age: 20 };
ellie.age = 21;
```

## Dynamic typing : dynamically typed language

- javascript는 런타임환경에서 타입이 변경될 수 있음
- typescript가 나오게되는 배경(타입스크립트는 babel이 필요)

```javascript
let text = 'hello';
console.log(text.charAt(0)); // h
console.log(`value: ${text}, type: ${typeof text}`); // value: hello, type: string
text = 1;
console.log(`value: ${text}, type: ${typeof text}`); // value: 1, type: number
text = '7' + 5;
console.log(`value: ${text}, type: ${typeof text}`); // value: 75, type: string
text = '8' + '2';
console.log(`value: ${text}, type: ${typeof text}`); // value: 4, type: number
console.log(text.charAt(0)); // Uncaught TypeError
```

## 참고
- [https://youtu.be/OCCpGh4ujb8]