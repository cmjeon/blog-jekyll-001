---
layout: single
title:  javascript-009 - JSON 개념 정리와 활용방법 및 유용한 사이트 공유 Javascript JSON
categories: 
  - learning javascript by ellie
tags: 
  - javascript
  - json
---

## HTTP

Client, Server 가 Request, Response 하는 프로토콜

ajax : asynchronous javascript and XML

XHR : XMLHttpRequest ajax 요청을 생성하는 javascript API

XML 은 HTML 와 같은 마크업 언어

클라이언트와 서버가 데이터를 주고받는 방식이 XML, 그러나 JSON 도 있음

XMLHttpRequest 앞에 XML 은 당시에 XML 을 감안하여 개발한 API

## JSON

JSON : JavaScript Object Notation

Key 와 Value 로 이루어진 가벼운 포맷

- 가장 간단함
- 가벼운 텍스트(String) 기반 구조
- 읽기 쉽다
- 키-밸류 형식
- 직렬화 할 수 있음
- 개발언어와 플랫폼에 무관함

json 은 직렬화와 역직렬화가 핵심임

object - stringify -> string
string - parse -> object

### Object to JSON

```js
// JSON.stringify()
let json = JSON.stringify(true);
console.log(json); // true

json = JSON.stringify(['apple', 'banana']);
console.log(json); // ["apple","banana"]

const rabbit = {
  name: 'tori',
  color: 'white',
  size: null,
  birthDate: new Date(),
  symbol: Symbol('id'),
  jump: () => {
    console.log(`${name} can jump`);
  },
};

json = JSON.stringify(rabbit);
console.log(json); // symbol, function 은 제외된 string 으로 변환

json = JSON.stringify(rabbit, ['name', 'color']);
console.log(json); // name, color 만 stirng 으로 변환

json = JSON.stringify(rabbit, (key, value) => {
  console.log(`key: ${key}, value: ${value}`);
  return value;
});
console.log(json) // rabbit 과 그 안의 key, value 가 순차적으로 key, value 로 처리됨
```

### JSON to Object

```js
// JSON.parse()
const rabbit = {
  name: 'tori',
  color: 'white',
  size: null,
  birthDate: new Date(),
  symbol: Symbol('id'),
  jump: () => {
    console.log(`${name} can jump`);
  },
};

let json = JSON.stringify(rabbit);

const obj = JSON.parse(json);
console.log(obj);
rabbit.jump(); // 오류발생 : stringify, parse 를 거치면 function 은 제외됨

console.log(rabbit.birthDate.getDate()); // 오늘 날짜 표시
console.log(obj.birthDate.getDate()); // 오류발생 : obj.birthDate 는 더이상 Date 객체가 아님

const obj2 =JSON.parse(json, (key, value) => {
  console.log(`key: ${key}, value: ${value}`);
  return key === 'birthDate' ? new Date(value) : value;
});

console.log(obj2.birthDate.getDate()); // 정상 : key 가 birthDate 인 경우 Date 객체를 생성하였음
```

## 더 공부할 수 있는 사이트

[MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON)
[JavaScript.info](https://javascript.info/json)
[JavaScript.info 한국어](https://ko.javascript.info/json)

## 유용한 사이트

[JSON Diff checker](http://www.jsondiff.com)
[JSON Beautifier/editor](https://jsonbeautifier.org)
[JSON Parser](https://jsonparser.org)
[JSON Validator](https://tools.learningcontainer.com/json-validator)

## 참고
- [https://youtu.be/FN_D4Ihs3LE]
