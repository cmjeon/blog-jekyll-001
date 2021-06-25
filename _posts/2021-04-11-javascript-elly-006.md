---
layout: single
title:  javascript-006 - 오브젝트 넌 뭐니?
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
console.log(person4);

function makePerson(name, age) {
  return {
    name,
    age,
  };
} 
```

### Constructor Function

```javascript
const person5 = new Person('ellie', 20);
console.log(person5);
// object 를 리턴하는 함수
function Person(name, age) {
  // this = {};
  this.name = name;
  this.age = age;
  // return this;
}
```

### In operator: property existence check (key in obj)

```javascript
console.log('name' in ellie); // true
console.log('age' in ellie); // true
console.log('random' in ellie); // false
console.log(ellie.random); // undefinced
```

### for..in vs for..of

#### for (key in obj)

```javascript
for(key in ellie) {
  console.log(key); // name, age, hasJob
}
```

#### for (value of iterable)

```javascript
const array = [1, 2, 4, 5];

// use for loop
for(let i=0; i<array.length; i++){
  console.log(array[i]);
}

// use for...of
for(value of array) {
  console.log(value);
}
```

### Fun cloning

```javascript
const user = { name: 'ellie', age: '20' };
const user2 = user;
user2.name = 'coder';
console.log(user.name); // coder, why?

// old way
const user = { name: 'ellie', age: '20' };
const user3 = {}
for(key in user) {
  user3[key] = user[key];
}
console.log(user3); // { name: 'ellie', age: '20' }

// new way
const user4 = Object.assign({}, user);
console.log(user4); // { name: 'ellie', age: '20' }

// another example
const fruit1 = { color: 'red' };
const fruit2 = { color: 'blue', size: 'big' };
const mixed = Object.assign({}, fruit1, fruit2);
console.log(mexed.color); // blue, 뒤의 값이 앞의 값을 덮어씌움
console.log(mexed.size); // big
```

## 참고
- [https://youtu.be/1Lbr29tzAA8](https://youtu.be/1Lbr29tzAA8)