---
layout: single
title: 함수형 프로그래밍 알아보기
categories: 
  - learn etc
tags: 
  - functional programming
---

## 함수형 프로그래밍이란?

- 인풋과 아웃풋이 있다.
- 외부환경으로부터 독립적이다.
- 같은 인풋으로는 같은 아웃풋을 만들어내고, 다른 함수에 결과물 외에는 다른 영향을 주지 않는다. (순수함수이다)
- 따라서 부작용에 의한 문제로부터 보다 자유롭다.

## 서로 다른 쓰레드가 한 변수를 고칠 때 발생하는 문제점

- 변수에 lock 을 걸거나, synchronized 방법으로 변수를 동기화한다. -> 아주 주의를 기울여 코딩해야함
- 함수형 프로그래밍은 외부 변수가 없으므로 병렬로 작업이 가능

## 함수형 프로그래밍은 이렇게 작업한다.

- 함수의 동작에 의한 변수의 부수적인 값 변경을 원천 배제한다.
- 외부의 변수를 이용하더라도 사본을 복사해서 작업한다.

## 그러면 값이 변하지 않는 프로그램이 무슨 소용이 있나?

- 카카오가 초콜렛이 되지 않는 공장은 필요없다.
- 값을 아예 바꾸지 않는 것이 아니고 특정 범위에서라도 부수적인 효과없이 프로그램을 짜는 것이 함수형 프로그램이다.

## 함수형 프로그래밍의 특징

### 함수형 프로그래밍은 '선언형'이다

- 함수형 프로그래밍은 '선언형'이다.
  - '명령형' 프로그래밍 > 너는 이것을, 너는 저것을 해서 그것을 만들어내라.
  - '선언형' 프로그래밍 > 이걸하고, 저걸하면 그것이 된다.

### 함수도 값이다

- 함수는 '행위'가 아닌 '값'이다
- 함수를 함수형이나 변수형으로 선언가능

```javascript
//함수형
function say_it(given) {
  console.log(given);
}
//변수형
var say_it = function(given) {
  console.log(given);
}
//es6 변수형
const say_it = (given) => console.log(given);
```

- 단순 호출 가능, 콜백 함수로 호출 가능

```javascript
say_it('안녕');
// 안녕
['aaa','bbb','cccc'].foreach(say_it);
// aaa
// bbb
// ccc
```
- 함수 자체로는 손에 만져지는 결과가 나오는 것은 아니지만, 인자만 넣어주면 절대로 예측가능한 값을 리턴한다는 점에서 값으로 바라볼 수 있다.
- 그러면 함수를 값처럼 바라본다는게 무슨 의미가 있나?

### 고계함수

- 함수가 값으로 볼 수 있다면 함수에서 다른 함수를 인자로 넣거나 결과로 받을 수 있다.
- 인자에 함수를 넣거나 결과값으로 함수를 반환하는 함수를 '고계함수'라고 한다.

```javascript
const calc = (num1, num2, op) => op(num1, num2);

const add = (num1, num2) => num1 + num2;
const multiply = (num1, num2) => num1 * num2;
const power = (num1, num2) => Math.pow(num2, num1);

calc(2, 3, add);
// 2 + 3 = 5
calc(2, 3, multiply);
// 2 * 3 = 6
calc(2, 3, power);
// 3 ^ 2 = 8
```

- 함수를 받아서 함수를 반환하는 calcWith2 함수

```javascript
const calcWith2 = (op) => (num) => op(2, num);
// calcWith2는 어떤 함수를 인자로 받고, num을 인자로 받아 인자로 받은 함수에 2와 num을 넣은 결과를 리턴하는 함수를 반환한다.
const add = (num1, num2) => num1 + num2;
// add는 num1, num2를 받아 그 둘을 더하는 함수이다.
const add2 = calcWith2(add);
// add2는 add를 받은 calcWith2 함수이다.
add2(3);
// 5

const multiply = (num1, num2) => num1 * num2;
const multyply2 = calcWith2(multiply);
multiply(3);
// 6
```
- 이렇게 프로그램이 돌아가는 중에도 함수를 만들 수 있게 된다.

### 커링

- 커링은 여러 인자를 받는 함수에 일부 인자만 넣어서 나머지 인자를 받는 다른 함수를 만들어 내는 함수형 프로그래밍 기법이다.

~~~javascript
var add_curry = (num1) => (num2) => num1 + num2;
// add_curry는 num1을 인자로 받고, num2를 인자로 받아서 num1과 num2를 더한 결과를 리턴하는 함수를 반환한다.
const add2 = add_curry(2);
// add2는 2를 인자로 받고, num2를 인자로 받으며, 2 + num2한 결과를 리턴하는 함수를 반환한다.
add2(9);
// 11
~~~
- bind함수나 lodash 라이브러리로 javascript에서 커링을 보다 제대로 활용할 수 있다.

### 함수 컴비네이터

- 다음 배열에서 이과생, 선착순 3명을 뽑아서 문자열로 출력해보자

~~~javascript
var students = [
  new Student("홍길동", "문과", "문학"),
  new Student("전우치", "이과", "기계"),
  new Student("임꺽정", "이과", "화학"),
  new Student("일지매", "문과", "언론"),
  new Student("장길산", "예체능", "체조"),
  new Student("연흥부", "이과", "컴퓨터"),
  new Student("연놀부", "이과", "수학"),
  new Student("옹고집", "문과", "경영"),
  new Student("이몽룡", "문과", "정치"),
  new Student("연오랑", "예체능", "사진"),
]
// 예상 결과
// 전우치(기계) 임꺽정(화학) 연흥부(컴퓨터) 
~~~

- 비함수형 코드 예시

~~~javascript
var filtered = [];
for(var i=0; i<students.length; i++) {
  if(students[i].division === "이과") {
    filtered.push(students[i]);
  }
}
filtered = filtered.splice(0, 3);
var result = "";
for(var i=0; i<filtered.length; i++) {
  result += `${filtered[i].name}(${filtered[i].major}) `;
}
console.log(result);
// 전우치(기계) 임꺽정(화학) 연흥부(컴퓨터) 
~~~

- 함수형 코드 예시

~~~javascript
// lodash 활용
console.log(
  _(students)
  .filter((i) => i.division === '이과')
  // 인자로 주어진 조건에 맞는 요소를 모아 리스트로 반환
  .take(3)
  // 인자로 주어진 갯수만큼 앞에서 뽑아서 반환
  .map((i) => `${i.name}(${i.major})`)
  // 각 요소들을 인자로 넣어서 그 결과물 형식으로 반환
  .reduce((sum, i) => sum + " " + i)
  // 결과물을 받아서 기존 문자열과 더해서 문자열로 반환
)
// 전우치(기계) 임꺽정(화학) 연흥부(컴퓨터) 
~~~

## 참고
- [https://www.youtube.com/watch?v=jVG5jvOzu9Y](https://www.youtube.com/watch?v=jVG5jvOzu9Y){:target="_blank"}
