---
layout: single
title: "자바 마스터북 정리 - 001"
categories:
  - "자바 마스터북"
tags:
  - "기술"
---

자바 마스터 북을 읽고 인상깊었던 내용을 정리하였다.

## 비트연산자

|기호|의미|
|---|---|
|&|비트 연산 AND|
|\||비트 연산 OR|
|^|비트 연산 XOR|
|<<|우변의 수만큼 비트를 왼쪽으로 옮김. 빈 비트를 0으로 채움|
|>>|우변의 수만큼 비트를 오른쪽으로 옮김. 양수와 부호 비트는 유지하고 빈 비트는 0으로 채움|
|>>>|우변의 수만큼 비트를 오른쪽으로 옮김. 빈 비트를 0으로 채움|
|~|를 반전|

예시

```java
// & 예시
int number1 = 0x12345678;
int lower = number1 & 0x0000ffff;
System.out.printf("lower = %x\n", lower); // lower = 5678

// >> 예시
int number2 = 0x12345678;
int higher = number2 >> 16;
System.out.printf("higher = %x\n", higher); // higher = 1234
```

## 클래스와 메소드의 수식자

### transient

객체의 직렬화 시에 해당 필드를 직렬화의 대상에서 제외하겠다는 지정이다.

직렬화(serialize)란 인스턴스화되어 있는 자바 객체를 바이트 열로 변화하는 것을 말한다.

### volatile

멀티 스레드로부터 액세스되는 필드에 대해 CPU cache 에 캐시되지 않도록 하는 지정이다.

필드가 CPU cache 에 저장되면 스레드마다 값이 다를 수 있는데, volatile 을 통해 Main Memory 에서만 참조가능하도록 할 수 있다.

### synchronized

대상의 처리를 동기화하기 위한 지정이다.

이것을 지정한 메소드나 블록의 내부는 동시에 하나의 스레드에서만 액세스할 수 있다.

### strictfp

부동소수점 수를 IEEE 754 규격으로 엄밀하게 관리한다는 지정이다.

인터페이스 또는 메소드에 지정할 수 있다.

## 래퍼 클래스

기본형은 객체가 아니라 단순히 값이며 그 자신은 메서드를 갖지 않는다. 하지만 프로그램 내에서는 기본형의 값에 대해 조작이 필요하게 되는 상황이 있다. 그래서 자바는 기본형의 값을 조작하는 기능이 있는 래퍼 클래스를 제공한다.

래퍼클래스에 있는 상수

|상수|설명|
|---|---|
|SIZE|비트 수|
|BYTES|바이트 수|
|MAX_VALUE|최댓값|
|MIN_VALUE|최소값|

래퍼클래스에 있는 메소드

|메소드|설명|예시|
|---|---|---|
|valueOf(기본형)|기본형 -> 래퍼클래스|Integer.valueOf(10)|
|valueOf(String s)|문자열 -> 래퍼클래스|Integer.valueOf("10")|
|valueOf(String s, int radix)|radix 진수로 표현된 문자열 -> 래퍼클래스|Integer.valueOf("10",2)|
|parseXxx(String s)|문자열 -> 기본형|Integer.parseInt("10")|
|parseXxx(String s, int radix)|radix 진수로 표현된 문자열 -> 기본형|Integer.parseInt("10", 2)
|toString(기본형)|기본형 -> 문자열|Integer.toString(10)|
|toString(기본형, int radix)|radix 진수로 표현된 기본형 -> 문자열|Integer.toString(10, 2)|

생성자를 이용하면 반드시 새로운 객체가 생성되지만 valueOf 를 이용하면 -128 ~ 127 범위의 값이라면 사전에 생성된 객체를 이용하기 때문에 메모리 소비를 하지 않는다.

래퍼클래스의 초기값은 null, 따라서 0인 값과 데이터가 없는 상태를 구별하고 싶다면 래퍼형을 준비해야 한다.

반면에 대량의 수치계산에는 기본형을 사용하는 것이 좋다.

## 오토박싱과 언박싱

기본형에서 래퍼 클래스로 자동 변환되는 것을 오토박싱, 래퍼 클래스에서 기본형으로 변환되는 것을 언박싱이라고 한다.

```java
int num = 10;

// 오토박싱(기본형 -> 래퍼)
Integer numInt = 10;

// 언박싱(래퍼 -> 기본형)
Integer sum =- num + numInt;
```

-128 ~ 127 범위의 값은 같은 객체를 사용하기 때문에 오토박싱된 객체를 비교해도 참이다.

### 방침

- 원칙적으로 오토박싱, 언박싱보다는 명시적인 변환
- 파일, HTTP 요청 등을 유지하는 값은 래퍼 클래스
- 수치 연산에는 기본형

## 익명 클래스

```java
public interface TaskHandler {
	boolean handle(Task task);
}

// ...

TaskHandler taskHandler = new TaskHandler() {
	public boolean handle(Task task) {
    	// 로직
    }
};
Task myTask = new Task();
taskHandler.handle(myTask);
```

