---
# layout: single
title: "6장 AOP - 6.7 애노테이션 트랜잭션 속성과 포인트컷"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

## 6.7 애노테이션 트랜잭션 속성과 포인트컷

세밀한 트랜잭션 속성의 제어가 필요한 경우를 위해 스프링이 제공하는 다른 방법이 있다.

직접 타깃에 트랜잭션 속성정보를 가진 애노테이션을 지정하는 방법이다.

### 6.7.1 트랜잭션 애노테이션

타깃에 부여할 수 있는 트랜잭션 애노테이션은 다음과 같이 정의되어 있다.

#### @Transactional

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    String value() default "";
    Propagation propagation() default Propagation.REQUIRED;
    Isolation isolation() default Isolation.DEFAULT;
    int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
    boolean readOnly() default false;
    Class<? extends Throwable>[] rollbackFor() default {};
    String[] rollbackForClassName() default {};
    Class<? extends Throwable>[] noRollbackFor() default {};
    String[] noRollbackForClassName() default {};
}
```

- @Target({ElementType.METHOD, ElementType.TYPE})
  - 애노테이션 사용이 가능한 대상 : 메소드와 타입(클래스, 인터페이스)에 사용가능
- @Retention(RetentionPolicy.RUNTIME)
  - 런타임까지 애노테이션 정보를 유지 : 런타임 때 애노테이션 정보를 리플렉션으로 얻을 수 있음
- @Inherited
  - 상속이 가능한 애노테이션 : 클래스에 애노테이션을 부여하면 하위 클래스에도 애노테이션이 적용됨
- @Documented
  - javadoc에 문서화될 수 있음

스프링은 @Transactional 애노테이션이 부여된 모든 오브젝트를 자동으로 타깃 오브젝트로 인식한다.

이 때 사용되는 포인트컷은 TransactionAttributeSourcePointcut 이다.

TransactionAttributeSourcePointcut 은 @Transactional 이 부여된 빈 오브젝트를 포인트컷의 선정 결과로 돌려준다.

따라서 @Transactional 은 기본적으로 트랜잭션 속성을 정의하지만, 포인트컷의 자동등록에도 사용된다.

#### 트랜잭션 속성을 이용하는 포인트컷

TransactionInterceptor 는 @Transactional 애노테이션의 엘리먼트에서 트랜잭션 속성을 가져오는 AnnotationTransactionAttributeSource 를 사용한다.

@Transactional 은 메소드마다 다르게 설정할 수 있기 때문에 매우 유연한 트랜잭션 속성 설정이 가능해진다.

동시에 @Transactional 로 트랜잭션 속성이 부여된 오브젝트라면 포인트컷의 선정 대상이기 때문에 포인트컷도 @Transactional 을 통한 트랜잭션 속성정보를 참조하도록 만든다.

이렇게 하면 포인트컷과 트랜잭션 속성을 애노테이션 하나로 지정할 수 있다.

트랜잭션 부가기능 적용 단위는 메소드이다.

메소드마다 @Transactional 을 부여하면 유연한 속성 제어는 가능하지만 코드는 지저분해지고, 무엇보다 동일한 속성 정보를 가진 애노테이션을 반복적으로 메소드마다 부여해주는 바람직하지 못한 결과를 가져올 수 있다.

#### 대체 정책

스프링은 @Transactional 을 적용할 때 4단계의 대체 fallback 정책을 이용하게 해준다.

타깃 메소드, 타깃 클래스, 선언 메소드, 선언 타입(클래스, 인터페이스)의 순서에 따라 @Transactional 이 적용됐는지 차례로 확인하고, 가장 먼저 발견되는 속성정보를 사용하게 하는 방법이다.

다시 말하면 아래의 [1] ~ [6] 에 @Transactional 이 선언되었을 때 우선순위는 다음과 같다

```java
// [1]
public interface Service {
    // [2]
    void method1();
    // [3]
    void method2();
}

// [4]
public class ServiceImpl implements Service {
    // [5]
    public void method1() {
    }

    // [6]
    public void method2() {
    }
}
```

1. [5], [6]
2. [4]
3. [2], [3]
4. [1]

이런 대체 정책을 활용하면 애노테이션 자체는 최소한으로 사용하면서도 세밀한 제어가 가능하다는 장점을 살릴 수 있다.

타깃 클래스에 @Transactional 을 두는 방법을 권장한다.

기본적으로는 @Transactional 적용 대상은 클라이언트가 사용하는 인터페이스와 그 메소드이므로, 타깃 클래스보다는 인터페이스에 두는 게 바람직하지만

프록시 방식의 AOP 가 아닌 방식으로 트랜잭션 부가기능을 적용하면 인터페이스에 정의한 @Transactional 은 무시될 수 있다는 것을 염두에 두어야 한다.

따라서 비 프록시 방식 AOP 의 동작원리를 이해하여 @Transactional 적용 대상을 적절하게 변경해줄 수 있는 확신이 없다면, 마음 편하게 타깃 클래스와 메소드에 적용해두는 편이 낫다.

인터페이스에 @Transactional 을 두면 구현 클래스에 관계없이 트랜잭션 속성을 유지할 수 있다는 장점이 있다.

#### 트랜잭션 애노테이션 사용을 위한 설정

@Transactional 을 이용한 트랜잭션 속성을 사용하는데 필요한 설정은 매우 간단하다.

```xml
<transaction:annotation-driven />
```

### 6.7.2 트랜잭션 애노테이션 적용

@Transactional 을 이용한 트랜잭션 설정은 직관적이로 간단해서 많이 사용된다.

때문에 무분별한 사용이나 누락의 위험이 있다. 

또한 트랜잭션이 적용되지 않았다는 사실을 파악하기 쉽지 않기 때문에, @Transactional 적용에 대해서는 별도의 코드리뷰를 거칠 필요가 있다.

애노테이션을 이용할 때는 많이 사용되는 한 가지를 타입 레벨에 공통 속성으로 지정해주고, 나머지 속성은 개별 메소드에 적용해야 한다.

@Transactional 애노테이션은 UserService 인터페이스에 적용하여 UserServiceImpl 과 TestUserService 에도 적용되도록 하겠다.

```java
@Transactional
public interface UserService {
    void add(User user);
    void upgradeLevels();
    void update(User user);
    void upgradeLevels();
    
    @Transactional(readOnly = true)
    User get(String id);
    @Transactional(readOnly = true)
    List<User> getAll();
}
```

UserService 에는 get 으로 시작하지 않는 메소드가 더 많으므로 인터페이스 레벨에 디폴트 속성을 부여하고, get 으로 시작하는 메소드에는 읽기전용 속성을 부여한다.

만약 타깃 클래스에 @Transactional 을 부여하면 어떻게 될까?

위의 대체 정책에 따라 interface 의 메소드에 선언한 @Transactional(readOnly = true) 는 무시되고, 타깃 클래스의 @Transactional 이 적용된다.

따라서 읽기전용 속성을 검증하는 readOnlyTransactional() 테스트는 실패하게 된다.

굳이 완전한 테스트로 만들어두지 않더라도 이런 식으로 스프링의 기능을 검증해보는 것은 좋은 습관이다.

때로는 어설프게 이해하고 넘어갈 수 있는 지식을 바로잡아주는 데 도움이 되기 때문이다.

