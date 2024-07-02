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


