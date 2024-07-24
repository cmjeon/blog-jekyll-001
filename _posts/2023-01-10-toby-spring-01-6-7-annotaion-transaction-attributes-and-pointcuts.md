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




