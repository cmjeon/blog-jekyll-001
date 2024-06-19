---
# layout: single
title: "6장 AOP - 6.2 고립된 단위 테스트"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

## 6.2 고립된 단위 테스트

가장 편하고 좋은 테스트 방법은 가능한 한 작은 단위로 쪼개서 테스트하는 것이다.

### 6.2.1 복잡한 의존관계 속의 테스트

UserService 의 구현 클래스가 동작하려면 세 가지 타입의 의존 오브젝트가 필요하다.

원래 UserServiceTest 가 테스트하고자 하는 대상인 UserService 는 사용자 정보를 관리하는 비즈니스 로직의 구현코드이다.

UserService 구현 클래스의 코드가 바르게 작성되어 있다면 성공하고, 아니면 실패하면 된다.

하지만 UserService 는 UserDao, TransactionManager, MailSender 라는 세 가지 의존관계를 가지고 있어서 테스트가 진행되는 동안 이 오브젝트들이 같이 실행된다.

이런 경우는 테스트 준비가 힘들고, 동일한 결과가 나오지 않는 등 부작용이 발생할 수 있어서 테스트를 작성하고 실행하는 것을 기피하는 원인이 된다.

### 6.2.2 테스트 대상 오브젝트 고립시키기

테스트의 대상이 환경이나, 외부 서버, 다른 클래스의 코드에 종속되고 영향을 받지 않도록 고립시킬 필요가 있다.

테스트를 의존 대상으로부터 분리해서 고립시키는 방법은 테스트를 위한 대역 Mock 을 사용하는 것이다.

#### 테스트를 위한 UserServiceImpl 고립

고립된 테스트가 가능하도록 UserService 를 재구성해보자.

UserServiceImpl 을 MockUserDao 와 MockMailSender 에 의존하게 한다.

트랜잭션 관련코드가 분리되어 UserServiceImpl 에서는 더이상 PlatformTransactionManager 를 의존하지 않는다.

MockUserDao 는 코드가 정상적으로 수행되도록 도와주고, 부가적인 검증 기능까지 가지도록 만든다.

UserServiceImpl 과의 사이에서 주고받은 정보를 저장해뒀다가, 테스트의 검증에 사용할 수 있게 만들 필요가 있다.

#### 고립된 단위 테스트 활용












