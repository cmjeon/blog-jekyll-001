---
layout: single
title: "JUnit 5 알아보기"
categories: 
  - "learn etc"
tags: 
  - "JUnit5"
---

# JUnit 5 알아보기

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

# 참고

- [https://junit.org/junit5/docs/current/user-guide/](https://junit.org/junit5/docs/current/user-guide/){:target="_blank"}
- [https://www.inflearn.com/course/the-java-application-test/](https://www.inflearn.com/course/the-java-application-test/){:target="_blank"}
- [https://docs.google.com/document/d/1j6mU7Q5gng1mAJZUKUVya4Rs0Jvn5wn_bCUp3rq41nQ/edit](https://docs.google.com/document/d/1j6mU7Q5gng1mAJZUKUVya4Rs0Jvn5wn_bCUp3rq41nQ/edit){:target="_blank"}
