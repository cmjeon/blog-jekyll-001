---
layout: single
title: 클린코드 스터디 - 015
categories: 
  - bookstudy - cleancode
tags: 
  - cleancode
---

## 15장 Junit 들여다보기

> JUnit 은 Java 용 유닛 테스트 프레임워크 이다.
> [https://ko.wikipedia.org/wiki/JUnit](https://ko.wikipedia.org/wiki/JUnit)

### JUnit 프레임워크

#### 들어가면서..

이번 장은 JUnit 중 ComparisonCompactor 라는 모듈을 보이스카웃 정신을 갖고 점진적으로 개선 하는 과정을 보여준다.
*ComparisonCompactor 모듈 = 두 문자열을 받아 차이를 반환한다.
예를들어, ABCDE 와 ABXDE를 받아 <...B[X]D...> 를 반환한다.

#### 의미 없는 접두어를 제거하자

오늘날 사용하는 개발 환경에서는 이처럼 변수 이름에 범위를 명시할 필요가 없다. 접두어 f는 중복되는 정보다.

- 변경 전

```java
private int fContextLength;
private String fExpected;
private String fActual;
private String fPrefix;
private String fSuffix;
```

- 변경 후

```java
private int contextLength;
private String **expected**;
private String **actual**;
private String **prefix**;
private String **suffix**;
```

#### 의도를 명확히 표현하기 위해 조건문을 캡슐화 해라
(+ 조건문을 메서드로 뽑아내 적절한 이름을 붙인다)

- 변경 전

```java
public String compace(String message){
	if (expected == null || actual == null || arStringEqual())
		return Assert.format(message, expected, actual);
	...
}
```

- 변경 후

```java
public String compace(String message){
	if(shouldNotCompact())
		return Assert.format(message, expected, actual);
	...
}
private boolean shouldNotCompact(){
	return expected == null || actual \\ null || areStringEqal();
}
```

#### 변수의 이름을 명확하게 붙인다

- 변경 전

```java
String expected = compactString(this.expected); 
String actual = compactString(this.actual);
```

- 변경 후

```java
String compactExpected = compactString(**expected**); 
String compactActual = compactString(**actual**);
```

#### 부정문이 아닌 긍정문 으로 만들어 조건문을 반전한다

: 부정문은 긍정문 보다 이해하기가 어렵기 때문

- 변경 전

```java
public String compace(String message){
	if(shouldNotCompact())
		return Assert.format(message, expected, actual);
	...
}
private boolean shouldNotCompact(){
	return expected == null || actual \\ null || areStringEqal();
}
```

- 변경 후

```java
public String compace(String message){
	if(canBeCompacted())
		return Assert.format(message, expected, actual);
	...
}
private boolean canBeCompacted(){
	return expected == null || actual \\ null || areStringEqal();
}
```

#### 함수를 만들 때에는 사용 방식을 일관성 있게 만들어라

- 변경 전

```java
private void compactExpectedAndActual() { 
	findCommonPrefix(); 
	findCommonSuffix(); 
	compactExpected = compactString(expected); 
	compactActual = compactString(actual); 
}
```

- 변경 후

```java
private void compactExpectedAndActual() { 
	**prefixIndex** = findCommonPrefix(); 
	**suffixIndex** = findCommonSuffix(); 
	compactExpected = compactString(expected); 
	compactActual = compactString(actual); 
}
```

#### 숨겨진 시간적인 결합 (hidden temporal coupling)을 제거해라 (중요)

만약 findCommonPrefix와 findCommonSuffix를 잘못된 순서로 호출하면 밤샘 디버깅이라는 고생문이 열린다. 시간 결합을 외부에 노출하자!

- 변경 전

```java
private void compactExpectedAndActual() { 
	prefixIndex = findCommonPrefix(); 
	suffixIndex = findCommonSuffix(); 
	compactExpected = compactString(expected); 
	compactActual = compactString(actual); 
}
```

- 1차 변경 후

```java
private void compactExpectedAndActual() { 
	prefixIndex = findCommonPrefix(); 
	suffixIndex = findCommonSuffix(prefixIndex); 
	compactExpected = compactString(expected); 
	compactActual = compactString(actual); 
}
```

- 2차 변경 후

```java
private void compactExpectedAndActual() { 
	findCommonPrefixAndSuffix();
	compactExpected = compactString(expected); 
	compactActual = compactString(actual); 
}

private void **findComonPrefixAndSuffix**(){
	findCommonPrefix();
	...
}

private void **findCommonPrefix**(){
	prefixIndex = 0;
	...
}
```

### 결론

⇒ 보이스카우트 규칙을 지키면서 점진적 개선을 했다.

## 참고
- 