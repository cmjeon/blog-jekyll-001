---
# layout: single
title: "6장 AOP - 6.4 스프링의 프록시 팩토리 빈"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

## 6.4 스프링의 프록시 팩토리 빈

### 6.4.1 ProxyFactoryBean

자바에는 JDK 에서 제공해주는 다이내믹 프록시 외에도 프록시를 만들 수 있는 다양한 기술이 존재한다.

따라서 스프링은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어를 제공한다.

스프링의 ProxyFactoryBean 은 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈이다.

ProxyFactoryBean 이 생성하는 프록시에서 사용할 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다.

MethodInterceptor 는 InvocationHandler 와 비슷하지만 ProxyFactoryBean 으로부터 타깃 오브젝트에 대한 정보까지 함께 제공받는다는 점이 다르다.

이 차이로 인해 MethodInterceptor 는 타깃 오브젝트에 상관없이 독립적으로 만들어져서 싱글톤 빈으로 등록이 간으하고 여러 프록시에 함께 사용할 수 있다.

스프링의 ProxyFactoryBean 을 이용하는 테스트를 만들어보자

```java
public class DynamicProxyTest {
    @Test
    public void DynamicProxyTest() {
        Hello proxiedHello = (Hello) Proxy.newProxyInstance(
                getClass().getClassLoader(),
                new Class[] {Hello.class},
                new UppercaseHandler(new HelloTarget())
        );
        // ...
    }
    
    @Test
    public void proxyFactoryBean() {
        ProxyFactoryBean pfBean = new ProxyFactoryBean();
        pfBean.setTarget(new HelloTarget());
        pfBean.addAdvice(new UppercaseAdvice());
        
        Hello proxiedHello = (Hello) pfBean.getObject();
        
        assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
        assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
        assertThat(proxiedHello.sayThankYou("Toby"), is("THANK YOU TOBY"));
    }
    
    static class UppercaseAdvice implements MethodInterceptor {
        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            String ret = (String) invocation.proceed();
            return ret.toUpperCase();
        }
    }
    
    static interface Hello {
        String sayHello(String name);
        String sayHi(String name);
        String sayThankYou(String name);
    }
    
    static class HelloTarget implements Hello {
        @Override
        public String sayHello(String name) {
            return "Hello " + name;
        }
        
        @Override
        public String sayHi(String name) {
            return "Hi " + name;
        }
        
        @Override
        public String sayThankYou(String name) {
            return "Thank You " + name;
        }
    }
}
```

#### 어드바이스: 타깃이 필요 없는 순수한 부가기능

MethodInterceptor 는 메소드 정보와 함께 타깃 오브젝트가 담긴 MethodInvocation 오브젝트가 전달된다.

MethodInvocation 은 일종의 콜백 오브젝트로,  proceed() 메소드를 실행하면 타깃 오브젝트의 메소드를 내부적으로 실행해주는 기능이 있다.

그래서 MethodInvocation 의 구현 클래스는 일종의 템플릿처럼 동작할 수 있다.

ProxyFactoryBean 에는 addAdvice() 메소드를 사용하여 여러 개의 MethodInterceptor 를 추가할 수 있다.

ProxyFactoryBean 하나만으로 여러 개이 부가기능을 제공해주는 프록시를 만들 수 있다는 뜻이다.

따라서 앞에서 살펴봤던 프록시 팩토리 빈의 단점 중 하나인 새로운 부가기능을 추가할 때마다 프록시와 프록시 팩토리 빈을 추가해주는 문제를 해결할 수 있다.

스프링은 부가기능을 추가하는 여러 가지 다양한 방법을 제공하고 있다.

스프링에서는 MethodInterceptor 처럼 타깃 오브젝트에 적용할 부가기능을 담은 오브젝트를 어드바이스 advice 라고 부른다.

ProxyFactoryBean 은 어떻게 인터페이스 타입을 제공받지도 않고 Hello 인터페이스를 구현한 프록시를 만들어낼 수 있을까?

ProxyFactoryBean 은 인터페이스 자동검출 기능을 사용해 타깃 오브젝트가 구현하고 있는 인터페이스 정보를 알아낼 수 있기 때문이다.