## 객체의 등가성(동등성)

equals() 메소드를 오버라이드해서 객체의 내용이 동등한지 비교할 수 있다.

### hashCode 메소드

hashCode() 메소드로 해시값을 확인할 수 있다.

- 동일 객체의 해시값은 반드시 동일하다.
- 해시값이 다른 경우에는 다른 객체이다.
- 다른 객체인데 해시값이 같은 경우도 있다.

그래서 빠르게 비교하기 위해 먼저 해시값을 통해 객체를 비교하고, 해시값이 동일하면 equals() 메소드로 엄격하게 살펴보는 것이 좋다.

따라서 두 객체가 동일하다고 하는 것은 hashCode() 메소드의 값이 동일하고, equals() 메소드로 같다고 판정되어졌다는 뜻이다.

equals() 메소드만 Override 하고, hashCode() 메소드는 구현하지 않으면 버그가 발생할 수 있다.

hashCode() 메소드는 Object 클래스의 hashCode() 를 사용한다.

Object 의 hashCode 메소드는 다른 객체이면 다른 hashCode 를 가진다.

hashCode 로 동등성을 판정하는 HashSet 의 경우, equals 로는 값이 같다고 판정되어도 hashCode 가 달라서 다른 객체로 저장되는 오류가 발생할 수 있다.

따라서 가지고 있는 값이 같으면 같은 객체라고 판정이 필요할 경우에는 equals 와 hashCode 를 모두 Override 해야한다.

```java
// hashCode 예시
@Override
public int hashCode() {
	final int prime = 31;
	int result = 1;
	result = prime * result + employeeNo;
	result = prime * result + ((employeeName == null) ? 0 : employeeName.hashCode());
    return result;
}
```

Objects.hash 메소드를 사용해서 hashCode 를 생성할 수도 있다.

```java
// hashCode 예시
@Override
public int hashCode() {
    return Objects.hash(employeeNo, employeeName.hashCode());
}
```

toString() 메소드도 Override 해서 객체의 값을 표시하도록 하면 좋다.

Apache Commons Lang 라이브러리를 이용할 수도 있다.

- [https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/builder/ToStringBuilder.html?ref=yomojomo](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/builder/ToStringBuilder.html)

```java
@Override
public String toString() {
    return ToStringBuilder.reflectionToString(this);
}
```

## 열거형(Enum)

public static final 상수는 '타입 안전이 아니다' 라는 문제가 있다.

그리고 자바에서는 컴파일 시에 상수를 이용하고 있는 클래스의 상수에 상수를 정의하고 있는 클래스의 상수값 그 자체가 전개된다.

그래서 상수를 정의하고 있는 클래스 쪽에서 상수의 값을 변경해도 상수를 이용하고 있는 쪽의 값은 변경되지 않는다.

## 제네릭스(Generics)

임의의 타입을 사용하는 클래스 GenericsStack 예시

```java
public class GenericsStack<E> {

    private List<E> taskList;

    public GenericsStack() {
    	taskList = new ArrayList<>();
    }

    public boolena push(E task) {
    	return taskList.add(task);
    }

    public E pop() {
    	if (taskList.isEmpty()) {
        	return null;
        }
        return taskList.remove(taskList.size() - 1);
    }
}
```

임의의 타입을 사용하는 메소드 as() 예시

```java
public class GenericsStack<E> {

    private List<E> taskList;

    public GenericsStack() {
    	taskList = new ArrayList<>();
    }

    public boolena push(E task) {
    	return taskList.add(task);
    }

    public E pop() {
    	if (taskList.isEmpty()) {
        	return null;
        }
        return taskList.remove(taskList.size() - 1);
    }
}
```

## 배열

배열 복사 방법

```java
Arrays.copyOf(array, array.length + 3);
```

배열의 요소를 문자열로 보기

```java
System.out.println(Arrays.toString(array));
```

### 정렬하기
Comparable 인터페이스를 구현해서 정렬하기

```java
Integer[] array = {3,1,4,2,5};
Arrays.sort(array);
// array [1,2,3,4,5]

Comparator<Integer> c = new Comparator<Integer> () {
	@Override
    public int compare(Integer o1, Integer o2) {
    	return o2.compareTo(o1);
    }
};

Arrays.sort(array, c);
// array [5,4,3,2,1]
```

정렬할 클래스를 Comparable 인터페이스의 구현 클래스로 만들어 compareTo() 메서드를 구현할 수도 있다.

차이점은 Comparable 인터페이스로 구현하면 한 종류의 정렬만 사용가능해지고, 정렬 시 Comparator 인터페이스를 구현한 클래스를 전달하면 원하는 방식의 정렬이 가능해진다.

따라서 대부분의 경우 Comparator 를 사용하는 편이 낫다.

### 검색하기

binarySearch 를 이용하면 선형 검색보다 빠르게 탐색할 수 있다.

```java
int [] array = { 1, 1, 2, 3, 5, 8, 9 }
int found = Arrays.binarySearch(array, 5);
// 4, 5 라는 숫자의 인덱스
int notFound = Arrays.binarySearch(array, 7);
// -6, 7 이라는 숫자가 있었다면 반환되었을 인덱스의 음수값에 -1 을 더함
```

