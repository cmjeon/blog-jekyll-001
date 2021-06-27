---
layout: single
title:  javascript-007 - 배열 제대로 알고 쓰자. 자바스크립트 배열 개념과 APIs 총정리
categories: 
  - learning javascript by ellie
tags: 
  - javascript
---

## Object

### Literals and properties

```javascript
// primitive
function print(name, age);
functon print(name, age) {
  console.log(name);
  console.log(age);
}

const name = 'ellie';
const age = 4;
print(name, age);

// object
function print(person) {
  console.log(person.name);
  console.log(person.age);
}

const ellie = { name : 'ellie', age: 4 };
print(ellie);

const obj1 = {}; // `object literal` syntax
const obj2 = new Object(); // `object constructor` syntax

ellie.hasJob = true; // 동적으로 property를 할당 가능
console.log(ellie.hasJob);

delete ellie.hasJob // 삭제 가능
console.log(ellie.hasJob);
```

### Computed properties

```javascript
console.log(ellie.name); // ellie : property name 을 알고 있을 때
console.log(ellie['name']); // ellie : property name 을 알 수 없을 때 
console.log(ellie[name]); // undefined

ellie['hasJob'] = true;
console.log(ellie.hasJob); // true

// ellie.name vs ellie.['name']
function printValue1(obj, key) {
  console.log(obj.key);
}

function printValue2(obj, key) {
  console.log(obj[key]);
}

printValue1(ellie, 'name'); // undefined
printValue2(ellie, 'name'); // ellie
printValue2(ellie, 'age'); // 4
```

### Property value shorthand

```javascript
const person1 = { name: 'bob', age: 2 };
const person2 = { name: 'steve', age: 3 };
const person3 = { name: 'dave', age: 4 };
const person4 = makePerson('ellie', 30);

function makePerson(name, age) {
  return {
    name,
    age,
  };
}

console.log(person4);
```



## 참고
- [https://youtu.be/1Lbr29tzAA8](https://youtu.be/1Lbr29tzAA8)