물론 setInterfaces() 메소드로 인터페이스를 지정해 줄 수도 있다.

어드바이스는 타깃 오브젝트에 종속되지 않는 순수한 부가기능을 담은 오브젝트라는 사실을 잘 기억해두자

#### 포인트컷: 부가기능 적용 대상 메소드 선정 방법

기존의 InvocationHandler 를 구현했을 때 부가기능을 적용하는 것 외에 부가기능의 적용 대상 메소드를 선정하는 일이 더 있었다.

TxProxyFactoryBean 이 pattern 이라는 문자열을 DI 받아서 TransactionHandler 에게 전달해줘서 요청이 들어오는 메소드의 이름과 패턴을 비교해서 부가기능 적용대상 여부를 판단했다.

스프링의 ProxyFactoryBean 과 MethodInterceptor 를 사용하는 방식에서는 어떻게 메소드 선정을 할 수 있을까?

MethodInterceptor 오브젝트는 여러 프록시가 공유해서 사용할 수 있도록 타깃 정보를 가지고 있지 않아서 메소드 선정기능을 넣을 수 없다.

대신 프록시에 부가기능 선정기능을 넣을 수 있다.

물론 프록시의 핵심 가치는 타깃을 대신해서 클라이언트의 요청을 받아 처리하는 오브젝트로써의 존재 자체이므로, 메소드를 선별하는 기능은 프록시로부터 다시 분리하는 편이 낫다.

스프링의 ProxyFactoryBean 방식은 부가기능 advice 와 메소드 선정 알고리즘 pointcut 을 활용하는 유연한 구조를 제공한다.

스프링은 부가기능을 제공하는 오브젝트를 어드바이스라고 부르고, 메소드 선정 알고리즘을 담은 오브젝트를 포인트컷이라고 부른다.

어드바이스와 포인트컷은 모두 프록시에 DI 로 주입되어 사용된다.

어드바이스와 포인트컷 모두 여러 프록시에서 공유가 가능하도록 만들어지기 때문에 스프링에서 싱글톤 빈으로 등록이 가능하다.

프록시는 클라이언트로부터 요청을 받으면 먼저 포인트컷ㅇ게 부가기능을 부여할 메소드인지 확인해달라고 요청한다.

포인트컷이 부가기능을 적용할 대상 메소드인지 확인해주면 MethodInterceptor 타입의 어드바이스를 호출한다.

어드바이스가 부가기능을 부여하는 중에 타깃 메소드의 호출이 필요하면 프록시로부터 전달받은 MethodInvocation 타입 콜백 오브젝트의 proceed() 메소드를 호출한다.

실제 위임 대상인 타깃 오브젝트의 레퍼런스를 가지고 있고, 이를 이용해 타깃 메소드를 직접 호출하는 것은 프록시가 메소드 호출에 따라 만드는 Invocation 콜백의 역할이다.

재사용이 가능한 기능을 만들어두고 바뀌는 부분만 외부에서 주입해서 사용하는 것은 전형적인 템플릿/콜백 구조이다.

어드바이스가 템플릿이 되고, 타깃을 호출하는 기능을 갖고 있는 MethodInvocation 오브젝트가 콜백이 되는 것이다.

프록시로부터 어드바이스와 포인트컷을 독립시키고 DI 를 사용할 수 있게 한 것은 전형적인 전략 패턴 구조이다.

MethodIntercepter 로 어드바이스와 메소드를 선정하는 포인트컷까지 적용되는 학습 테스트를 만들어보자.

```java
@Test
public void pointcutAdvisor() {
    ProxyFactoryBean pfBean = new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());
    
    NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
    pointcut.setMappedName("sayH*");
    
    pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UppercaseAdvice()));
    Hello proxiedHello = (Hello) pfBean.getObject();
    
    assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
    assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
    assertThat(proxiedHello.sayThankYou("Toby"), is("Thank You Toby"));
}
```

포인트컷이 없을 때는 addAdvice() 메소드로 어드바이스만 등록하면 되었다.

하지만 포인트컷을 함께 등록할 때는 어드바이스와 포인트컷을 Advisor 타입으로 조합하여 addAdvisor() 메소드로 등록해야 한다.

