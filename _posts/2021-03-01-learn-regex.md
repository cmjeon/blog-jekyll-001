---
layout: single
title: 정규표현식 알아보기
categories: 
  - learn etc
tags: 
  - regex
---

## regex 정규표현식

- REGular EXpress
- 개발자라면 누구나 기본적으로? 알고 있고 다양한 곳에서 쓰이고 있다.
- 문자열의 패턴을 찾는데 쓰인다.
- 어지간한 언어에서 지원하고 있다.
- 정규표현식을 보면 난해할 수 있는데, 한번만 배워두면 두고두고 써먹을 수 있다.
~~~
/regex?/i
// /와 /는 정규표현식 선언
// regex?는 정규표현식
// i는 flag
~~~

## 정규표현식 학습사이트

- [https://regexr.com/5mhou](https://regexr.com/5mhou){:target="_blank"} 에 접속
- 상단이 정규표현식 입력창, 하단이 샘플 문자열
- 상단에 정규표현식을 입력하면 하단에 해당하는 문자열이 하이라이트 됨
- Explain 확인

## 정규표현식 사용 기본기

### Group and Ranges

~~~
| 또는
/Hi|Hello/gm

() 그룹
/(Hi|Hello)/gm
/(Hi|Hello)|(And)/gm : Hi 이거나 Hello인 그룹1, And인 그룹2
/gr(e|a)y/gm : 그룹지정


[] 문자셋, 괄호안의 어떤 문자든
/gr[aed]y/gm : gr안에 aed 중 어떤 문자든 들어오고 y로 끝나는 문자열
/[a-zA-Z0-9]/gm : a~z, A~Z, 0~9 안에 들어오는 문자열

[^] 부정 문자셋, 괄호안의 어떤 문자가 아닌 문자열
/[^a-zA-Z0-9]/gm : a~z, A~Z, 0~9 안에 들어오는 문자열이 아닌 문자열

(?:) 찾지만 기억하지는 않음
/gr(?:e|a)y/gm : 그룹미지정
~~~

### Quantifiers

~~~
? 없거나 있거나(zero or one)
/gra?y/gm

* 없거나 있거나 많거나(zero or many)
/gra*y/gm

+ 있거나 많거나(zero or many)
/gra+y/gm

{n} n번 반복
/gra{2}y/gm

{min,} 최소
/gra{1,}y/gm

{min, max} 최소, 그리고 최대
/gra{2,3}y/gm
~~~

### Boundary-type

~~~
\b 단어 경계
/\bYa/gm : 문자열 앞부분에서 쓰이는 Ya
/Ya\b/gm : 문자열 끝부분에서 쓰이는 Ya

\B 단어 경계가 아님
/\BYa/gm : 문자열 앞부분에서 쓰이는 Ya를 제외한 문자열
/Ya\B/gm : 문자열 끝부분에서 쓰이는 Ya를 제외한 문자열

^ 문장의 시작
/^Ya/gm : Ya로 시작하는 문장에 있는 Ya

$ 문장의 끝
/Ya$/gm : Ya로 끝나는 문장에 있는 Ya
~~~

### Character classes

~~~
\ 특수 문자가 아닌 문자. 이스케이프 문자
/\./gm : "." 찾기
/\[\]\{\}/gm : "[", "]", "{", "}" 찾기

. 어떤 글자(줄바꿈 문자 제외)
/./gm

\d digit 숫자
/\d/gm

\D digit 숫자 아님
/\D/gm

\w word 문자
/\w/gm

\W word 문자 아님
/\W/gm

\s space 공백
/\s/gm

\S space 공백 아님
/\S/gm
~~~

## 연습하기

### 전화번호 선택하기

~~~
010-898-0893
010 898 0893
010.989.0893
010-405-3412
02-878-8888

/\d{2,3}[- .]\d{3}[- .]\d{4}/gm
~~~

### 이메일 선택하기

~~~
dream.coder.ellie@gmail.com
hello@daum.net
hello@daum.co.kr

/[a-zA-Z0-9._+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9.]+/gm
~~~

### youtu.be URL 선택하기

~~~
https://www.youtu.be/-ZClicWm0zM
https://youtu.be/-ZClicWm0zM
youtu.be/-ZClicWm0zM

/(https?:\/\/)?(www\.)?youtu.be\/([a-zA-Z0-9-]{11})/gm
~~~

## javascript에서 활용하기

~~~javascript
const regex = /(?:https?:\/\/)?(?:www\.)?youtu.be\/([a-zA-Z0-9-]{11})/;
const url = 'https://www.youtu.be/-ZClicWm0zM'
cosnt result = url.match(regex);
console.log(result[0]); // 전체 문자열
console.log(result[1]); // 매칭되는 그룹의 문자열
~~~

## 참고
- [https://www.youtube.com/watch?v=t3M6toIflyQ](https://www.youtube.com/watch?v=t3M6toIflyQ){:target="_blank"}
- [https://github.com/dream-ellie/regex](https://github.com/dream-ellie/regex){:target="_blank"}
- [https://regexone.com/](https://regexone.com/){:target="_blank"}
- [http://academy.dream-coding.com/](http://academy.dream-coding.com/){:target="_blank"}
