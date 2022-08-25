---
layout: single
title: "JUnit 5 알아보기"
categories: 
  - "learn etc"
  - "Java Test"
tags: 
  - "JUnit5"
---

## JUnit 5 란?

[https://junit.org/junit5/docs/current/user-guide/](https://junit.org/junit5/docs/current/user-guide/){:target="_blank"}

Junit 5 는 자바의 테스팅 프레임워크 5번째 버전임

> The 5th major version of the programmer-friendly testing framework for Java and the JVM

JUnit 5 의 세부모듈

- Jupiter : TestEngine API 구현체로 JUnit 5 를 제공 -> 우리가 주로 사용할 구현체
    - Jupiter 의 친구들 : AssertJ, Hemcrest, Truth
- JUnit Platform : 실행가능한 런처 제공, TestEngine API 제공
- Vintage : JUnit 4, 3 를 지원하느 구현체

JUnit 5 는 Java 8 이상 버전에서 동작

## 시작하기

스프링부트 프로젝트 생성하는 방법 : https://start.spring.io/ 에서 Spring web 추가 

기존 프로젝트에 의존성 추가하는 방법

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.5.2</version>
    <scope>test</scope>
</dependency>
```

지난번 스프링부트로 쇼핑몰 만들기 프로젝트 With JPA 를 활용하여 테스트 작성법 확인

[https://github.com/cmjeon/spring-boot-project-jpa-001](https://github.com/cmjeon/spring-boot-project-jpa-001){:target="_blank"}

```bash
$ git checkout mock-testcase
```

## 테스트 관련 IntelliJ 단축키

- 테스트할 클래스를 열고 shift + command + t : 테스트 클래스 생성
- 메소드를 선택하고 shift + ctrl + r : 해당 메소드 테스트 케이스 실행
- 클래스를 선택하고 shift + ctrl + r : 해당 클래스 테스트 케이스 실행


## Annotation

대표적인 Annotation 을 소개한다.

전체 Annotation 은 [https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations){:target="_blank"} 에서 확인

@Test : 이 메소드가 테스트임을 나타낸다. denote

@ParameterizedTest : 메개변수화된 테스트임을 나타낸다.

@RepeatedTest : 반복테스트임을 나타낸다.

@DisplayName : 테스트에 표시될 이름을 선언한다.

@BeforeEach : 각 @Test, @RepeatedTest, @ParameterizedTest, @TestFactory 들 전에 실행됨을 나타낸다.

@AfterEach : 각 @Test, @RepeatedTest, @ParameterizedTest, @TestFactory 들 후에 실행됨을 나타낸다.

@BeforeAll : 모든 @Test, @RepeatedTest, @ParameterizedTest, @TestFactory 들 전에 실행됨을 나타낸다. 

@AfterAll : 모든 @Test, @RepeatedTest, @ParameterizedTest, @TestFactory 들 후에 실행됨을 나타낸다.

@Tag : 테스트 필터링을 위해 선언한다.

@Disabled : 테스트 메소드가 아님을 나타냄

@Timeout : 테스트에 성공에 주어진 시간을 나타냄 

### Meta-Annotations(Composed Annotations)

커스텀 Annotation 을 만들 수 있는 방법

@Fast 만들기

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.junit.jupiter.api.Tag;

@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Tag("fast")
public @interface Fast {
}
```

@Fast 사용하기

```java
@Fast
@Test
void myFastTest() {
    // ...
}
```

## Definitions 정의

### Platform Concepts

컨테이너 : 다른 컨테이너나 테스트를 자식으로 가지고 있는 테스트 트리의 노드

테스트 : 실행되어 예상되는 동작을 확인하는 테스트 트리의 노드

### Jupiter Concepts

라이프사이클 메서드 : @BeforeAll, @AfterAll, @BeforeEach, @AfterEach 같은 메소드

테스트 클래스 : 하나 이상의 테스트 메서드나 컨테이너를 포함하는 최상위 클래스, static or @Nested 클래스

테스트 메서드 : @Test, @RepeatedTest, @ParameterizedTest, @TestFactory, @TestTemplate 으로 지정된 메소드

## Test Classes And Methods

테스트 메서드, 라이프사이클 메서드는 테스트 클래스에 선언되거나, 수퍼클래스에서 상속되거나, 인터페이스에서 구현되거나 할 수 있다.

테스트 메서드, 라이프사이클 메서드는 abstract 가 아니어야 하고 리턴값이 void 이어야 한다. 단, 값을 반환하는 @TestFactory 는 제외

테스트 클래스, 테스트 메서드, 라이프사이클 메서드는 public 일 필요는 없지만 private 는 안된다. -> 일반적으로는 public 도 생략하는 것이 좋다

## Display Names

테스트 클래스, 테스트 메서드의 이름을 지정하는 Annotation

@DisplayNameGenerators 보다 우선적으로 표현된다.

### Display Name Generators

@DisplayNameGeneration 를 통해 이름을 자동으로 생성할 수 있다.

DisplayNameGenerator 종류

- Standard : argument 를 포함한 메서드명 그대로 표현
- Simple : 메서드명에서 뒤에 () 제거하고 표현
- ReplaceUnderscores : _ 를 공백으로 표현
- IndicativeSentences : 테스트 클래스명 + separator + 메소드명으로 표현
    
  ```java
  @IndicativeSentencesGeneration(separator = " -> ", generator = DisplayNameGenerator.ReplaceUnderscores.class)
  class this_is_my_test_class {
    @Test
    void it_is_my_test_method() {
      
    }
  }
  ```
  
### Setting the Default Display Name Generator

Display Name Generator 는 properties 에 등록하여 기본 생성자로 선언할 수 있다.

```properties
# src/test/resources/junit-platform.properties
junit.jupiter.displayname.generator.default = \
    org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
```

Display Name 은 아래와 같은 순서로 표현됨

1. @DisplayName
2. @DisplayNameGeneration
3. default DisplayNameGenerator
4. org.junit.jupiter.api.DisplayNameGenerator.Standard

> IntelliJ 에서 Gradle 실행 시 Display Name Generators 가 작동하지 않으면 preferences > Build, Executrion, Deployment > Build Tools > Gradle 에 들어가서 Run Tests using 을 IntelliJ IDEA 로 변경

## Assertions

JUnit Jupiter 가 제공하는 기능

모든 Jupiter Assertion 은 org.junit.jupiter.api.Assertions 클래스의 static 메서드이다.

Java 8 lambdas 사용가능

### Kotlin Assertion Support

JUnit Jupiter 는 Kotlin 에서 사용가능한 assertion 메소드도 제공한다.

### Third-party Assertion Libraries

JUnit Jupiter 외에 다른 기능이 필요할 수도 있다(ex matcher)

이럴 때는 AssertJ, Hamcrest, Truth 를 사용할 수 있다.

matcher 와 풍부한 API 의 조합을 사용해서 assertion 을 더 설명적이고 읽기 쉽게 만들 수 있다.

JUnit Jupiter 의 Assertion 클래스는 assertThat 메서드를 제공하지 않기 때문에 Thrid-party 의 matcher 를 사용하는 것이 좋다.

## Assumptions

JUnit Jupiter 의 assumptions 은 람다표현식 가능하고, static 이어야 한다.

## Disabling Tests

클래스나 메서드에 @Disabled Annotation 으로 비활성화 할 수 있다.

## Conditional Test Execution

JUnit Jupiter 의 ExecutionCondition 은 특정 기반으르 테스트틀 활성화/비활성화 할 수 있다. (ex @Disabled)

org.junit.jupiter.api.condition 패키지에서 선언적으로 컨테이너 및 테스트를 활성화/비활성화할 수 있는 조건을 지원한다.

여러 ExecutionCondition 이 등록되면 조건 중 하나라도 비활성화되면 테스트는 비활성화 된다.

### Operating System and Architecture Conditions

@EnabledOnOs, @DisabledOnOs 으로 특정 운영체제나 아키텍쳐 에서 활성화/비활성화 할 수 있다.

### Java Runtime Environment Conditions

@EnabledOnJre, @DisabledOnJre 으로 특정 JRE 에서 활성화/비활성화 할 수 있다.

@EnabledForJreRange, @DisabledForJreRange 로 특정 범위안의 JRE 에서 활성화/비활성화 할 수 있다. (기본적으로 가장 하단은 JAVA_8)

### System Property Conditions

@EnabledIfSystemProperty, @DisabledIfSystemProperty 로 JVM System Property 의 값을 기반으로 활성화/비활성화 할 수 있다.

### Environment Variable Conditions

@EnabledIfEnvironmentVariable, @DisabledIfEnvironmentVariable 로 환경변수의 값을 기반으로 활성화/비활성화 할 수 있다.

### Custom Conditions

@EnabledIf 로 조건 메서드를 기반으로 활성화/비활성화 할 수 있다.

조건 메서드는 테스트 클래스 밖에 있을 수 있다.

## Tagging and Filtering

@Tag 를 통해 테스트의 검색, 실행을 필터링하는데 사용할 수 있다.

## Test Execution Order

테스트 클래스, 테스트 메서드의 순서를 보장되지는 않는다.

### Method Order

실행 순서를 적용해야하는 경우에 사용에 사용한다.

@TestMethodOrder Annotation 을 지정하고 MethodOrderer 구현체를 지정한다.

MethodOrderer 인터페이스의 구현체를 만들거나 내장된 MethodOrderer 구현체를 사용할 수 있다.

- DisplayName: DisplayName 영문자 순으로
- MethodName: 메서드명 영문자순으로(6.0 에서 제거될 예정)
- OrderAnnotation: @Order 에 정의된 순서 순으로
- Random: 무작위 순으로
- Alphanumeric: 메서드명과 parameter 목록 순으로

### Setting the Default Method Orderer

Method Orderer 는 properties 에 등록하여 기본 생성자로 선언할 수 있다.

```properties
# src/test/resources/junit-platform.properties
junit.jupiter.testmethod.order.default = \
    org.junit.jupiter.api.MethodOrderer$OrderAnnotation
```

### Class Order

테스트 클래스는 실행 순서에 의존하지 않지만 어떤 경우에는 실행 순서를 적용해야할 때도 있다.

테스트 클래스는 빌드 시간을 최적화하기 위해 아래 시나리오 모드를 사용

- fail fast: 이전에 실패한 테스트와 더 빠른 테스트 먼저 실행
- shortest test plan execution duration: 병렬 실행이 활성화된 상태에서 긴 테스트를 먼저 실행

ClassOrderer 인터페이스를 구현한 구현체를 사용한다.

사용자 정의 ClassOrderer 를 구현하거나 다음 내장 ClassOrderer 구현체 중 하나를 사용한다.

- ClassName: 클래스명 영문자 순으로
- DisplayName: DisplayName 영문자 순으로
- OrderAnnotation: @Order 에 정의된 순서 순으로
- Random: 무작위 순으로

Class Orderer 는 properties 에 등록하여 기본 생성자로 선언할 수 있다.

```properties
# src/test/resources/junit-platform.properties
junit.jupiter.testclass.order.default = \
    org.junit.jupiter.api.ClassOrderer$OrderAnnotation
```

ClassOrderer 는 모든 테스트 클래스 @Nested 클래스에 적용

> 최상위 클래스들 끼리 순서가 정렬되고, @Nested 클래스는 상위 클래스의 다른 @Nested 클래스와 관련하여 순서가 정렬된다.

@Nested 클래스의 실행 순서를 구성하려면 상위 클래스에 @TestClassOrder 를 선언한다.

## Test Instance Lifecycle

JUnit 은 각 테스트 메서드를 실행하기 위해 각 테스트 클래스의 새 인스턴스를 만들어서 실행한다.

이 메서드별 테스트 인스턴스의 생명 주기는 JUnit Jupiter 의 기본 동작이다.

JUnit Jupiter 가 동일한 테스트 인스턴스에서 모든 테스트 메서드를 실행하도록 하려면 @TestInstance(LifeCycle.PER_CLASS) annotation 을 추가핸다.

PER_CLASS 라면 테스트 메서드의 상태를 @BeforeEach, @AfterEach 에서 재설정해야 할 수 도 있다.

PER_CLASS 모드는 PER_METHOD 모드에 비해 @BeforeAll, @AfterAll 을 static 이 아닌 메서드로 선언가능한 이점이 있다.

그래서 @Nested 테스트 클래스에서 @BeforeAll, @AfterAll 메서드를 사용할 수 있다.

### Changing the Default Test Instance Lifecycle

@TestInstance annotation 이 없으면 기본 생명 주기 모드인 PER_METHOD 를 사용한다. 

Junit.jupiter.testinstance.lifecycle.default 의 구성 매개변수를 설정하여 기본 테스트 인스턴스 생명 주기 모드를 변경할 수 있다.

```properties
# src/test/resources
junit.jupiter.testinstance.lifecycle.default = per_class
```

JVM 설정에서 변경할 수도 있다.

```java
# JVM 설정
Djunit.jupiter.testinstance.lifecycle.default=per_class
```

## Nested Tests

@Nested 테스트는 테스트 작성자에게 여러 테스트 그룹간의 관계를 표현할 수 있는 기능을 제공한다.

외부 테스트의 전제 조건은 내부 테스트에 사용됩니다.

예컨데 @Nested 안에 @BeforeEach annotation 은 하위 테스트에 사용된다.

상위 테스트의 설정 코드가 항상 실행되기 때문에 하위 테스트만 실행할 수도 있다.

static 이 아닌 테스트 클래스만 @Nested 를 사용할 수 있다.

내부 클래스의 정적 메서드를 허용하지 않기 때문에 @Nested 클래스는 @BeforeAll, @AfterAll 이 기본적으로 작동하지 않는다.

@Nested 테스트 클래스에 @TestInstance(Lifecycle.PER_CLASS) annotation 을 추가해서 우회할 수 있다.

## Dependency Injection for Constructors and Methods

JUnit Jupiter 의 주요 변경사항 중 하나는 테스트 생성자와 테스트 메서드 모두 parameter 를 가질 수 있다는 것이다.

ParameterResolver 는 런타임에 parameter 를 동적으로 처리할 수 있다.

3개의 내장 resolver 가 있다.

- TestInfoParameterResolver: 생성자나 메서드의 parameter 가 TestInfo 형이면 TestInfoParameterResolver 는 TestInfo 인스턴스를 제공한다.
- RepetitionInfoParameterResolver: @RepeatedTest, @BeforeEach, @AfterEach 메서드의 parameter 가 RepetitionInfo 형이면 RepetitionInfoParameterResolver 는 RepetitionInfo 인스턴스를 제공한다.
- TestReporterParameterResolver: 생성자나 메서드의 parameter 가 TestReporter 형이면 TestReporterParameterResolver 는 TestReporter 인스턴스를 제공한다.

## Test Interfaces and Default Methods

@ExtendWith, @Tag 를 테스트 인터페이스에 선언해서 테스트 인터페이스를 구현하는 클래스가 자동으로 상속하도록 할 수 있다.

## Repeated Tests

@RepeatedTest 로 원하는 횟수만큼 테스트를 반복할 수 있다.

아래의 placeholders 를 지원한다.
- {displayName}: 테스트 이름
- {currentRepetition}: 현재 반복수
- {totalRepetitions}: 총 반복횟수

기본 표시이름은 "repetition {currentRepetition} of {totalRepetitions}" 이다.

RepeatedTest.LONG_DISPLAY_NAME 은 "{displayName} :: repetition {currentRepetition} of {totalRepetitions}" 이다.

@RepeatedTest, @BeforeEach, @AfterEach 에는 RepetitionInfo 인스턴스를 주입할 수 있다.

### Repeated Test Examples

RepetitionInfo 인스턴스를 사용하여 테스트 이름을 사용자 지정 이름으로 표시하는 예시를 보여준다.

[https://junit.org/junit5/docs/current/user-guide/#writing-tests-repeated-tests-examples](https://junit.org/junit5/docs/current/user-guide/#writing-tests-repeated-tests-examples){:target="_blank"}

## Parameterized Tests

다른 변수로 여러번 테스트 하는 것이 가능하다.

@Test 대신 @ParameterizedTest 를 사용한다.

최소 하나의 Source annotation 을 선언해줘야 한다.

### Required Setup

@ParameterizedTest 를 사용하려면 junit-jupiter-params artifact 를 dependency 에 추가해야 한다.

### Consuming Arguments

@ParameterizedTest 는 일반적으로 인수 소스 인덱스, 매개변수 인덱스의 1:1 관계인 인수를 사용한다.

ParameterizedTest 에서 집계 aggregate 된 인수를 사용할 수도 있다.

ParameterResolver 에서 추가 인수를 제공할 수도 있다.

@ParameterizedTest 메서드는 다음 규칙을 따르는 매개변수가 선언되어야 한다.

- 0개 이상의 인덱싱된 인수를 먼저 선언해야 한다.
- 0개 이상의 aggregator 가 선언되어야 한다.
- ParameterResolver 에서 제공하는 0개 이상의 인수는 마지막에 선언된다.

인덱스된 인수는 ArgumentsProvider 에 의해 공급되는 인수이다.

매개변수 목록에서 동일한 인덱스의 인수들이 parameterized methods 에 전달된다.

aggregator 는 argumentsAccessor 형의 매개변수이거나 @AggregateWith annotation 이 있는 매개변수이다.

### Sources of Arguments

#### @ValueSource

단일 배열을 지정할 수 있으며 테스트 호출당 단일인수를 제공한다.

```java
@ParameterizedTest
@ValueSource(ints = { 1, 2, 3 })
void testWithValueSource(int argument) {
    assertTrue(argument > 0 && argument < 4);
}
```

#### Null And Empty Sources

- @NullSource: 테스트 메소드에 null 인수를 제공한다.
- @EmptySource: 테스트 메소드에 빈 인수를 제공한다.
- @NullAndEmptySource: @NullSource, @EmptySource 의 합성 annotation

#### @EnumSource

Enum 을 상수를 소스로 제공한다.

```java
@ParameterizedTest
@EnumSource(ChronoUnit.class)
void testWithEnumSource(TemporalUnit unit) {
    assertNotNull(unit);
}
```

annotation 의 값은 선택사항이고, 생략하면 메서드의 첫번째 매개변수의 형식을 사용된다.

테스트 메서드의 매개변수를 ChronoUnit 으로 변경하면 @EnumSource 의 값은 생략가능하다.

[https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources-EnumSource](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources-EnumSource){target="_blank"}

#### @MethodSource

@MethodsSource 는 다른 팩토리 메서드를 참조할 수 있다.

```java
@ParameterizedTest
@MethodSource("stringProvider")
void testWithExplicitLocalMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> stringProvider() {
    return Stream.of("apple", "banana");
}
```

Collection, Iterator, Iterable, 객체배열 등 Stream 으로 변환가능한 것을 리턴하면 된다.

팩토리 메서드 이름이 제공되지 않으면 @ParameterizedTest 와 동일한 이름의 메서드를 참조한다.

[https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources-MethodSource](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources-MethodSource){target="_blank"}

#### @CsvSource

@CsvSource 를 사용해서 여러 인수를 테스트에 사용할 수 있다.

```java
@ParameterizedTest
@CsvSource({
    "apple,         1",
    "banana,        2",
    "'lemon, lime', 0xF1",
    "strawberry,    700_000"
})
void testWithCsvSource(String fruit, int rank) {
    assertNotNull(fruit);
    assertNotEquals(0, rank);
}
```

첫번째 레코드는 CSV 헤더로 사용될 수 있다.

```java
@ParameterizedTest(name = "[{index}] {arguments}")
@CsvSource(useHeadersInDisplayName = true, textBlock = """
    FRUIT,         RANK
    apple,         1
    banana,        2
    'lemon, lime', 0xF1
    strawberry,    700_000
    """)
void testWithCsvSource(String fruit, int rank) {
    // ...
}
```

기본 구분자는 쉼표(,) 이지만 delimiter 속성을 이용해 다른 문자를 사용할 수 있고, delimiterString 을 사용해서 문자열을 구분자로 사용할 수 있다.

기본적으로 작은 따옴표를 사용하지만 quoteCharacter 속석으로 변경할 수 있다.

useHeadersInDisplayName 속성으로 인수를 테스트이름에 표현할 수 있다.

#### @CsvFileSource

@CsvFileSource 를 사용하면 클래스 경로 또는 로컬 파일 시스템에서 파일을 사용할 수 있다.

첫번째 레코드는 CSV 헤더로 사용될 수 있다.

numLinesToSkip 속석으로 헤더를 무시하도록 할 수도 있다.

#### @ArgumentsSource

@ArgumentsSource 는 ArgumentProvider 클래스를 통해 인수를 공급한다.

ArgumentProvider 클래스는 ArgumentsProvider 인터페이스의 구현체로 만들어진다.

```java
@ParameterizedTest
@ArgumentsSource(MyArgumentsProvider.class)
void testWithArgumentsSource(String argument) {
    assertNotNull(argument);
}

public class MyArgumentsProvider implements ArgumentsProvider {

    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return Stream.of("apple", "banana").map(Arguments::of);
    }
}
```

### Argument Conversion

확대 변환 Widening Conversion : 상위크기의 컨버전을 지원한다.

- byte to short, int, long, float, or double
- short to int, long, float, or double
- char to int, long, float, or double
- int to long, float, or double
- long to float or double
- float to double

암시적 변환 Implicit Conversion : JUnit jupiter 는 암시적 유형 변환기를 제공한다.

[https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit){target="_blank"}

대체 문자열-객체 변환 : Fallback String-to-Object Conversion : 문자열을 객체로 바꿔준다.

> factory method : 단일 String 인수를 받고 대상 형식의 인스턴스를 반환하는 메서드. 대상 형식에 선언된 비공개가 아닌 정적 메서드
> 메서드의 이름은 임의적일 수 있다.

> factory constructor : 단일 String 인수를 허용하는 대상 형식에 선언된 비공개가 아닌 생성자

여러 개의 팩토리 메서드 있어도 무시됨.

팩토리 메서드와 팩토리 생성자가 함께 있으면 팩토리 생성자가 무시됨

```java
@ParameterizedTest
@ValueSource(strings = "42 Cats") // 문자열이 book 객체로 변환되었음
void testWithImplicitFallbackArgumentConversion(Book book) {
    assertEquals("42 Cats", book.getTitle());
}

public class Book {

    private final String title;

    private Book(String title) {
        this.title = title;
    }

    public static Book fromTitle(String title) {
        return new Book(title);
    }

    public String getTitle() {
        return this.title;
    }
}
```

명시적 변환 Explicit Conversion : @ConvertWith 을 사용하여 매개변수로 사용할 ArgumentConverter 를 지정할 수 있다.

```java
@ParameterizedTest
@EnumSource(ChronoUnit.class)
void testWithExplicitArgumentConversion(
        @ConvertWith(ToStringArgumentConverter.class) String argument) {

    assertNotNull(ChronoUnit.valueOf(argument));
}

public class ToStringArgumentConverter extends SimpleArgumentConverter {

    @Override
    protected Object convert(Object source, Class<?> targetType) {
        assertEquals(String.class, targetType, "Can only convert to String");
        if (source instanceof Enum<?>) {
            return ((Enum<?>) source).name();
        }
        return String.valueOf(source);
    }
}
```

### Argument Aggregation 

Custom Aggregators

### Customizing Display Names

### Lifecycle and Interoperability

## Test Templates

## Dynamic Tests

## Timeouts

## Parallel Execution

## Built-in Extensions





# 참고

- [https://junit.org/junit5/docs/current/user-guide/](https://junit.org/junit5/docs/current/user-guide/){:target="_blank"}
- [https://www.inflearn.com/course/the-java-application-test/](https://www.inflearn.com/course/the-java-application-test/){:target="_blank"}
- [https://docs.google.com/document/d/1j6mU7Q5gng1mAJZUKUVya4Rs0Jvn5wn_bCUp3rq41nQ/edit](https://docs.google.com/document/d/1j6mU7Q5gng1mAJZUKUVya4Rs0Jvn5wn_bCUp3rq41nQ/edit){:target="_blank"}
