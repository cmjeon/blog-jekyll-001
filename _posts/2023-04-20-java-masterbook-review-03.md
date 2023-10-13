---
layout: single
title: "자바 마스터북 정리 - 003"
categories:
  - "자바 마스터북"
tags:
  - "기술"
---

## 파일 조작

기존의 File 클래스에는 제약이 있다.

- 파일의 메타 데이터와 심볼릭 링크를 취급할 수 없음
- 디렉터리 밑의 파일의 생성/삭제/갱신을 감시할 수 없음

이런 문제를 해소한 것이 Path 클래스와 Path 클래스의 유틸리티 클래스이다.

파일을 읽기 위해서 Files.newInputStream() 메서드를 사용한다.

```java
Path path = Paths.get("C:/work/sample.dat");

try (InputStream is = Files.newInputStream(path)) {
	// ...
} catch (IOException ex) {
	// ...
}
```

try ~ with ~ resources 구문으로 가능하다.

텍스트 파일을 읽을 때는 Files.newBufferedReader() 메서드로 BufferReader 객체 생성하여 사용한다.

파일을 쓰기 위해서 Files.newOutputStream() 메서드나 write() 메서드를 이용하면 된다.

```java
Path path = Paths.get("C:/work/sample.dat");

try (InputStream is = Files.newInputStream(path)) {
	// ...
} catch (IOException ex) {
	// ...
}
```

텍스트 파일을 쓰기 위해서는 Files.newBufferWriter() 메소드로 BufferWriter 객체를 생성하여 사용한다.

### 속성 파일

속성파일 읽을 때는 java.util.Properties 클래스를 사용한다.

```java
Path path = Paths.get("mail.properties");

try (BufferedReader reader = Files.newBufferReader(path, StandardCharsets.UTF_8)) {
	Properties properties = new Properties();
    properties.load(reader);
    
    String address = properties.getPropertiy("system.mail.address");
} catch (IOException ex) {
	// ...
}
```

### 국제화 대응

java.util.ResourceBundle 클래스를 사용하여 속성 파일을 로드하면 국제화 대응이 가능하다.

파일이름은 `기본이름 +_+ 로케일 + .properties`

파일양식에는 CSV, XML, JSON 등이 있고 각 파일을 쉽게 다루기 위한 라이브러리가 있다.

## Date and Time API

```java
// 날짜
LocalDate date = LocalDate.now();

// 시간
LocalTime time = LocalTime.now();

// 일시
LocalDateTime dateTime = LocalDateTime.now();
```

### 날짜계산

```java
LocalDateTime dateTime = LocalDateTime.of(2017, 2, 3, 21, 30, 15);

// 3일 후
dateTime.plusDays(3));

// 100일 전
dateTime.minusDays(100));

// 30초 전
dateTime.minusSeconds(30));

// 원래 dateTime 객체는 바뀌지 않는다.
```

### SimpleDateFormat 클래스의 패턴 문자

|문자|설명|
|---|---|
|G|기원|
|y|연도|
|M|월|
|w|연도에 대한 주|
|W|월에 대한 주|
|D|연도에 대한 일수|
|d|월에 대한 일수|
|F|월에 대한 요일|
|E|요일|
|a|오전/오후|
|H|1일에 대한 시간(0~23)|
|k|1일에 대한 시간(1~24)|
|K|오전/오후에 대한 시간(0~11)|
|h|오전/오후에 대한 시간(1~12)|
|m|분|
|s|초|
|SSS|밀리초|
|z|타임 존|
|Z|타임 존(4행숫자)|


### DateTimeFormatter

DateTimeFormatter 를 이용해 문자열을 날짜/시간 클래스로 변환한다.

```java
TemporalAccessor parsed = DateTimeFormatter
        .ofPattern("yyyy/MM/dd HH:mm:ss")
        .parse("2017/02/25 19:09:59");
LocalDateTime date = LocalDateTime.from(parsed);
```

SimpleDateFormat 은 스레드 세이프가 아니지만 DateTimeFormatter 는 스레드 세이프하여 재사용이 가능하다.

