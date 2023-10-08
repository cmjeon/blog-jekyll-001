---
layout: single
title: "docusaurus Algolia 적용하기"
categories:
  - "docusaurus"
tags:
  - "정적사이트생성기(SSG)"
---

## 이 함수가 이해되는가?

add2(3) 의 결과는 무엇일까?

```javascript
const calcWith2 = (op) => (num) => op(2, num);

const add = (num1, num2) => num1 + num2;

const add2 = calcWith2(add);

add2(3);
```

정답은 5이다.

## 설명

add2() 는 add() 함수를 인자로 받는 calcWith2() 함수이다.

calcWith2() 함수의 인자로 들어가는 add() 함수는 2개의 파라미터를 받아 더해주는 함수이다.

add2(3) 를 하면 3 은 calcWith2() 함수로 전달된다.

calcWith2() 함수는 인자로 받은 함수 op 에 num 을 전달하고 그 함수에 2 와 num 을 전달하는 함수이다.

결국 calcWith2() 함수는 인자로 받은 함수 add() 에 3 을 전달하고, add(2, 3) 을 수행하도록 함수인 것이다.

add() 함수는 두 파라미터를 더하는 함수이다.

## 그래서?

이렇게 함수를 인자로 넣거나 리턴하는 함수를 고계함수라고 한다.

고계함수를 사용하면 함수를 감싼 함수를 만들 수 있다.

```javascript
const calc = (num1, num2, op) => op(num1, num2);

const add = (num1, num2) => num1 + num2;
const multiply = (num1, num2) => num1 * num2;

calc(2, 3, add);

calc(2, 3, multiply);
```

위 사례처럼 어떤 것을 계산하는 함수 calc() 함수를 만들고, 어떤 동작을 할지를 나중에 정할 수 있다.

뭔가 스프링의 DI 가 떠오르고 괜찮지 않는가?

## 이런 것도 있다.

이과생, 선착순 3명을 뽑아서 문자열로 출력해보기

```javascript
import _ from 'lodash';

class Student {
    constructor(name, division, major) {
        this.name = name;
        this.division = division;
        this.major = major;
    }
}

const students = [
    new Student("홍길동", "문과", "문학"),
    new Student("전우치", "이과", "기계"),
    new Student("임꺽정", "이과", "화학"),
    new Student("일지매", "문과", "언론"),
    new Student("장길산", "예체능", "체조"),
    new Student("연흥부", "이과", "컴퓨터"),
    new Student("연놀부", "이과", "수학"),
    new Student("옹고집", "문과", "경영"),
    new Student("이몽룡", "문과", "정치"),
    new Student("연오랑", "예체능", "사진")
];

// 함수형 프로그래밍 없이 작성하기
var filtered = [];
for (var i = 0; i < students.length; i++) {
    if (students[i].division === "이과") {
        filtered.push(students[i]);
    }
}
filtered = filtered.splice(0, 3);
var result = "";
for (var i = 0; i < filtered.length; i++) {
    result += `${filtered[i].name}(${filtered[i].major}) `;
}
console.log(result);

// 함수 컴비네이터로 작성하기
console.log(
    _(students)
        .filter((i) => i.division === '이과')
        .take(3)
        .map((i) => `${i.name}(${i.major})`)
        .reduce((sum, i) => sum + " " + i)
)
```

## 위에 나온 유명한 함수들 한 번 살펴보자

javascript Array 객체에 구현된 함수

## .filter

목록을 순회하면서 조건에 맞는 요소를 선택하여 목록에 담아준다.

```javascript
const filterdList = students.filter(student => student.division === '이과');

/*
[
  {
    "name": "전우치",
    "division": "이과",
    "major": "기계"
  },
  {
    "name": "임꺽정",
    "division": "이과",
    "major": "화학"
  },
  {
    "name": "연흥부",
    "division": "이과",
    "major": "컴퓨터"
  },
  {
    "name": "연놀부",
    "division": "이과",
    "major": "수학"
  }
]
 */
```

## .map

목록을 순회하면서 map 내부에 선언한 로직의 결과를 리턴해준다.

```javascript
const mappedList = students.map(student => `${student.name}(${student.major})`);

/*
[
  "홍길동(문학)",
  "전우치(기계)",
  "임꺽정(화학)",
  "일지매(언론)",
  "장길산(체조)",
  "연흥부(컴퓨터)",
  "연놀부(수학)",
  "옹고집(경영)",
  "이몽룡(정치)",
  "연오랑(사진)"
]
 */
```

## .reduce

목록을 순회하면서 redue 내부에 선언한 로직의 결과를 최종 결과에 계속 누적해서 리턴해준다.

```javascript
const reducedList = mappedStudents.reduce((text, student) => text + " " + `[${student.major}] ${student.name}`, "명단:");

/*
명단: 홍길동(문학) 전우치(기계) 임꺽정(화학) 일지매(언론) 장길산(체조) 연흥부(컴퓨터) 연놀부(수학) 옹고집(경영) 이몽룡(정치) 연오랑(사진)
 */
```

[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array)
에서 다른 메소드도 배워보자

## 참고

- [https://www.youtube.com/watch?v=jVG5jvOzu9Y](https://www.youtube.com/watch?v=jVG5jvOzu9Y)
- [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array)