굳이 Adviser 타입으로 묶어서 등록하는 이유는 ProxyFactoryBean 에는 여러 개의 어드바이스와 포인트컷이 추가될 수 있기 때문이다.

다시 말해 어드바이스와 포인트컷을 따로 등록하면 어떤 어드바이스에 어떤 포인트컷을 적용할지 알 수 없어지기 때문이다.

이렇게 어드바이스와 포인트컷을 묶은 오브젝트를 인터페이스 이름을 따서 어드바이저라고 부른다.

> 어드바이저 = 포인트컷(메소드 선정 알고리즘) + 어드바이스(부가기능)

참고로 앞선 테스트에서 사용된 NameMatchMethodPointcut 은 메소드 이름을 비교해서 부가기능을 적용할 메소드를 선정하는 포인트컷이다.

### 6.4.2 ProxyFactoryBean 적용

JDK 다이내믹 프록시로 만들었던 TxProxyFactoryBean 을 스프링 ProxyFactoryBean 을 이용하도록 변경해보자

#### TransactionAdvice

부가기능을 담당하는 어드바이스는 MethodInterceptor 인터페이스를 구현한 TransactionAdvice 로 만든다.

```java
public class TransactionAdvice implements MethodInterceptor {
    PlatformTransactionManager transactionManager;
    
    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }
    
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            Object ret = invocation.proceed(); // 콜백 오브젝트의 proceed() 메소드를 호출하면 타깃 오브젝트의 메소드를 호출해준다.
            this.transactionManager.commit(status);
            return ret;
        } catch (RuntimeException e) {
            this.transactionManager.rollback(status);
            throw e;
        }
    }
}
```

#### 스프링 XML 설정파일

스프링 XML 설정을 변경하자.

어드바이스 빈을 등록한다.

```xml
<bean id="transactionAdvice" class="springbook.user.service.TransactionAdvice">
    <property name="transactionManager" ref="transactionManager"/>
</bean>
```

포인트컷 빈을 등록한다.

스프링이 제공하는 포인트컷 클래스를 사용하면 되기 때문에 빈 설정만 해주면 된다.

```xml
<bean id="transactionPointcut" class="org.springframework.aop.support.NameMatchMethodPointcut">
    <property name="mappedName" value="upgrade*"/>
</bean>
```

어드바이스와 포인트컷을 조합한 어드바이저 빈을 등록한다.

```xml
<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
    <property name="pointcut" ref="transactionPointcut"/>
    <property name="advice" ref="transactionAdvice"/>
</bean>
```

마지막으로 ProxyFactoryBean 빈을 등록한다.

프로퍼티에 타깃 빈과 어드바이저 빈을 지정해준다.

```xml
<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="userServiceTarget"/>
    <property name="proxyInterfaces" value="springbook.user.service.UserService"/>
    <property name="interceptorNames">
        <list>
            <value>transactionAdvisor</value>
        </list>
    </property>
</bean>
```

어드바이저는 interceptorNames 라는 프로퍼티에 넣어준다.

#### 테스트

테스트 코드도 변경해준다.

```java
@Test
@DirtiesContext
public void upgradeAllOrNothing() {
    TestUserService testUserService = new TestUserService(users.get(3).getId());
    testUserService.setUserDao(this.userDao);
    testUserService.setTransactionManager(transactionManager);
    testUserService.setMailSender(mailSender);
    
    ProxyFactoryBean txProxyFactoryBean = context.getBean("&userService", ProxyFactoryBean.class);
    txProxyFactoryBean.setTarget(testUserService);
    UserService txUserService = (UserService) txProxyFactoryBean.getObject();
    // z...
}
```

ProxyFactoryBean 은 스프링의 DI 와 템플릿/콜백 패턴, 서비스 추상화 등의 기법이 모두 적용된 것이다.

이제 새로운 비즈니스 로직을 담은 서비스 클래스가 만들어져도 이미 만들어둔 TransactionAdvice 를 그대로 재사용할 수 있다.

포인트컷이 필요하면 이름 패턴만 지정해서 ProxyFactoryBean 에 등록해주면 된다.