DateTimeFormatter 클래스에는 몇 가지 패턴에 대응한 인스턴스가 정의되어 있다.

|인스턴스|설명|
|---|---|
|ISO_LOCAL_DATE|yyyy-MM-dd|
|ISO_LOCAL_TIME|HH:mm, HH:mm:ss|
|ISO_LOCAL_DATE_TIME|yyyy-MM-dd'T'HH:mm:ss|

```java
TemporalAccessor parsed = DateTimeFormatter
        .ISO_LOCAL_DATE
        .parse("2017-02-25");
LocalDate date = LocalDate.from(parsed);
```

## 자바의 값 전달

### 기본형과 참조형

파라미터로 전달한 인수가 기본형이면 값이 변경되어도 원래의 값은 변하지 않는다.

파라미터로 전달한 인수가 참조형이면 값이 변경되었을 때 원래의 값도 변한다.

따라서

- 원칙적으로는 인수 객체의 수정은 피한다.
- 반환값이 void 인 경우 인수 객체를 수정한다.
- 반환값이 void 가 아니면 인수 객체를 수정하지 않는다.

### 불변(Immutable) 클래스와 변경 가능(Mutable) 클래스

객체의 값을 변경할 수 없도록 하는 것이 불변 클래스이다.

반면에 객체의 값을 변경할 수 있는 클래스는 변경 가능한 클래스이다.

객체 자신의 값을 변경하는 메서드가 하나라도 존재하는 경우, 그 클래스는 변경 가능 클래스라고 볼 수 있다.

불변 객체 클래스가 좋은 이유는 의도하지 않은 변경에 의한 버그를 일으키지 않기 때문이다.

물론 불변 객체 클래스도 대량으로 생성되면 성능이 저하된다는 문제가 있다.

## 가시성

### 가시성과 접근 제한자

|가시성|설명|접근 제한자|
|---|---|---|
|public|모든 클래스에서 이용 가능|public|
|protected|서브 클래스, 동일 패키지의 클래스가 이용 가능|protected|
|package private|동일 패키지의 클래스가 이용 가능|\-|
|private|자신의 클래스만 이용 가능|private|

원칙적으로 가장 범위가 좁은 가시성으로 선언한다.

확작성을 높이기 위해서 protected 로 한다.

테스트 용이성을 높이기 위해서 protected 로 한다.

### 인터페이스의 디폴트 구현

```java
public interface List<E> extends Collection<E> {
    // ...
    default void sort(Comparator<? super E> c) {
        Collections.sort(this, c);
    }
    // ...
}
```

인터페이스의 디폴트 구현은 java.util.List 인터페이스의 '이전 버전과의 호환성'을 위해 태어난 것이라고 볼 수 있다.

따라서 공통의 구현을 가진 클래스를 생성하고 싶은 경우에 인터페이스의 디폴트 구현보다는 추상 클래스를 만드는 편이 좋다.

### 인터페이스의 static 메서드

인터페이스에 static 메서드를 정의할 수 있게 됨으로써 인터페이스를 구현하는 클래스의 인스턴스를 반환하는 팩토리 메서드를 인터페이스에 정의할 수 있게 되었다.

```java
public interface Bar {
    String say();
    
    static Bar newInstance(String message) {
        return new DefaultBar(message);
    }
}

class DefaultBar implements Bar {
    private String message;
    
    DefaultBar(String message) {
        this.message = message;
    }
    
    @Override
    public String say() {
        return this.message;
    }
    
}

Bar bar = Bar.newInstance("Hello");
// Hello
```

static 메서드를 정의할 수 있게 됨으로써 사용하는 측에서 구현 클래스를 의식할 필요가 없게 되었다.

### 스레드 세이프

여러 스레드에서 읽거나 쓸 경우에도

- 데이터가 파괴되지 않는다.
- 처리 오류가 발생하기 않는다.
- 교착 상태/처리 정지가 발생하지 않는다.

스레드 세이프 하지 않는 경우

- int 의 증가 처리
- SimpleDateFormat 의 partse 메서드
- HashMap 의 put 메서드
- ArrayList 의 add/remove 메서드
- javax.xml.bind.Marshaller 의 marshal 메서드
- long(원시 변수)로의 대입

