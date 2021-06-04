---
layout: single
title:  javascript-005 - 클래스와 오브젝트의 차이점, 객체지향 언어 클래스 정리
categories: 
  - learning javascript by ellie
tags: 
  - javascript
---

## class & object

### class

- fields, methods로 구성
- template
- declare once
- no data in

### object

- instance of a class
- created many times
- data in

## Object-oriented programming

- class: template
- object: instance of a class

### Javascript classes

- introdued in ES6
- syntactical sugar over prototype-based inheritance

## Clas declarations

```javascript
class Person {
  // constructor
  constructor(name, age) {
    // fields
    this.name = name;
    this.age = age;
  }

  // methods
  speak() {
    console.log(`${this.name}: hello!`);
  }
}

const ellie = new Person('ellie', 20);
console.log(ellie.name); // ellie
console.log(ellie.age); // 20
ellie.speak(); // ellie: hello!
```

## Getters and setters

```javascript
class User {
  constructor(firstName, lastName, age) {
    this.firstName = firstName;
    this.lastNAme = lastName;
    this.age = age;
  }

  get age() {
    return this._age;
  }

  set age(value) {
    // this._age = value
    this._age = value < 0 ? 0 : value;
  }
}

const user1 = new User('Steve', 'Jobs', -1);
console.log(user1.age); // -1
```

## Fields (public, private)

- Too soon!

```javascript
class Experiment {
  publicField = 2;
  #privateField = 0;
}
const expeiment = new Experiment();
console.log(experiment.publicField);
console.log(experiment.privateField);

```

## Static properties and methods

- Too soon!

```javascript
class Article {
  static publisher = 'Dream Coding';
  constructor(articleNumber) {
    this.articleNumber = articleNumber;
  }

  static printPublisher() {
    console.log(Article.publisher);
  }
}

const article1 = new Article(1);
const article2 = new Article(2);
console.log(article1.publisher);
console.log(Article.publisher); // Dream Coding
Article.printPublisher(); // Dream Coding
```

## Inheritance & Polymo

```javascript
class Shape {
  constructor(width, height, color) {
    this.width = width;
    this.height = height;
    this.color = color;
  }

  draw() {
    console.log(`drawing ${this.color} color of`);
  }

  getArea() {
    return this.width * this.height;
  }
}

class Rectangle extends Shape {}
const rectangle = new Rectangle(20, 20, 'blue');
rectangle.draw(); // draw blue color of
console.log(rectangle.getArea()); // 400

class Triangle extends Shape {
  draw() {
    super.draw();
    console.log('▲');
  }
  getArea(){
    return (this.width * this.height) / 2;
  }
}
const triangle = new Triangle(20, 20, 'red');
triangle.draw(); // draw red color of
console.log(rectangle.getArea()); //200
```

## Class checking: instanceOf

```javascript
console.log(rectangle instanceof Rectangle); // true
console.log(triangle instanceof Rectangle); // false
console.log(triangle instanceof Triangle); // true
console.log(triangle instanceof Shape); // true
console.log(triangle instanceof Object); // true
console.log(triangle.toString()); // [object Object]
```

## 참고
- [https://youtu.be/_DLhUBWsRtw]