---
layout: single
title:  javascript-008 - 유용한 10가지 배열 함수들. Array APIs 총정리
categories: 
  - learning javascript by ellie
tags: 
  - javascript
---

## Array APIs


### Q1. make a string out of an array

```javascript
// join
const fruits = ['apple', 'banana', 'orange'];
console.log(fruits.join()); // apple,banana,orange
console.log(fruits.join(':')); // apple:banana:orange
```

### Q2. make an array out of a string

```javascript
// split
const fruits = '🍎, 🥝, 🍌, 🍒';
console.log(fruits.split(',')); // ["🍎", "🥝", "🍌", "🍒"]
console.log(fruits.split(',', 2)); // ["🍎", "🥝"]
console.log(fruits.split()); // ["🍎, 🥝, 🍌, 🍒"]
```

### Q3. make this array look like this: [5, 4, 3, 2, 1]

```javascript
// reverse
const array = [1, 2, 3, 4, 5];
const result = array.reverse();
console.log(result); // [5, 4, 3, 2, 1]
console.log(array); // [5, 4, 3, 2, 1]
```

### Q4. make new array without the first two elements

```javascript
// splice
const array = [1, 2, 3, 4, 5];
const result = array.splice(0, 2);
console.log(result); // [1, 2]
console.log(array); // [3, 4, 5]

// slice
const array = [1, 2, 3, 4, 5];
const result2 = array.slice(2, 5); // this is [start index, end index + 1]
console.log(result2); // [3, 4, 5]
console.log(array); // [1, 2, 3, 4, 5]
```

---

**NOTE**

```javascript
class Student {
  constructor(name, age, enrolled, score) {
    this.name = name;
    this.age = age;
    this.enrolled = enrolled;
    this.score = score;
  }
}
const students = [
  new Student('A', 29, true, 45),
  new Student('B', 28, false, 80),
  new Student('C', 30, true, 90),
  new Student('D', 40, false, 66),
  new Student('E', 18, true, 88),
];
```

---

### Q5. find a student with the score 90

```javascript
// find
const result = students.find(function(student, index) {
  console.log(student, index); // 5명의 학생 객체 출력, index
  return student.score === 90;
});
// const result = students.find((student) => student.score === 90);
console.log(result); // Student { name: "C", age: 30, enrolled: true, score: 90 }
```

### Q6. make an array of enrolled students

```javascript
// filter
const result = students.filter(function(student) {
  return student.enrolled;
});
// const result = students.filter((student) => student.enrolled);
console.log(result); // [ Student, Student, Student ]
```

### Q7. make an array containing only the students' scores

- result should be: [45, 80, 90, 66, 88]

```javascript
// map
const result = students.map(function(student) {
  return student.score;
});
// const result = students.map((student) => student.score);
console.log(result); // [ 45, 80, 90, 66, 88 ]
```

### Q8. check if there is a student with the score lower than 50

```javascript
// some
const result = students.some(function(student) {
  return student.score < 50;
});
// const result = student.some((student) => student.score < 50);
console.log(result); // true

// every
const result = !students.every(function(student) {
  return student.score >= 50;
});
// const result = !students.every((student) => student.score >= 50);
console.log(result) // true
```

### Q9. compute students' average score

```javascript
// reduce, reduceRight
// 배열의 값마다 콜백함수가 실행됨
// 이 번 콜백함수가 실행 될 때, 앞에 실행한 콜백함수의 결과가 prev, 배열의 값이 curr로 들어옴
// '0' 은 initial value
const result = students.reduce(function(prev, curr) {
  console.log('------');
  console.log(prev);
  console.log(curr);
  return prev + curr.score;
}, 0);
// const result = students.reduce((prev, curr) => prev + curr.score, 0)
console.log(result / students.length); // 73.8
```

### Q10. make a string containing all the scores

- result should be: '45, 80, 90, 66, 88'

```javascript
// map, join
const result = students
  .map((student) => student.score)
  .filter((score) => score >= 50)
  .join();
console.log(result); // 80,90,66,88
```

### Bonus! do Q10 sorted in ascending order

- result should be: '45, 66, 80, 88, 90'

```javascript
// ascending
const result = students
  .map((student) => student.score)
  .sort((a, b) => a - b)
  .join()
console.log(result); // 45,66,80,88,90

// descending
const result = students
  .map((student) => student.score)
  .sort((a, b) => b - a)
  .join()
console.log(result); // 90,88,80,66,45
```


## 참고
- [https://youtu.be/3CUjtKJ7PJg](https://youtu.be/3CUjtKJ7PJg)