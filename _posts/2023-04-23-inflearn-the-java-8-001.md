---
layout: single
title: "더 자바, Java 8 - 001"
categories:
  - "더 자바, Java 8"
tags:
  - "강좌"
---

[https://www.inflearn.com/course/the-java-java8/dashboard](https://www.inflearn.com/course/the-java-java8/dashboard)

## 참고
- https://www.oracle.com/java/technologies/java-se-support-roadmap.html
- https://www.javacodegeeks.com/2019/07/long-term-support-mean-openjdk.html
- https://en.wikipedia.org/wiki/Java_version_history

## 함수형 인터페이스와 람다 표현식 소개

### 함수형 인터페이스

인터페이스에 추상 메서드가 하나만 있으면 함수형 인터페이스

@FunctinalInterface 애노테이션 선언

인터페이스의 static 메서드의 public 은 생략가능

default 메서드

### 람다 표현식(Lambda Expression)

기존의 함수형 인터페이스의 인스턴스 만드는 방법은 내부 익명 클래스였다.

이제는 람다 표현식으로 가능하다.

메소드 매개변수, 리턴 타입, 변수로 만들어 사용할 수 있다.

### 함수형 프로그래밍

일급 객체로 사용할 수 있다.

순수 함수이다.

- 입력값이 동일하면 결과가 같다.
- 상태가 없다.
- 사이드 이펙트가 없다

고차 함수(= 고계 함수)이다.

- 함수가 함수를 매개변수를 받고, 리턴할 수 있다.
- 불변

### 자바에서 제공하는 함수형 인터페이스

[https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)

### Function

[https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html)

- R apply(T t)

추상 메서드: 타입을 받아서 타입을 리턴

함수 조합을 위한 compose(), andThen() 메서드

```java
Function<Integer, Integer> plus10 = (i) -> i + 10;
Function<Integer, Integer> multiply2 = (i) -> i * 2;

Function<Integer, Integer> multiplyAndPlus = plus10.compose(multiply2);
System.out.println(multiplyAndPlus.apply(2));
// 14

Function<Integer, Integer> plusAndMultiply = plus10.andThen(multiply2);
System.out.println(plusAndMultiply.apply(2));
// 24
```

### BiFunction

두 개의 값(T, U)를 받아서 R 타입을 리턴하는 함수 인터페이스

- R apply(T t, U u)

### Consumer

T 타입을 받아서 아무값도 리턴하지 않는 함수 인터페이스

```java
Consumaer<Integer> printT = (i) -> System.out.println(i);
printT.accept(10);
```

### Supplier

T 타입의 값을 제공만 하는 함수 인터페이스

```java
Supplier<Integer> get10 = () -> 10;
get10();
```

### Predicate

T 타입을 받아서 boolean을 리턴하는 함수 인터페이스

- boolean test(T t)

### UnaryOperator<T>

Function<T, R>의 특수한 형태로, 입력값 하나를 받아서 동일한 타입을 리턴하는 함수 인터페이스

### BinaryOperator<T>
BiFunction<T, U, R>의 특수한 형태로, 동일한 타입의 입렵값 두 개를 받아 동일한 타입을 리턴하는 함수 인터페이스

## 람다 표현식

(인자) -> { 바디 }

인자

```
// 인자없음
() ->

// 인자 1개
(param) ->
param ->

// 인자 n개
(param1, param2) ->

// 타입은 생략가능
(Integer param1, Integer param2) ->
(param1, param2) ->
```

바디

```
// 한 줄인 경우에 생략 가능, return도 생략 가능.
() -> "Hello";

//여러 줄인 경우에 { }를 사용해서 묶는다.
() -> {
    String message = "lambda";
    return "Hello" + message;
}
```

### 변수 캡처(variable capture)

로컬 변수 캡처

- final이거나 effective final 인 경우에만 참조할 수 있다.
- 그렇지 않을 경우 concurrency 문제가 생길 수 있어서 컴파일가 방지한다.

```java
final int baseNumber = 10;

IntConsumer printInt = (i) -> {
    System.out.println(i + baseNumber);
}
```

일때 baseNumber 가 캡처된다.

만약 로컬 클래스나 익명 클래스에 똑같은 변수명인 baseNumber 가 있다고 가정한다.

로컬 클래스나 익명 클래스에서 baseNumber 를 호출하면 해당 클래스 안에 있는 변수의 값이 나온다.

즉 바깥의 baseNumber 를 가리게 된다. (shadowing)

[https://catsbi.oopy.io/9b757e48-a756-4469-973e-a06d0f34e7a4](https://catsbi.oopy.io/9b757e48-a756-4469-973e-a06d0f34e7a4)

반면에 람다에서는 같은 이름의 변수명 baseNumber 를 쓸 수 없다.

람다와 람다를 감싸고 있는 메소드와 같은 scope 라고 간주하기 때문이다.

effective final 은 사실상 final 인 변수로 익명 클래스, 람다에서 참조가능하다.

### 참고

- [https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html#shadowing](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html#shadowing)
- [https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)

## 메서드 레퍼런스

람다의 일이 기존 메소드 또는 생성자를 호출하는 것이라면, 메소드 레퍼런스를 사용해서 매우 간결하게 표현할 수 있다.

|참조|사용법|
|---|---|
|스태틱 메소드 참조|타입::스태틱 메소드|
|특정 객체의 인스턴스 메소드 참조|객체 레퍼런스::인스턴스 메소드|
|임의 객체의 인스턴스 메소드 참조|타입::인스턴스 메소드|
|생성자 참조|타입::new|

```java
String[] names = {"kkk", "www", "ttt"};
Arrays.sort(names, String::compareToIgnoreCase);
// String 타입의 compareToIgnoreCase 참조
```

### 참고

- [https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

## 인터페이스 default 메소드와 static 메소드

### default 메소드

인터페이스에 메소드 선언만이 아니라 구현체를 제공할 수도 있다.

인터페이스를 구현한 클래스를 깨트리지 않고 새 기능을 추가할 수 있다.

구현체가 모르는 인터페이스의 기능으로 컴파일 에러가 발생하지는 않지만 구현체에 따라 런타임 에러가 발생할 수 있다.

인터페이스에 기본구현체를 만들었으면 문서화를 할 것(@implSpec 자바독 태그)

default 메소드는 인터페이스의 구현체에서 제정의할 수 있다.

default 메소드를 가진 인터페이스를 상속해서 default 메소드를 추상메소드로 만들어서 default 메소드를 못 쓰게 만들 수 있다.

만약 똑같은 이름의 default 메서드를 구현한 구현체라면 컴파일 에러가 발생하고 해당 메서드를 구현체에서 직접 구현해야 한다.

### static 메소드

유틸리티성 메소드를 제공할 때 인터페이스에 스태틱 메소드를 제공할 수 있다.

### 참고

- [https://docs.oracle.com/javase/tutorial/java/IandI/nogrow.html](https://docs.oracle.com/javase/tutorial/java/IandI/nogrow.html)
- [https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)

## 자바 8 API의 default 메소드와 static 메소드

### Iterable

```java
List<String> names = new ArrayList<>();
names.add("aaa");
names.add("bbb");
names.add("ccc");

// forEach()
name.forEach(System.out:println);

// spliterator()
Spliterator<String> spliterator = name.spliterator();
while (spliterator.tryAdvance(System.out::println));
```

### Collection

```java
List<String> names = new ArrayList<>();
names.add("aaa");
names.add("bbb");
names.add("ccc");

// stream()
name.stream().map(String::toUpperCase)
    .filter(s -> s.startsWith("k")
    .collect(Collectors.toList());

// removeIf()
name.removeIf(s -> s.startsWith("k"));
```

### Comparator

```java
List<String> names = new ArrayList<>();
names.add("aaa");
names.add("bbb");
names.add("ccc");

// comparator()
name.sort(String::compareToIgnoreCase);

Comparator<String> compareToIgnoreCase = String::compareToIgnoreCase;
name.sort(compareToIgnoreCase.reversed());

name.sort(compareToIgnoreCase.reversed().thenComparing);

/**
 * static reverseOrder() / naturalOrder()
 * static nullsFirst() / nullsLast()
 * static comparing()
 */
 ```

인터페이스의 default 메소드가 생기면서 상속을 강제하지 않는 방법이 생겼다.

이는 비침투적 방법으로 스프링에서 주로 사용된다.

WebMvcConfigurer 인터페이스 사례

### 참고

- [https://docs.oracle.com/javase/8/docs/api/java/util/Spliterator.html](https://docs.oracle.com/javase/8/docs/api/java/util/Spliterator.html)
- [https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html)
- [https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html)
- [https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html)

## Stream 소개

### Stream

연속된 데이터를 처리하는 operation 모음

데이터를 담고 있는 저장소(컬렉션)가 아니다

데이터 원천은 변경하지 않는다.

parallelStream() 메소드가 있다.

멀티스레드로 나눠서 처리하는 방안인데, 특별히 유용한 상황이 있다.

보통은 stream() 메소드를 쓰는 것이 좋다.

### Stream pipeline

0 ~ n 개의 중개 오퍼레이션 + 한 개의 종료 오퍼레이션으로 구성한다.

### 중개 오퍼레이션 intermediate operation

Stream 을 리턴한다.

중개 오퍼레이션은 Lazy 하다.
예컨데 중개 오퍼레이션 내에서 출력을 수행해도 종료 오퍼레이션이 없으면 출력이 되지 않는다.

종료 오퍼레이션 terminal operation

Stream 을 리턴하지 않는다.

### 참고

- [https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)
- [https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)

## Stream API

### filter(): 걸러내기

filter(predicate)

```java
// 종료된 수업
filter(oc -> oc.isClosed());
// 메서드 레퍼런스
filter(OnlineClass::isClosed);

// 종료되지 않은 수업
filter(oc -> !oc.isClosed());
// 메서드 레퍼런스
filter(Predicate.not(OnlineClass::isClosed));
```

### map(): 변경하기

map(Function)

FlatMap(Function)

컬렉션 안의 값을 풀어주는 방식

```java
list.stream().flatMap(Collection::stream)
    .forEach(oc -> System.out.println(oc.getId()));
```

### generate(Supplier): 생성하기 or Iterate(T seed, UnaryOperator)

limit(long) or skip(long)

```java
Stream.iterate(10, i -> i + 1)
    .skip(10) // 처음 10개 무시
    .limit(10) // 10개만 취득
    .forEach(System.out::println);
```

### 스트림에 있는 데이터가 특정 조건을 만족하는지 확인

```java
boolean test = list.stream().anyMatch(oc -> 
    oc.getTitle().contains("Test"));
```

### 개수 세기

```java
list.stream
    .filter(oc -> oc.getTitle().contains("spring"))
    .map(OnlineClass::getTitle)
    .collect(Collectors.toList());
```

### 스트림을 데이터 하나로 뭉치기

reduce(identity, BiFunction), collect(), sum(), max()