## 스레드 세이프 프로그래밍을 위해

상태를 유지하지 않게(stateless) 한다.

불필요한 인스턴스 변수를 사용하지 않게 한다.

'메서드 단위'가 아니라 최소한의 '일련의 처리'에 대해 동기화한다.

synchronized 블록으로 묶을 수 있다.

- 서로 다른 클래스의 synchronized 메서드는 동기화하지 않는다.
- 동일한 잠금 객체에 대한 처리만 동기화된다.
- 동일 클래스의 동일 메서드라도 인스턴스가 다르면 잠금은 걸리지 않는다.
- 동일한 처리라도 잠금 객체의 인스턴스가 다르면 잠금은 걸리지 않는다.

## 디자인 패턴

### 생성에 관한 패턴

|패턴|개요|
|---|---|
|AbstractFactory|관련된 일련의 인스턴스를 상황에 따라 적절하게 생성하는 방법을 제공|
|Builder|복합화된 인스턴스의 생성 과정을 은폐|
|Singleton|특정 클래스에 대해 인스터스가 하나임을 보장|

### 구조에 관한 패턴

|패턴|개요|
|---|---|
|Adapter|인터페이스에 호환성이 없는 클래스들을 조합시키기|
|Composite|재귀적인 구조의 취급을 쉽게 하기|

### 행동에 관한 패턴

|패턴|개요|
|---|---|
|Command|\'명령\'을 인스턴스로 취급함으로써 처리의 조합 등을 쉽게 한다.|
|Strategy|전략을 간단히 전환할 수 있는 구조를 제공한다.|
|Iterator|보유하는 인스턴스의 각 요소에 순차적으로 액세스하는 방법을 제공한다.|
|Observer|어떤 인스턴스의 상태가 변화할 때 그 인스턴스 자신이 상태의 변화를 통지하는 구조를 제공한다.|

### Checkstyle
읽기 쉬운 코드를 작성할 수 있도록 도와주는 툴이다.

자바의 소스 코드의 포맷을 체크하여 규칙에 따르지 않은 기술에 대해서는 경고해 주는 도구이다.

[https://juneyr.dev/checkstyle](https://juneyr.dev/checkstyle)

### FindBugs

일정한 패턴의 버그를 검출하고, 프로그램의 품질을 높이는데 도움이 되는 도구가 FindBugs 이다.

### Jenkins 빌드

jenkins 설치한다.

[https://github.com/mywisejin/JavaBook.git](https://github.com/mywisejin/JavaBook.git) 을 취득원으로 설정한다.

## Apache Commons

### commons-lang3

- 빈문자 판정
- hashCode / equals 구현

### commons BeanUtils

- copyProperties : 객체 복사

### Jackson

JSON 문자열 -> 객체

```java
File file = new File("employee.json");
ObjectMapper mapper = new ObjectMapper();
Employee employee = mapper.readValue(file, Employee.class);
```

객체 -> JSON 문자열

```java
Employee employee = new Employee;
File file = new File("newEmployee.json");
ObjectMapper mapper = new ObjectMapper();
mapper.writeValue(file, empolyee);
```

## Logger

### 로그와 레벨

|레벨|이용 상황|예|
|---|---|---|
|ERROR|오류가 발생하고 작업을 계속할 수 없는 경우|필수 데이터 취득 불가, 외부 시스템과의 통신 오류|
|WARN|오류가 발생했지만 계속 처리할 수 있는 경우|설정 파일이 없으면 기본값 사용|
|INFO|정상 동작 중에 상태 전이/처리가 실시된 경우|외부 시스템과의 통신 시작/종료, 일련의 업무 처리 종료|
|DEBUG|동적 확인을 위한 상세한 값|데이터 업데이트 시의 값|

패키지별로 출력 로그 레벨 변경가능

```xml
<configuration>
	<logger name="{패키지}" level="DEBUG">
	<root level="INFO">
		<appender-ref ref="FILE" />
	</root>
</configuration>
```

