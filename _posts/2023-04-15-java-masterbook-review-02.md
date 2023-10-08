---
layout: single
title: "자바 마스터북 정리 - 002"
categories:
  - "자바 마스터북"
tags:
  - "books"
---

생각보다 정리할게 좀 많다.

## Stream API
Stream API 는 작성, 중간 작업, 종료 작업의 세단계로 볼 수 있다.

```java
List<Student> students = new ArrayList<>();
students.add(new Student("Haeun", 100));
students.add(new Student("Shin", 60));
students.add(new Student("Ken", 80));

students.stream() // 작성
	.filter(s -> s.getScore() >= 70) // 중간 작업
    .forEach(s -> System.out.println(s.getName()); // 종료 작업
```

### 람다식 작성법

람다식은 메서드의 인수 등에 처리 그 자체를 건네는 것이 가능하다.

```java
List<Student> students = new ArrayList<>();
students.add(new Student("Haeun", 100));
students.add(new Student("Shin", 60));
students.add(new Student("Ken", 80));

// Java 7 이전
Collection.sort(students, new Comparator<Student>() {
	@Override
    public int compare(Student student1, Student student2) {
    	return Integer.compare(student1.getScore(), student2.getScore());
    }
});

// 람다식
Collection.sort(students, (student1, student2) -> 
	Integer.compare(student1.getScore(), student2.getScore())
);
```

Java 8 에서는 구현해야 할 메서드가 하나밖에 없는 인터페이스를 '함수형 인터페이스'로 취급할 수 있다.

람다식은 이 함수형 인터페이스 대신 사용할 수 있다. (ex. java.util.Comparator)

람다식을 전달할 때 인수 타입을 생략할 수 있다.

또한 인수가 하나인 경우는 소괄호 생략도 가능하다.

처리가 한줄밖에 없는 경우는 return 을 생략가능하다.

### 메서드 참조

준비된 메서드 그 자체를 전달할 수도 있다.

대입할 곳의 함수형 인터페이스에 있는 인수의 수와 타입이 일치하고 있다면 거기에는 메서드를 대입할 수 있다.

```java
List<String> list = Arrays.asList("XXX", "YYY", "ZZZ");
list.forEach(System.out::println);

// 동일한 결과
list.forEach(string -> System.out.println(string));
```

위 예시는 forEach 의 인수의 수와 타입이 println 의 인수의 수와 타입과 일치하고 있기 때문에 가능했다.

메서드 참조 문법

|용법|문법|
|---|---|
|인스턴스의 메서드를 참조|{인스턴스명}::{메서드명}|
|자기 자신의 인스턴스의 메서드를 참조|this::{메서드명}|
|static 메서드를 참조|{클래스명}::{메서드명}|

## Stream

### List 에서 Stream

```java
List<String> list = Arrays.asList("Haeun", "Shin", "Shion");
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
```

### Map 에서 Stream

```java
Map<String, String> map = new HashMap<>();
map.put("1", "Haeun");
map.put("2", "Shin");
map.put("3", "Ken");

// Value
Stream<String> stream = map.values().stream();
stream.forEach(System.out::println);

// Entry
Stream<Entry<String, String>> stream = map.entrySet().stream();
stream.forEach(e ->
	System.out.println(e.getKey() + ":" + e.getValue())
);
```

## Stream 중간작업

### Map

map 으로 요소를 치환할 수 있다.