binarySearch 는 사전 정렬이 필요하다.

정렬을 하지 않았다면 선형 검색을 실시한다.

### 가변 길이 인수로 메서드 정의하기

```java
void log(String message, String[] args) {
	System.out.println(message);
  	System.out.println("매개변수:");
    for(String arg : args) {
    	System.out.println(arg);
    }
}

// ...

log("사용자를 등록", new String[]{"userName", "Ken"});
log("처리 종료", new String[0]);
```

호출하는 쪽에서는 arg 인수에 배열을 보내야 해서 매번 배열을 new 해야 한다.

가변 길이 인수 메서드를 정의하면 배열을 new 할 필요가 없다.

```java
void log(String message, String... args) {
	System.out.println(message);
  	System.out.println("매개변수:");
    for(String arg : args) {
    	System.out.println(arg);
    }
}

// ...

log("사용자를 등록", "userName", "Ken");
log("처리 종료");
```

## 컬렉션
컬렉션은 배열과 달리 처음부터 크기의 제한을 결정할 필요가 없다.

|명칭|개요|
|---|---|
|java.util.List|배열처럼 복수의 요소를 취급. 인덱스를 지정하여 값의 취득, 등록 가능|
|java.util.Set|중복이 없는 요소를 취급. 순서성이 없음|
|java.util.Map|키와 값을 이용하여 요소를 취급. 순서성이 없음. aka 딕셔너리|

### Iterator

List 인터페이스의 iterator 메서드를 실행해서 Iterator 인터페이스를 취득할 수 있다.

```java
List<Student> students = new ArrayList<>();
students.add(new Student("Ken", 100));
students.add(new Student("Shin", 60));
students.add(new Student("Kim", 80));

Iterator<Student> iterator = students.iterator();
while (iterator.hasNext()) {
	Student student = iterator.next();
    if (student.getScore() < 70) {
    	iterator.remove(); // iterator 로 요소 삭제 가능
    }
}
```

### List 의 구현 클래스들

|클래스|사용|
|---|---|
|ArrayList|for 문을 사용한 전체적인 반복 처리가 많은 경우|
|LinkedList|배열의 중간에서 요소의 추가나 삭제할 일이 많은 경우|
|CopyOnWriteArrayList|복수의 스레드에서 동시에 액세스하는 경우|

### Map 의 구현 클래스들

|클래스|사용|
|---|---|
|HashMap|기본적인 경우|
|LinkedHashMap|요소의 순서를 유지할 필요가 있는 경우|
|ConcurrentHashMap|복수 스레드로부터 동시에 액세스하는 경우|
|TreeMap|키의 대소를 의식한 부분 집합을 취급할 경우|

## Set 인터페이스

Set 내의 값 검색

```java
List<Integer> integerList = new ArrayList<>();

// List -> Set
Set<Integer> intergerSet = new HashSet<>(integerList);

// 배열 -> Set
Integer[] intergerArray = new Integer[] {1,2,3,4,5};
List<Integer> integerList = Arrays.asList(integerArray);
Set<Integer> integerSet = new HashSet<>(integerList);
```

### Set 의 구현 클래스들

HashSet 클래스는 값의 해시값을 계산해서 내부의 해시 테이블에 저장하여 요소를 관리한다.

요소를 추가할 때 해시 테이블의 크기를 확장하고, 요소에 대한 액세스가 동시에 진행되면 무한 루프에 빠질 수 있다.

HashSet 은 요소의 추가 순서가 유지되지 않는다.

LinkedHashSet 클래스는 요소의 순서를 유지한다.

LinkedHashSet 의 요소 추가/삭제는 요소 사이의 링크교체가 필요하여 HashSet 에 비해 성능이 약간 저하된다.

TreeSet 클래스는 이진 탐색 알고리즘에 의해 요소를 정렬하는 클래스이다.

정렬은 요소를 추가할 때 이류어진다.

Set 의 내부에 Map 존재하고, Set 에 추가된 요소들은 Map 의 Key 로 유지된다. Map 의 Value 에는 boolean 타입의 true 가 보관된다.

```java
// ConcurrentHashMap 에서 Set 만들기
Set<Integer> concurrentHashSet = Collections.newSetFromMap(new ConcurrentHashMap<Integer, Boolean>();
```

## 기타

### Queue

Queue 는 선입선출로 값을 넣고 빼는 컬렉션이다.

```java
Queue<Integer> queue = new ArrayBlockingQueue<>(10);
queue.offer(1);
queue.offer(2);
queue.offer(3);
System.out.println(queue.poll()); // 3
System.out.println(queue.poll()); // 2
queue.offer(4);
queue.offer(5);
System.out.println(queue.peek()); // 5
System.out.println(queue.peek()); // 5
```

### Dequeue

Double Ended Queue 는 양방향으로 값을 넣고 빼는 것이 가능한 컬렉션이다.

