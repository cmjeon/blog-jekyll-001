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

JUnit Jupiter 의 ExecutionCondition 확장 API 를 사용하면 

> IntelliJ 에서 Gradle 실행 시 Display Name Generators 가 작동하지 않으면 preferences > Build, Executrion, Deployment > Build Tools > Gradle 에 들어가서 Run Tests using 을 IntelliJ IDEA 로 변경 













# 참고

- [https://junit.org/junit5/docs/current/user-guide/](https://junit.org/junit5/docs/current/user-guide/){:target="_blank"}
- [https://www.inflearn.com/course/the-java-application-test/](https://www.inflearn.com/course/the-java-application-test/){:target="_blank"}
- [https://docs.google.com/document/d/1j6mU7Q5gng1mAJZUKUVya4Rs0Jvn5wn_bCUp3rq41nQ/edit](https://docs.google.com/document/d/1j6mU7Q5gng1mAJZUKUVya4Rs0Jvn5wn_bCUp3rq41nQ/edit){:target="_blank"}
