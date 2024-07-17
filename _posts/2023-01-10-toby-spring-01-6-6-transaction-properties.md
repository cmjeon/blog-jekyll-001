---
# layout: single
title: "6장 AOP - 6.6 트랜잭션 속성"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

## 6.6 트랜잭션 속성

TransacitonAdvice 의 트랜잭션 경계설정 코드를 다시 살펴보자

```java
public Object invoke(MothodInvocation invocation) throws Throwable {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    // ...
}
```

transactionManager 로부터 트랜잭션을 가져올 때 파라미터로 전달하는 DefaultTransactionDefinition 의 용도가 무엇인지 알아보자

### 6.6.1 트랜잭션 정의

트랜잭션의 기본 개념은 '더 이상 쪼갤 수 없는 최소 단위의 작업' 이다.

DefaultTransactionDefinition 이 구현하고 있는 TransactionDefinition 인터페이스는 트랜잭션 동작방식에 영향을 줄 수 있는 네 가지 속성을 정의하고 있다.

#### 트랜잭션 전파

트랜잭션 전파 transaction propagation 란 트랜잭션의 경계에서 이미 진행 중인 트랜잭션이 있을 때 또는 없을 때 얼떻게 동작할 것인가를 결정하는 방식을 말한다.

- PROPAGATION_REQUIRED
  - 진행 중인 트랜잭션이 없으면 새로 시작하고, 이미 시작된 트랜잭션이 있으면 이에 참여한다.
  - DefaultTransactionDefinition 의 트랜잭션 전파 속성은 바로 이 PROPAGATION_REQUIRED 이다.
- PROPAGATION_REQUIRES_NEW
  - 진행 중인 트랜잭션이 있든 없든 항상 새로운 트랜잭션을 시작한다.
- PROPAGATION_NOT_SUPPORTED
  - 진행 중인 트랜잭션이 있든 없든 트랜잭션 없이 동작한다.
  - 특별한 메소드만 트랜잭션 적용에서 제외하고 싶을 때, 모든 메소드에 트랜잭션 AOP 가 적용되도록 포인트컷을 작성하고, 특정 메소드의 트랜잭션 전파 속성만 PROPAGATION_NOT_SUPPORTED 로 설정하는 식으로 만든다.

transactionManager 매니저를 통해 getTransaction() 메소드를 사용하는 이유는 트랜잭션 전파 속성이 있기 때문이다.

현재 진행 중인 트랜잭션이 존재하는지 여부와 트랜잭션 전파 속성에 따라 새로운 트랜잭션을 시작하거나 이미 진행중인 트랜잭션에 참여하기만 할 수도 있다.

#### 격리수준

모든 DB 트랜잭션은 격리수준 isolation level 을 갖고 있어야 한다.

서버환경에서는 여러 개의 트랜잭션이 동시에 진행될 수 있다.

적절하게 격리수준을 조정해서 가능한 한 많은 트랜잭션을 동시에 진행시키면서도 성능에 문제가 발생하지 않도록 해야 한다.

DefaultTransactionDefinition 의 격리수준 속성은 ISOLATION_DEFAULT 로 설정되어 있고, 이는 DataSource 의 기본 격리수준을 그대로 따른다는 뜻이다.

#### 제한시간

트랜잭션을 수행하는 제한시간 timeout 을 설정할 수 있다.

DefaultTransactionDefinition 의 제한시간 속성은 -1 로 설정되어 있고, 이는 제한시간이 없다는 뜻이다.

제한시간은 PROPAGATION_REQUIRED 와 PROPAGATION_REQUIRES_NEW 두 가지 전파속성에서만 의미가 있다.

#### 읽기전용

일기전용 read only 는 트랜잭션 내에서 데이터를 조작하는 시도를 막아줄 수 있다.

이는 성능 향상에 도움을 주기도 한다.

---

이렇게 TransactionDefinition 타입 오브젝트를 사용하면 네 가지 속성을 이용해 트랜잭션의 동작방식을 제어할 수 있다.

트랜잭션 정의를 수정하려면 TransactionDefinition 타입의 오브젝트를 DI 받아 사용하도록 하면 된다.

하지만 이 방법으로 트랜잭션 속성을 수정하면 TransactionAdvice 를 사용하는 모든 트랜잭션 속성이 한꺼번에 변경된다.

원하는 메소드만 별도의 트랜잭션 정의를 적용할 수 있도록 해보자.

### 6.6.2 트랜잭션 인터셉터와 트랜잭션 속성

메소드별로 다른 트랜잭션 정의를 적용하려면 어드바이스의 기능을 확장해야 한다.

메소드 이름 패턴에 따라 다른 트랜잭션 정의가 적용되도록 할 수 있다.

#### TransactionInterceptor

이제부터는 TransactionAdvice 를 그만 사용하고 스프링의 TransactionInterceptor 를 이용해보자.

TransactionInterceptor 는 트랜잭션 정의를 메소드 이름 패턴을 이용해서 다르게 지정할 수 있는 방법을 추가로 제공해 준다.

TransactionInterceptor 는 PlatformTransactionManager 와 Properties 타입의 두 가지 프로퍼티를 갖고 있다.

Properties 타입인 두 번째 프로퍼티는 트랜잭션 속성을 정의한 프로퍼티로 transactionAttributes 라고 한다.

트랜잭션 속성은 TransactionDefinition 의 네 가지 기본항목에 rollbackOn() 이라는 메소드를 하나 더 갖고 있는 TransactionAttribute 인터페이스로 정의된다.

rollbackOn() 메소드는 어떤 예외가 발생하면 롤백을 할 것인가를 결정하는 메소드다.

TransactionAttribute 를 이용해서 트랜잭션 부가기능의 동작방식을 모두 제어할 수 있다.

```java
public Object invoke(MethodInvocation invocation) throws Throwable {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        Object ret = invocation.proceed();
        this.transactionManager.commit(status);
        return ret;
    } catch (RuntimeException e) {
        this.transactionManager.rollback(status);
        throw e;
    }
}
```

TransactionAdvice 는 RuntimeException 이 발생하는 경우에만 트랜잭션을 롤백시키기 때문에 RuntimeException 이 아니면 트랜잭션이 제대로 처리되지 않고 메소드를 빠져나가게 된다.

일부 체크 예외는 정상적인 작업 흐름 안에서 사용될 수 있기 때문에 모든 예외에 대해 롤백시키도록 해서는 안된다.

스프링이 제공하는 TransactionInterceptor 는 기본적으로 두 가지 종류의 예외 처리 방식이 있다.

런타임 예외가 발생하면 트랜잭션은 롤백된다.

반면에 체크 예외를 던지는 경우에는 이를 일종의 비즈니스 로직에 의한 리턴 방식으로 인식해서 트랜잭션을 커밋해버린다.

스프링의 기본적인 예외처리 원칙에 따라 비즈니스 로직에 의한 예외상황에만 체크 예외를 사용하고, 그 외의 모든 복구 불가능한 순수한 예외의 경우는 런타임 예외로 포장하여 전달하는 방식을 따른다고 가정하기 때문이다.

하지만 기본 예외처리 원칙을 따르지 않는 경우가 있을 수 있다.

그래서 TransactionAttribute 는 rollbackOn() 이라는 속성을 둬서 기본 원칙과 다른 처리를 가능하게 해준다. 

특정 체크 예외의 경우에 트랜잭션을 롤백시키거나 특정 런타임 예외의 경우에 트랜잭션을 커밋시킬 수도 있다.

#### 메소드 이름 패턴을 이용한 트랜잭션 속성 지정
