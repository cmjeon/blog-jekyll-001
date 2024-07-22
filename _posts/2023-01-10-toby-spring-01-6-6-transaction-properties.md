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

Properties 타입의 transactionAttributes 프로퍼티는 메소드 이름 패턴을 이용해서 트랜잭션 속성을 지정할 수 있다.

```
PROPAGATION_NAME, ISOLATION_NAME, readOnly, timeout_NNNN, -exception1, +Exception2
```

- PROPAGATION_NAME : 트랜잭션 전파 방식. 필수항목
- ISOLATION_NAME : 격리수준. 생략 시 디폴트 격리 수준
- readOnly : 읽기전용 여부. 생략 시 false
- timeout_NNNN : 제한시간. timeout_초단위시간. 생략 시 제한시간 없음
- -exception1 : 체크 예외 중에서 롤백 대상으로 추가할 항목. 1개 이상 등록가능
- +Exception2 : 런타임 예외 중에서 롤백 대상에서 제외할 항목. 1개 이상 등록가능

속성을 하나의 문자열로 표현하게 만든 이유는 트랜잭션 속성을 메소드 패턴에 따라 여러 개를 지정해줘야 하는데, 일일이 중첩된 태그와 프로퍼티로 설정하게 만들면 번거롭기 때문이다.

```xml

<bean id="transactionAdvice"
      class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <prop key="get*">PROPAGATION_REQUIRED,readOnly,timeout_30</prop>
            <prop key="upgrade*">PROPAGATION_REQUIRED_NEW,ISOLATION_SERIALIZABLE</prop>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

트랜잭션 속성 중 readyOnly 나 timeout 등은 트랜잭션이 처음 시작될 때가 아니면 적용되지 않는다.

예컨데 get 으로 시작하는 메소드에서 트랜잭션이 시작되면 일기전용 제한시간이 적용되지만 그 외의 경우에는 진행 중인 트랜잭션의 속성을 따른다.

가끔 메소드 이름이 하나 이상의 패턴과 일치하는 경우가 있다. 이 때는 메소드 이름 패턴 중에서 가장 정확히 일치하는 것이 적용된다.

#### tx 네임스페이스를 이용한 설정 방법

TransactionInterceptor 타입의 어드바이스 빈과 TransactionAttribute 타입이 속성정보도 tx 스키마의 전용 태그를 이용해 정의할 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...
        xmlns:tx="http://springframework.org/schema/tx"
        xsi:schemaLocation="...
                            http://springframework.org/schema/tx
                            http://springframework.org/schema/tx/spring-tx-2.5.xsd">
     <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
          <tx:attributes>
                <tx:method name="get*" propagation="REQUIRED" read-only="true" timeout="30"/>
                <tx:method name="upgrade*" propagation="REQUIRED_NEW" isolation="SERIALIZABLE"/>
                <tx:method name="*" propagation="REQUIRED"/>
          </tx:attributes>
     </tx:advice>
</beans>
```

`<bean>` 태그로 직접 등록하는 것에 비해 장점이 많으므로 tx 스키마의 태그를 이용해 어드바이스를 등록하도록 권장한다.

### 6.6.3 포인트컷과 트랜잭션 속성의 적용 전략

포인트컷 표현식과 트랜잭션 속성을 정의할 때 따르면 좋은 몇 가지 전략을 생각해보자.

#### 트랜잭션 포인트컷 표현식은 타입 패턴이나 빈 이름을 이용한다

쓰기 작업이 없는 단순한 조회 작업만 하는 메소드에도 모두 트랜잭션을 적용하는게 좋다.  

조회의 경우에는 읽기전용으로 트랜잭션 속성을 설정해두면 그만큼 성능의 향상을 가져올 수 있다.   

또, 복잡한 조회의 경우는 제한시간을 지정해줄 수도 있고, 격리수준에 따라 조회도 반드시 트랜잭션 안에서 진행해야 할 필요가 발생하기도 한다.  

트랜잭션용 포인트컷 표현식에는 메소드나 파라미터, 예외에 대한 패턴은 정의하지 않는게 바람직하다.  

트랜잭션의 경계로 삼을 클래스들이 선정됐다면, 그 클래스들이 모여있는 패키지를 통째로 선택하거나 클래스 이름에서 일정한 패턴을 찾아서 표현식으로 만들면 된다.

가능하면 클래스보다 인터페이스 타입을 기중느로 타입 패턴을 적용하는 것이 좋다.

인터페이스가 클래스에 비해 변경 빈도가 적고 일정한 패턴을 유지하기 쉽게 때문이다.  

빈 이름을 기준으로 선정하기 때문에 클래스나 인터페이스 이름에 일정한 규칙을 만들기 어려운 경우, execution() 표현식 대신 bean() 표현식 사용을 고려해볼 수 있다.  

#### 공통된 메소드 이름 규칙을 통해 최소한의 트랜잭션 오드바이스와 속성을 정의한다

기준이 되는 몇 가지 트랜잭션 속성을 정의하고 그에 따라 적절한 메소드 명명 규칙을 만들어두면 하나의 어드바이스만으로 애플리케이션의 모든 서비스 빈에 트랜잭션 속성을 지정할 수 있다.  

일단 트랜잭션 속성의 종류와 메시지 패턴이 결정되지 않았으면 가장 단순한 티폴드 속성으로부터 출발하면 된다.  

개발이 진행됨에 따라 단계적으로 속성을 추가해준다.

```xml

<tx:advice id="transactionAdvice">
    <tx:attributes>
        <tx:method name="*" />
    </tx:attributes>
</tx:advice>
```

일반화하기에는 적당하지 않은 특별한 트랜잭션 속성이 필요한 타깃 오브젝트에 대해서는 별도이 어드바이스와 포인트컷 표현식을 사용하는 편이 좋다.

#### 프록시 방식 AOP는 같은 타깃 오브젝트 내의 메소드를 호출할 때는 적용되지 않는다


주의사항이다.  

프록시 방식의 AOP 에서는 프록시를 통한 부가기능이 적용은 클랑이ㅓㄴ트로부터 호출이 일어날 때만 적용된다.  

일단 타깃 오브젝트 내로 들어와서 타깃 오브젝트의 다른 메소드를 호출하는 경우에는 프록시를 거치지 않고 직접 타깃의 메소드가 호출된다.  

이 경우, update() 메소드의 트랜잭션 프록시를 통하지 않으므로 해당 트랜잭션 속성이 적용되지 않는다.  

이렇게 같은 타깃 오브젝트 안에서 메소드 호출이 일어나는 경우에는 프록시 AOP 를 통해 부여해준 부가기능이 적용되지 않는다는 점을 주의해야 한다.  

타깃 안에서의 메소드 호출 시 프록시가 적용되지 않는 문제를 해결할 수 있는 두 가지 방법이 있다.  

스프링 API 를 이용해 프록시 오브젝트에 대한 레퍼런스를 가져온 뒤에 같은 오브젝트의 메소드 호출도 프록시를 이용하도록 강제하는 방법이다.  

하지만 별로 추천되지 않는다.  

다른 방법은 AspectJ 와 같은 타깃의 바이트코드를 직접 조작하는 방식의 AOP 기술을 적용하는 것이다.  

### 6.6.4 트랜잭션 속성 적용



