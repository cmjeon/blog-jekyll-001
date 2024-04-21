---
layout: single
title: 정규표현식 실습해보자
categories: 
  - learn etc
tags: 
  - regex
---

# 정규표현식 실습해보자

정규표현식은 규칙에 맞는 문자열을 찾는 방법

/[A-Z]\w+/g : 대문자 A-Z 로 시작하는 아무 문자

앞 뒤의 / 는 정규표현식의 시작과 끝을 나타냄

g 위치는 flags g는 정규표현식에 맞는 모든 문자열을 찾는 것

flags 는 이번 내용에서 생략

[https://regexr.com/](https://regexr.com/){:target="_blank"} 에서 직접 해보자

아래에서 표현한 regex 는 모두 /{표현식}/g 의 표현식 부분에 들어감

## 기본사용법

찾고자 하는 문자를 그대로 입력하면 찾을 수 있음.

공백도 개수가 맞아야 함

```
- source
single and double

- regex
single

single and  double
```

^ 은 문자열 전체에서 첫 부분

$ 은 문자열 전체에서 마지막 부분

```
- source
http://www.naver.com

- regex
^http

com$
```

특수기호는 이스케이핑(\) 처리해야 찾을 수 있음

```
- source
$300

- regex
^\$
```

```
- source
hi^

- regex
\^$
```

```
- source
\\hello

- regex
^\\\\
```

. 은 모든 문자를 뜻함

```
- source
Regular expressions

- regex
.

.....
```

```
- source
O.K.

- regex
\..\.
```

## 대괄호

대괄호는 대괄호안의 문자중 하나를 찾음

```
- source
How do you do?

- regex
[oyu]

[dH].

[owy][yow]
```

대괄호 안의 - 는 범위를 뜻함

```
- source
ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz 0123456789

- regex
[C-K]

[CDEFGHIJK]

[C-Ka-d2-6]
```

대괄호 안의 ^ 은 not 을 의미함

```
- source
ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz 0123456789

- regex
[^CDghi45]

[^W-Z]
```

## 소괄호

소괄호는 표현된 단어 자체를 찾음

소괄호 안의 `|` 은 or 의 의미함

```
- source
Monday Tuesday Friday Thursday

- regex
(sda)

(on|ues|rida)

(Mon|Tues|Fri)day

..(id|esd|nd)ay
```

## 수량자 (Quantifier)

`*` : 앞의 문자가 0 ~ n 개 = {0, }

`+` : 앞의 문자가 1 ~ n 개 = {1, }

`?` : 앞의 문자가 0 ~ 1 개 = {0, 1}

```
- source
aaabc aabc abc bc

- regex
a*b

a+b

a?b
```

```
- source
-@- *** -- "*" -- *** -@-

- regex
.*

-A*-

[-@]*
```

```
- source
-@@@- * ** - - "*" -- * ** -@@@-

- regex
\*+

-@+-

[^ ]+
```

```
- source
-XX-@-XX-@@-XX-@@@-XX-@@@@-XX-@@-@@-

- regex
-X?XX?X

-@?@?@?-

[^@]@?@
```

## 중괄호

{} 중괄호는 수량자를 보다 정교하게 표현하기 위함

```
- source
One ring to bring them all and in the darkness bind them

-regex
.{5}

[els]{1,3}

[a-z]{3,}
```

수량자는 기본적으로 최대한의 매칭 결과를 표현함

```
- source
One ring to bring them all and in the darkness bind them

- regex
r.*
```

수량자 뒤의 ?는 앞의 수량자를 가장 적게 일치시킴 lazy quantifier

```
- source
One ring to bring them all and in the darkness bind them

- regex
r.*?

r.+?

r.??
```

```
- source
<div>test</div>  <div>test</div>

- regex
<div>.+<\/div>

<div>.+?<\/div>
```

\w 는 단어인 경우 = [A-z0-9_]

```
- source
A1 B2 c3 d_4 e:5 ffGG77--__--

- regex
\w

[A-z0-9_]

[a-z]\w*

\w{5}
```

\W 는  모든 단어가 아닌 경우

```
- source
AS _34:AS11.23  @#$ %12^*

- regex
\W

[^A-z0-9_]

\w
```

\s 는 공백인 경우, \S 는 공백이 아닌 경우

```
- source
Ere iron was found or tree was hewn,    When young was mountain under moon; Ere ring was made, or wrought was woe,    It walked the forests long ago.

- regex
\s

\S
```

\d 는 숫자인 경우, \D 는 숫자가 아닌 경우

```
- source
Page 123; published: 1234 id=12#24@112

- regex
\d

\D

[0-9]
```

\b 는 단어의 경계인 경우, \B 는 단어 경계가 아닌 경우

앞에 쓰이는 \b 는 단어의 앞, 뒤에 쓰이는 \b 는 단어의 뒤

```
- source
Ere iron was found or tree was hewn,    When young was mountain under moon; Ere ring was made, or wrought was woe,    It walked the forests long ago.

- regex
\b\w{0,2}

\w{0,2}\b

\B.\B
```

\A 는 문자열의 시작, \Z 는 문자열의 끝을 의미. 여러행이 경우도 처음과 마지막을 뜻함

```
- source
Ere iron was found or tree was hewn,    When young was mountain under moon; Ere ring was made, or wrought was woe,    It walked the forests long ago.

- regex
\A...

...\Z
```

(?=<pattern>) 은 선택에 활용이 되나 선택되지는 않음 

```
- source
AAAX---aaax---111

- regex
\w+(?=X)

\w+

\w+(?=\w)
```

# 항상 정규표현식이 옳은가?

If 문을 안써서 좋지만 복잡해지면 가독성이 낮아진다.

# 참고

- [https://www.youtube.com/watch?v=t3M6toIflyQ](https://www.youtube.com/watch?v=t3M6toIflyQ){:target="_blank"}
- [https://github.com/dream-ellie/regex](https://github.com/dream-ellie/regex){:target="_blank"}
- [https://regexone.com/](https://regexone.com/){:target="_blank"}
- [http://academy.dream-coding.com/](http://academy.dream-coding.com/){:target="_blank"}
- [https://www.youtube.com/watch?v=CjoDIgDOHA4](https://www.youtube.com/watch?v=CjoDIgDOHA4){:target="_blank"}
- [http://zvon.org/](http://zvon.org/){:target="_blank"}
