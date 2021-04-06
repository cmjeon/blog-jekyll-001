---
layout: single
title: 프로그래밍 기초 in Python 배우기 002
categories: 
  - learn python
tags: 
  - python
---

## 파이썬 자료형

### 문자열

- 문자들의 집합

~~~python
# 큰 따옴표, 작은 따옴표로 표현
"python"
'python'
"""python 3"""
'''python 3'''

# 큰 따옴표안에 작은 따옴표
"'hello'"

# 작은 따옴표안에 큰 따옴표
'"HELLO"'

# 백슬래시
"\'Hello\'"

# 줄바꾸기 \n 사용
"Life is \n so cool"

# 줄바꾸기 """, ''' 사용
"""
Life is
so cool
"""
'''
Life is
so cool
'''
~~~

### 문자열 연산

~~~python
# 문자열 연결
> a = 'Life is'
> b = 'so cool'
> a + b
'Life is so cool'

# 문자열 곱하기
> a = "Love"
> a * 2
'LoveLove'
~~~

### 문자열 인덱싱과 슬라이싱

~~~python
# 문자열 인덱스
> a = 'Life is so cool'
> a[6]
s
> a[-1]
l

# 문자열 슬라이싱
> a = 'Life is so cool'
> a[0:3] # 인덱스 0에서 3앞까지의 문자열
Lif
> a[6:]
s so cool
> a[:10]
Life is so;

# 문자열 나누기
> a = "2021030312:12:00"
> year = a[:4]
> day = a[4:8]
> time = a[8:]
> year
'2021'
> day
'0303'
> time
'12:12:00'
~~~

### 문자열 포매팅

~~~python
# 숫자
> "I went %d days ago." % 3

# 문자열
> "I went %s days ago." % "three"

# 숫자, 문자열
number = 3
unit = "weeks"
> "I went %d $s ago." $ (number, unit)
~~~

#### 문자열 포맷 코드

코드 | 설명
:---|:---
%s | 문자열(String)
%c | 문자 1개(character)
%d | 정수(Integer)
%f | 부동소수(floating-point)
%o | 8진수
%x | 16진수
%% | Literal % (문자 % 자체)

#### 포맷 코드와 숫자

~~~python
# 정렬과 공백
> "%10s" % "hi"
'        hi'

# 소수점
> "%0.4f" % 3.42134234
'3.4213'

# 조합
> "%10.4f" % 3.42134234
'    3.4213'
~~~

#### format 함수를 사용한 포맷팅

~~~python
# 값 대입하기
> "I went {0} {1} ago.".format(3, "weeks")
'I went 3 weeks ago.'

# 변수명으로 대입하기
> "I went {number} {unit} ago.".format(number=3, unit="weeks")
'I went 3 weeks ago.'

# 정렬 포맷팅
> "{0:<5}".format("hi")
'hi   '
> "{0:>5}".format("ago")
'  ago'
> "{0:^8}".format("hi")
'  went  '

# 소수점 표현
> pi = 3.42134234
> "{0:0.3f}".format(pi)
'3.421'

# 중괄호
> "{{ brace }}".format()
'{ brace }'
~~~

#### f 문자열 포매팅

- python 3.6 부터 f 문자열 포매팅 사용가능

~~~python
> name = '홍길동'
> age = 30
> f'나의 이름은 {name}입니다. 나이는 {age}입니다.'
'나의 이름은 홍길동입니다. 나이는 30입니다.'

> d = {'name':'홍길동', 'age':30}
> f'나의 이름은 {d["name"]}입니다. 나이는 {d["age"]}입니다.'
'나의 이름은 홍길동입니다. 나이는 30입니다.'

> f'{"hi":<6}'  # 왼쪽 정렬
'hi    '
> f'{"hi":>6}'  # 오른쪽 정렬
'    hi'
> f'{"hi":^6}'  # 가운데 정렬
'  hi  '

> f'{"hi":=^6}'  # 가운데 정렬하고 '=' 문자로 공백 채우기
'==hi=='
> f'{"hi":!<6}'  # 왼쪽 정렬하고 '!' 문자로 공백 채우기
'hi!!!!'

pi = 3.42134234
> f'{y:0.3f}'  # 소수점 4자리까지만 표현
'3.421'
> f'{y:6.3f}'  # 소수점 4자리까지 표현하고 총 자리수를 10으로 맞춤
' 3.421'

> f'{{ brace }}'
'{ brace }'
~~~

### 문자열 관련 함수들

~~~python
# 문자 개수
> "tree".count('e')
2

# 문자 위치 find
> "Life is so cool".find('f')
2
> "Life is so cool".find('k')
-1

# 문자 위치 index
> "Life is so cool".index('s')
6
> "Life is so cool".index('d')
-1

# 문자열 삽입 join
> ':'.join('life')
'l:i:f:e'
> ":".join(['l', 'i', 'f', 'e'])
'l:i:f:e'

# case 변환
> small = "life"
> small.upper()
'LIFE'
> big = "LIFE"
> big.lower()
'life'

# 공백지우기 왼쪽 lstrip
> text = " life "
> text.lstrip()
'life '

# 공백지우기 오른쪽 rstrip
> text = " life "
> text.lstrip()
' life'

# 공백지우기 strip
> text = " life "
> text.strip()
'life'

# 문자열 바꾸기 replace
> "Life is so cool".replace("Life", "Weather")
'Weather is so cool'

# 문자열 나누기 split
> "Life is so cool".split()
['Life', 'is', 'so', 'cool']
> b = "l:i:f:e"
>>> b.split(':')
['l', 'i', 'f', 'e']
~~~

## 참고
- [https://wikidocs.net/13]