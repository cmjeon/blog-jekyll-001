---
layout: single
title: "TDD 알아보기"
categories: 
  - "learn etc"
tags: 
  - "TDD"
---

## TDD 란?

Test Driven Development

테스트 코드를 먼저 만들고 프로덕션 코드를 나중에 만드는 개발 방법

> 프로그램을 작성하기 전에 테스트를 먼저 작성하라 - 켄트 벡

짧은 주기의 피드백을 받으며 프로그래미가 더 큰 코드를 작성해 나가는 개발방법론

언제나 적한한 도구는 아니지만 그래도 이만한게 없다

## TDD 3가지 법칙

1. 실패하는 테스트 없이는 프로덕션 코드 만들지 마라
2. 실패라는 것을 표현할만큼만 테스트 코드를 작성하라
3. 테스트를 통과할만큼만 프로덕션 코드를 작성하라

한마디로 너무 앞서나가지 마라는 뜻 (오버엔지니어링 하지마)

## TDD 개발 사이클 : red, green, refactor

3가지 모자로 비유됨

1. red : 실패하는 테스트를 구현
2. green : 테스트가 성공하도록 프로덕션 코드를 구현
3. blue : 프로덕션 코드와 테스트 코드를 리팩토링

refactoring is mandatory not optional

TDD 는 설계방법론이 아님 -> TDD 를 한다고 좋은 설계 산출물이 나오는건 아님

## TDD 원칙

쉬운 테스트부터, 아주 간단한 기능부터 -> 코드를 점진적으로 발전시킬 수 있음

테스트를 성공시키기 위한 최소한의 프로덕션 코드 little golf game

테스트코드가 구체화 specific 될 수록, 프로덕션 코드는 점점 범용적 generic 으로 변함 -> 모든 테스트 케이스를 수용할 수 있는 프로덕션 코드가 되려면

## TDD 잇점

시간이 지날수록 테스트 커버리지가 높아짐

자연스럽게 요구사항에 딱맞게 코드를 작성하게 됨 -> 오버 엔지니어링 방지

### 디버깅 시간 단축

배포 & 테스트 & 변경 & 배포 & 테스트 ... -> TDD 를 쓰면 동작하는 코드를 만드는데 시간을 더 쓸 수 있음

### 동작하는 문서 역할

테스트 케이스를 보면 기능을 알 수 있게 됨

### 낮은 결합도 Loose coupling

테스트 코드를 먼저 작성하면 프로덕션 코드는 테스트하기 쉽게 만들어짐

테스트를 하기 위해서는 프로덕션 코드가 다른 프로덕션 코드들과 격리되어야 함 -> 자연스럽게 loose coupling 된 코드를 만듬