|메서드명|처리내용|인수|반환값|
|---|---|---|---|
|map|요소를 별도의 값으로 치환한다.|Function(인수가 Stream 대상 객체, 반환값이 임의의 타입)|Stream<변환 후의 타입>|
|mapTo(Int\|Double\|Long)|요소를 (int\|double\|long) 값으로 치환한다.|Function(인수가 Stream 대상 객체, 반환값이 (int\|double\|long)|(Int\|Double\|Long)Stream|
|flatMap|요소의 Stream 을 결합한다.|Function(인수가 Stream 대상 객체, 반환값이 Stream)|Stream<변환 후의 타입>|

Function 은 람다식이나 메서드 참조를 대입할 수 있는 곳에서 사용할 함수형 인터페이스이다.

```java
@FunctionalInterface
public interface Function<T, R>
```

라는 선언 내용으로 다음의 메서드를 갖는다.

```java
R apply(T t)
```

T 는 함수의 입력 타입, R 은 함수의 결과 타입

`@FunctionalInterface` 는 인터페이스가 함수형 인터페이스인 것을 명시하기 위한 애노테이션이다.

mapTo(Int|Double|Long) 메서드 들은 sum, average 와 같은 수치 처리 메서드 활용이 가능하다.

flatMap 메서드는 Stream 을 결합하여 하나의 Stream 으로 처리하기 위한 메서드이다.

```java
List<Group> groups = new ArrayList<>();
Group group1 = new Group();
group1.add(new Student());

// ...

groups.add(group1);

// ...

List<Student> allStudents = new ArrayList<>();

// map
groups.stream().forEach(g -> 
	allStudents.addAll(g.getStudents())
);
allStudents.stream()
	.sorted((s1, s2) -> s2.getScroe() - s1.getScore())
    .forEach(s -> System.out.println(s.getName() + " " + s.getScore())
);
    
// flatMap
groups.stream()
	.flatMap(g -> g.getStudents().stream())
    .sorted((s1, s2) -> s2.getScroe() - s1.getScore())
    .forEach(s -> System.out.println(s.getName() + " " + s.getScore())
);
```

### filter, limit, distinct

|메서드명|처리내용|인수|
|---|---|---|
|filter|조건에 일치하는 요소만으로 걸러낸다.|Predicate(인수가 Stream 대상의 객체, 반환값은 boolean)|
|limit|지정한 건수만을 추출한다.|숫자값|
|distinct|유니크한 요소만으로 추출한다.|없음|


### sorted

```java
List<Student> students = new ArrayList<>();
students.add(new Student("Haeun", 100));
students.add(new Student("Shin", 60));
students.add(new Student("Ken", 80));

students.stream()
	.sorted((s1, s2) -> s2.getScore() - s1.getScore())
    .forEach(s -> System.out.println(s.getName() + " " + s.getScore())
);
```

## Stream 종료 작업

### 반복 처리를 실시하는 작업

|메서드명|처리내용|인수|
|---|---|---|
|forEach|이 스트림의 각 요소에 대해 액션을 실행한다.|Consumer(인수가 Stream 대상인 객체, 반환값은 없음|

### 결과를 정리해서 추출하는 작업

|메서드명|처리내용|인수|반환값|
|---|---|---|---|
|collect|요소를 스캐닝하여 결과를 작성한다.|Collector|-|
|toArray|모든 요소를 배열로 변환한다.|없음|OptionalObject[]|
|reduce|값을 집약한다.|BinaryOperator|Optional|

collect 메서드는 Collector 클래스의 각 메서드로 반환한다.

|메서드명|처리내용|인수|
|---|---|---|
|toList|List로 반환한다.|없음|
|toSet|Set로 반환한다.|없음|
|joining|전체 요소를 하나의 문자열로 결합한다.|없음|
|joining|전체 요소를 인수로 지정한 구분 문자를 사용해 결합한다.|String|
|groupingBy|요소를 그룹으로 나눈다.|값을 집계하는 함수|

```java
List<Student> students = new ArrayList<>();
students.add(new Student("Haeun", 100));
students.add(new Student("Shin", 60));
students.add(new Student("Ken", 80));
students.add(new Student("Jung", 100));

Map<Integer, List<Student>> map = students.stream()
	.collect(Collectors.groupingBy(Student::getScore));
/** 점수를 Key 로 하는 학생목록을 반환
"100":["Haeun","Jung"],
"80": ["Ken"],
"60": ["Shin"]
 */
 ```

### 결과를 하나만 추출하는 종료 작업

|메서드명|처리내용|인수|반환값|
|---|---|---|---|
|findFirst|선두의 요소를 반환한다.|없음|Optional|
|findAny|선두의 요소를 반환한다.|없음|Optional|
|min|최소값을 갖는 요소를 반환한다.|Comparator(인수가 Stream 대상 객체)|Optional|
|max|최대값을 갖는 요소를 반환한다.|Comparator(인수가 Stream 대상 객체)|Optional|

Optional 은 객체의 참조가 null 일지 모른다는 명시적으로 나타내는 클래스이다.

### 집계 처리를 실시하는 종료 작업

|메서드명|처리내용|반환값|
|---|---|---|
|count|요소의 개수를 반환한다.|long|
|min|최소 요소를 반환한다.|Optional(Int\|Double\|Long)|
|max|최대 요소를 반환한다.|Optional(Int\|Double\|Long)|
|sum|합계 값을 반환한다.|Optional(Int\|Double\|Long)|
|average|평균 값을 반환한다.|OptionalDouble|

### List 나 Map 에 대한 처리

|메서드명|처리내용|인수|
|---|---|---|
|removeIf|인수로 주어진 함수의 결과값이 true 를 반환하는 요소는 삭제한다.|Predicate(인수가 Stream 대상인 객체, 반환값은 boolean|
|replaceAll|인수로 주어진 함수의 결과로 List 의 모든 요소를 치환한다.| UnaryOperator(인수가 Stream 대상인 객체, 반환값은 동일 타입|

```java
List<String> list = new ArrayList<>();
list.add("HaeunJung");
list.add("IYOU");
list.add("ShionJung");

list.removeIf(s -> s.length() <= 5); // 5 문자 이하 삭제
list.replaceAll(s -> s.toUpperCase()); // 대문자로 변환
```

|메서드명|처리내용|인수|
|---|---|---|
|compute|인수로 주어진 함수의 결과를 Map 에 재설정.|첫번째 인수 Key, 두번째 인수 BiFunction(인수는 키와 값, 반환값은 설정할 값)|
|computeIfPresent|키가 있을 때만, 인수로 주어진 함수의 결과를 Map 에 재설정한다.|첫번째 인수 Key, 두번째 인수 BiFunction(인수는 키와 값, 반환값은 설정할 값)|
|computeIfAbsent|키가 없을 때만, 인수로 주어진 함수의 결과를 Map 에 재설정한다.|첫번째 인수 Key, 두번째 인수 BiFunction(인수는 키와 값, 반환값은 설정할 값)|

## 예외의 세 가지 종류

### 검사 예외(Exception)

비정상 상태를 통지하기 위해 사용한다.

java.lang.Exception 클래스

### 실행 시 예외(RuntimeException)

예상할 수 없는 오류를 통지하기 위해서 사용

java.lang.RuntimeException 클래스

### 오류(Error)

시스템의 동작을 계속할 수 없는 '치명적 오류' 를 나타냄

java.lang.Error

## 예외 처리의 세 가지 구문

### try ~ catch ~ finally

```java
try {
    // ...
} catch (SomeException ex) {
    // ...
} finally {
   // ...
}
```

### try ~ with ~ resources

```java
try (SomeResource resource = getResource()) { // n 개를 넣을 수 있음
    // ...
} catch (SoemException ex) {
    // ...
}
```

Java 7 부터는 리소스를 취급하는 클래스는 java.lang.AutoClosable 인터페이스 or java.io.Closable 인터페이스를 구현한다.

그리고 try 블록 시작 시 해당 클래스(ex. resource)를 선언해두면 try ~ catch 블록의 종료 시의 처리에서 실시할 close 메서드를 자동으로 호출해주기 때문이다.

### 예외 처리

오류 코드를 return 하지 않는다.

예외의 스택 트레이스를 로그에 출력해 두도록 한다.

예외가 발생 했을 때, 처리를 계속할지 판단해야 한다. 처리를 계속해도 된다면 디폴트 값을 줘서 그 이후의 처리를 계속 하도록 한다.

Exception 클래스는 모든 예외의 기저 클래스이므로 IOException 등 구상 클래스의 예외가 발생해도 thows Exception 으로 선언이 가능하다. 그래서 Exception 으로 선언해버리면 Exception 을 받아 처리해야 하는 쪽은 예외의 종류를 이해해서 코드를 작성할 수가 없게된다.

만약 호출한 곳이 Exception 을 throw 하는 경우라면 어쩔 수 없이 Exception 을 받아서 처리해야 한다.

또는 어떤 문제가 발생하더라도 처리를 계속해야하는 경우에도 Exception 을 포착해서 처리해준다.

### 어느 계층에서 예외를 포착해서 처리해야 하는가?

예외가 발생하는 장소는 주로 말단의 처리지점인데, 말단의 처리지점에서는 예외를 발생시키기만 하는 것이 좋다. 왜냐하면 예외 처리시의 처리의 중지 및 회복의 판단을 해야하는데, 말단에서는 전체적인 흐름이 보이지 않기 때문이다.

반면에 처리의 흐름을 판단하는 장소에서는 예외를 포착해서 처리의 중지 혹은 회복을 판단해야 한다.

### 독자적인 예외 작성하기

1. 업무에 특화한 처리인 경우
2. 프레임워크나 시스템에서 공통적인 예외 처리를 하는 경우

위의 두 경우에는 독자적인 예외 처리가 필요하다.

### 독자적인 예외가 만들어지는 경우는 크게 2가지

1. 애플리케이션 예외: 비즈니스 로직에서 오류 발생을 포착
2. 시스템 예외: 시스템이 정상적인 동작을 할 수 없는 상태일 때 오류 발생을 포착

독자적인 예외 클래스에서는 할 수 있는 한 인수를 강제하는 형태로 해야 한다.

메시지 대신에 오류 ID 와 매개변수를 지정하는 것이 좋다.

```java
public class ApplicationException extends Exception {
	private String id;
    private Object[] params;
    
    public ApplicationException(String id, Object... params) {
    	super();
        this.id = id;
        this.params = Arrays.copyOf(params, params.length);
    }
    
    public ApplicationException(String id, Throwable cause, Object... params) {
    	super(cause);
        this.id = id;
        this.params = Arrays.copyOf(params, params.length);
    }

	public String getId() {
    	return id;
    }
    
    public Object[] getParams() {
    	return Arrays.copyOf(params, params.length);
    }
}
```

실제 메시지 등을 '오류ID에 해당하는 정보'로 프로그램 밖으로 내보낼 수 있게 된다.

프로그램에 메시지를 넣어두면 유지보수가 힘들어진다. 다국어 대응이 필요할 수도 있다.

또한 로그 메시지에 넣을 매개변수는 가변 길이 인자로 사용할 수 있도록 해둔다.

## 예외의 트렌드

### 검사 예외보다도 실행 시 예외를 사용한다.

예외 처리 자체를 간단하게 하는 목적과 예외를 일일이 개별 로직으로 처리하지 않아도 되기 때문에 로직의 코드도 간단하게 된다는 장점이 있다.

### 람다식 안에서 발생한 예외의 취급

람다식에서는 람다 안에서 기술한 처리에서 예외가 발생할 가능성도 있다.

람다 안에서 발생한 실행 시 예외를 throw 해서 람다 바깥쪽으로 전달할 수 있다.

하지만 병렬 실행하는 경우가 있기 때문에 바깥쪽에서 모든 예외를 수취하는 것보다는 람다 안에서 예외를 처리하는 것이 좋다.

### Optional 클래스의 도입에 의한 장점

Optional 클래스가 가진 주요 메서드

|메서드|설명|
|---|---|
|Optional\<T\> optional = Optional.of(T value)|값을 갖는 Optional 객체를 반환한다.|
|Optional\<T\> optional = Optional.empty()|값을 갖지 않는 Optional 객체를 반환한다.|
|T value = optional.get()|값을 반환한다. ifPresent() 메서드로 확인 후 사용한다.|
|T value = optional.orElse(T value)|값이 없는 경우는 인수로 지정된 값을 반환한다.|
|optional.isPresent()|값을 갖는 경우는 true, 아니면 false|
|optional.isPresent(value -> {...})|값을 갖는 경우에 람다식의 처리를 수행한다.|

###  String 클래스의 특징

String 클래스는 불변 객체다.

문자열을 결합할 때는 성능을 고려해서 StringBuilder 클래스를 사용한다.

```java
String aaa = "aaa";
String bbb = "bbb";

StringBuilder builder = new StringBuilder();
builder.append(aaa);
builder.append(bbb);

String str = builder.toString();
```

여러 스레드로부터 사용되는 문자열 조작을 위해서는 StringBuffer 클래스를 사용한다.

### 문자열 관련 메서드

join() 메서드

```java
List<String> stringList = new ArrayList<>();
stringList.add("This");
stringList.add("is");
stringList.add("a");
stringList.add("pen");

String message = String.join(" ", stringList);

String message2 = String.join(".", "www", "jpub", "co", "kr");
```

replace() 메서드

```java
String sentence = "This is a pen";
String replaced = sentence.replace("is", "at");
```

indexOf() 메서드

```java
String sentence = "This is a pen";
int index = sentence.indexOf("is"); // 2

// 문자열이 없으면 -1

// 4 번째 이후부터 검색
int index2 = sentence.indexOf("is", 3); // 5
```

### 정규표현 처리의 흐름

1. 정규 표현 패턴을 생성
2. 체크할 문자열
3. 정규 표현 처리를 위한 객체 취득
4. 객체로 수행

```java
// 1
Pattern pattern = Pattern.compile("This is a .*\\.");

// 2
String sentence = "This is a pen.";

// 3
Matcher matcher = pattern.matcher(sentence);

// 4 정규 패턴에 일치하는지 체크
if (matcher.matches()) System.out.println("적합");
else System.out.println("부적합");

// 4 문자열 치환
matcher.replaceAll("That is a pen!");
```

String 클래스의 메서드로 정규 표현 사용도 가능하다.

```java
String sentence = "This             is a pen.";
sentence.replaceAll("\\s+", " ");
```

String 클래스 내부에서 Pattern 클래스와 Matcher 클래스의 객체를 생성하여 처리하기 때문에 성능저하가 발생한다.

따라서 한번만 문자열을 처리할 경우 String 클래스를 사용해도 되지만 대량의 문자열을 반복해서 처리할 경우는 직접 Pattern 클래스와 Match 클래스의 객체를 생성하는 편이 좋다.

### 문자열을 포맷에 맞게 출력하는 방법

|문자열|설명|
|---|---|
|%d|10진수정수로 표현|
|%X|16진수정수로 표현|
|%s|문자열로 표현|
|%S|대문자문자열로 표현|

```java
int number = 13;
String string = "apples";
System.out.printf("I Have %X %S.", number, string);
// I Have D APPLES.`
```

### MessageFormat

```java
int number = 13;
String string = "apples";
MessageFormat.format("I Have {0} {1}.", number, string);
// I Have 13 apples.`
```

## 자바의 문자코드

자바는 내부적으로 UTF-16 으로 인코딩된 Unicode 를 사용하여 문자열을 보관하고 있다.

```java
char c = '가';
System.out.printf("%02x", (int)c);
// ac00
```

문자열에서 문자코드로 변환하기

```java
String str = "가나다";

byte[] utf8;
utf8 = str.getBytes("UTF-8");
for (byte b : utf8) {
	System.out.printf("%02x", b);
}

byte[] utf16;
utf16 = str.getBytes("UTF-16");
for (byte b : utf16) {
	System.out.printf("%02x", b);
}

// ... getBytes("UTF-32");
// ... getBytes("MS949");
```

### 서로게이트 페어(Surrogate Pair)

서로게이트 페어는 '여러 문자(16비트 부호)로 한 문자를 표현하는 형식'이다.

서로게이트 페이를 금지하는 방법

```java
char[] chars = str.toCharArray();
for (char c : chars) {
	if (Character.isLowSurrogate(c) || Character.isHighSurrogate(c)) {
    	// 서로게이트 페어가 포함되어 있음
    	return true;
    }
}

// 서로게이트 페어가 포함되어 있지 않음
return false;
```

서로게이트 페어에 대응하는 부분에서는 String 객체의 codePointCount() 메서드를 사용한다.

String 에서 동일 객체를 명시적으로 참조하도록 하려면 intern() 메서드를 사용한다.

```java
String aaa = "aaa";
String aa = "aa";
String a = "a";

System.out.println(aaa.equals(aa + a)); // true
System.out.println(aaa == (aa + a)); // false
System.out.println(aaa == (aa + a).intern()); // true
```