[https://www.leafcats.com/68](https://www.leafcats.com/68){:target="_blank"}

### 변경에 대한 용기

TDD 는 변경에 대한 두려움을 줄여줌. -> 리팩토링이 편해짐

회귀테스트 regression test 보장 : 이전에 돌았던 기능들을 검증하는 테스트 -> 회귀테스트에서 실패가 잦아지면 불신이 자람

설계가 좋았더라도 변경이 두렵다면 리팩토링이 불가능 -> 코드가 썪음

반대로 설계가 안좋더라도 변경이 두렵지 않다면 리팩토링 가능 -> 코드를 살릴 수 있음

### 신뢰성 보장

프로덕션을 구현하고 테스트 케이스를 만든다면? 그것을 Test After Development(TAD) 라고 부름 -> 나쁜 케이스

TAD 는 프로덕션 코드에 딱 맞는 테스트 케이스만을 해 봄 -> 테스트 안해본 부분에서 버그 발생 -> 신뢰할 수 없는 코드

## TDD 가 실패하는 이유

코드가 이루고자 하는 가치나 기능을 테스트하기 보다는 그 기능을 어떻게 구현하고 있는지를 테스트하기 때문임

구현체와 결합도가 높은 테스트 케이스를 작성. 그래서 구현체를 리팩토링하면 테스트 케이스가 깨짐

-> 구현보다는 인터페이스를 테스트 해라, 가치나 기능을 테스트

## 단위테스트 Unit test

무엇이 단위인가? 소스 한 줄? 메서드? 클래스?

마틴파울러는 행동을 단위로 보았다? -> 그러면 메서드? -> 인터페이스?

[https://martinfowler.com/bliki/UnitTest.html](https://martinfowler.com/bliki/UnitTest.html){:target="_blank"}

실제 깨체를 사용한 테스트 : Sociable Test -> Classicist, 상태검증, Inside-out,

가짜 객체(Double) 을 사용한 테스트 : Solitary Test -> Mockist, 행위검증, Outside-in,

## Test Double

Double 은 실제 객체를 사용하기 어려울 경우 사용하는 가짜 객체를 일컬음

종류 : Dummy, Spy, Fake, Stub, Mock

Mock 은 호출하는 메소드의 반환값을 프로그래밍 해놓은 객체

## SUT 와 Collaborator

SUT System Under Test : 테스트 하려는 그 객체

Collaborator : SUT 가 의존하는 객체

## 상태검증과 행위검증

테스트 성패는 무엇이 기준인가?

상태검증 : 객체의 상태를 직접적으로 검증 -> Classicist 

행위검증 : 행동이 이루어졌는지 검증 -> Mockist

## Fixture 와 Mock

사전준비하는 방법

Fixture : Collaborator 를 직접 생성, 복잡, 재사용 가능

Mock : Mock 객체를 생성, 간단, 재사용 불가

## Inside-out 와 Outside-in

### Inside-out(Classicist)

개발이 내부(도메인)에서 시작해서 점점 외부(API)로 구현 

시스템 전반에 대한 완전한 이해없이 시작 가능

### Outside-in(Mockist)

개발이 외부(API)에서 시작해서 점점 세부적(도메인)인 사항을 구현

객체지향적인 코드 작성이 가능

## 단위테스트 목적

문제점 발견이 쉽고, 변경이 가능하다.

-> 품질 향상, 코드의 문서화

## 좋은 단위테스트

F - Fast 빠르게 : 테스트가 느리면 테스트를 돌릴 엄두가 안나게 된다.

I - Independent 독립적으롤 : 각 테스트가 독립적이지 않다면 실패 시 연달아 실패하게 되고 실패 원인을 찾기 어려워진다.

R - Repeatable 반복 가능하게 : 어떤 환경에서도 반복 가능하게, 네트워크가 연결되어 있지 않더라도. 

S - Self-Validating 자가검증하는 : 테스트는 오직 true/false 로 구분한다. 사람의 주관적 판단 배제

T - Timey 적시에 : 실제 코드를 구현하기 직전에 구현해야 함 -> 먼저 작성한 프로덕션 코드가 테스트하기 힘든 코드일 수 있음

# 참고

- [https://www.youtube.com/watch?v=n01foM9tsRo](https://www.youtube.com/watch?v=n01foM9tsRo){:target="_blank"}
- [https://www.youtube.com/watch?v=wmHV6L0e1sU&list=PLeQ0NTYUDTmMM71Jn1scbEYdLFHz5ZqFA&index=10](https://www.youtube.com/watch?v=wmHV6L0e1sU&list=PLeQ0NTYUDTmMM71Jn1scbEYdLFHz5ZqFA&index=10){:target=_blank"}
- [https://www.youtube.com/watch?v=3LMmPXoGI9Q](https://www.youtube.com/watch?v=3LMmPXoGI9Q){:target="_blank"}
- [https://velog.io/@codemcd/OKKYCON-2018-이규원-당신들의-TDD가-실패하는-이유-u2k4w01f6h](https://velog.io/@codemcd/OKKYCON-2018-이규원-당신들의-TDD가-실패하는-이유-u2k4w01f6h){:target="_blank"}
- [https://www.youtube.com/watch?v=gX9PkEibWsM](https://www.youtube.com/watch?v=gX9PkEibWsM){:target="_blank"}
- [https://www.youtube.com/watch?v=Npi21gLIEZM](https://www.youtube.com/watch?v=Npi21gLIEZM){:target="_blank"}
- [https://www.leafcats.com/68](https://www.leafcats.com/68){:target="_blank"}
