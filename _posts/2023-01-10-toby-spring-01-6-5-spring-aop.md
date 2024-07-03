---
# layout: single
title: "6장 AOP - 6.5 스프링 AOP"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

## 6.5 스프링 AOP

지금까지 해온 작업은 비즈니스 로직에 반복적으로 등장하는 트랜잭션 코드를 깔끔하고 효과적으로 분리해내는 것이다.

### 6.5.1 자동 프록시 설정

부가기능을 적용하는 과정에서 있었던 문제는 대부분 해소되었다.

타깃 코드는 여전히 깔끔한 채로 남아있고, 부가기능과 타깃의 적용 메소드를 선정하는 방식도 독립적으로 작성할 수 있게 되었다.

그래도 부가기능의 적용이 필요한 타깃 오브젝트마다 거의 비슷한 내용의 ProxyFactoryBean 빈 설정정보를 추가해주는 부분이 남아있다.

target 프로퍼티를 제외하면 빈 클래스의 종류, 어드바이스, 포인트컷의 설정이 동일하다.

#### 중복 문제의 접근 방법

지금까지 다뤄봤던 반복적이고 기계적인 코드에 대한 해결책을 생각해보자.

전략 패턴과 DI 를 이용해서 템플릿과 콜백, 클라이언트로 나뉘는 방법을 통해 바뀌지 않는 부분과 바뀌는 부분을 구분해서 분리했다.

반복적인 위임 코드가 필요한 프록시 클래스 코드의 경우는 다이내믹 프록시라는 런타임 코드 자동생성 기법을 이용하였다.

JDK 의 다이내믹 프록시는 특정 인터페이스를 구현한 오브젝트에 대해서 프록시 역할을 해주는 클래스를 런타임 시 내부적으로 만들어준다.

변하지 않는 타깃으로의 위임과 부가기능 적용 여부 판단이라는 부분은 코드 생성 기법을 이용하는 다이내믹 프록시 기술에 맡기고, 변하는 부가기능 코드는 별도로 만들어서 다이내믹 프록시 생성 팩토리에 DI 로 제공하는 방법이다.

반복적인 ProxyFactortBean 설정 문제를 설정 자동등록 기법으로 해결할 수 없을까?

#### 빈 후처리기를 이용한 자동 프록시 생성기

스프링은 OCP 의 가장 중요한 요소인 유연한 확장이라는 개념을 스프링 컨테이너 자신에게도 다양한 방법으로 적용하고 있다.

관심을 가질 만한 확장 포인트는 BeanPostProcessor 인터페이스를 구현한 빈 후처리기이다.

DefaultAdvisorAutoProxyCreator 는 어드바이저를 이용한 자동 프록시 생성기다.

빈 후처리기를 스프링에 빈으로 등록해놓으면 스프링은 빈 오브젝트가 생성될 때마다 빈 후처리기에 보내서 후처리 작업을 요청한다.

이를 잘 이용하면 스프링이 생성하는 빈 오브젝트의 일부를 프록시로 포장하고, 프록시를 빈으로 대신 등록할 수 도 있다.

이것이 자동 프록시 생성 빈 후처리기이다.

스프링에 DefaultAdvisorAutoProxyCreator 을 빈으로 등록한다. 

DefaultAdvisorAutoProxyCreator 빈 후처리기는 빈이 생성되어 전달되어 오면 등록된 모든 어드바이저 내의 포인트컷을 이용해 프록시 적용 대상인지 확인한다.

프록시 적용 대상이면 내장된 프록시 생성기에게 현재 빈에 대한 프록시를 만들게 하고, 만들어진 프록시에 어드바이저를 연결한다.

그리고 원래 스프링이 전달해준 빈 오브젝트 대신 프록시 오브젝트를 되돌려준다.

#### 확장된 포인트컷

포인트컷이란 타깃 오브젝트의 메소드 중에서 어떤 메소드에 부가기능을 적용할지를 선정해주는 역할을 한다.

사실 포인트컷은 클래스 필터와 메소드 매처 두가지를 돌려주는 메소드를 갖고 있다.

지금까지 사용해온 NameMatchMethodPointcut 은 클래스 필터는 모든 클래스를 다 받아주고, 메소드 선별 기능만 가진 특별한 포인트컷이다.

Pointcut 인터페이스는 원래 선별 로직 두 가지를 가지고 있다.

```java
public interface Pointcut {
    ClassFilter getClassFilter();
    MethodMatcher getMethodMatcher();
}
```

따라서 Pointcut 의 선정 기능을 모두 적용한다면 프록시를 적용할 메소드인지 판단하고나서, 어드바이스를 적용할 메소드인지 확인하는 식으로 동작하는 것이다.

#### 포인트컷 테스트

NameMatchMethodPointcut 을 확장한 포인트컷을 만들고 테스트로 확인해보자

```java
@Test
public void pointcut() throws Exception {
    NameMatchMethodPointcut classMethodPointcut = new NameMatchMethodPointcut() {
        @Override
        public ClassFilter getClassFilter() {
            return clazz -> clazz.getSimpleName().startsWith("HelloT");
        }
    };
    classMethodPointcut.setMappedName("sayH*");

    // 테스트
    checkAdviced(new HelloTarget(), classMethodPointcut, true);

    class HelloWorld extends HelloTarget {};
    checkAdviced(new HelloWorld(), classMethodPointcut, false);

    class HelloToby extends HelloTarget {};
    checkAdviced(new HelloToby(), classMethodPointcut, true);
}
```

기존의 NameMatchMethodPointcut 을 내부 익명 클래스 방식으로 확장해서 만들었다.

getClassFilter() 를 오버라이드해서 이름이 "HelloT" 로 시작하는 클래스만 선정하도록 만들었다.

포인트컷이 클래스 필터가 동작해서 클래스를 걸러내버리면 아무리 프록시를 적용했다 해도 부가기능은 전혀 제공되지 않는다는 점을 주의하자.

### 6.5.2 DefaultAdvisorAutoProxyCreator의 적용

NameMatchMethodPointcut 을 상속해서 클래스 이름을 비교하는 ClassFilter 를 추가하도록 만들어야 한다.

```java
public class NameMatchClassMethodPointcut extends NameMatchMethodPointcut {
    public void setMappedClassName(String mappedClassName) {
        this.setClassFilter(clazz -> clazz.getName().contains(mappedClassName));
    }
    
    static class SimpleClassFilter implements ClassFilter {
        private String mappedName;
        
        public SimpleClassFilter(String mappedName) {
            this.mappedName = mappedName;
        }
        
        @Override
        public boolean matches(Class<?> clazz) {
            return clazz.getName().contains(mappedName);
        }
    }
}
```

#### 어드바이저를 이용하는 자동 프록시 생성기 등록

자동 프록시 생성기인 DefaultAdvisorAutoProxyCreator 는 등록된 빈 중에 Advisor 인터페이스를 구현한 빈을 모두 찾는다.

그리고 생성되는 모든 빈에 대해 어드바이저 포인트컷을 적용해보면서 프록시 적용 대상을 선정한다.

DefaultAdviserAutoProxyCreator 는 한줄로 빈 등록을 해준다.

```xml
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>
```

다른 빈에서 참조되거나 코드에서 빈 이름으로 조회될 필요가 없다면 id 를 등록하지 않아도 된다.

#### 포인트컷 등록

클래스 필터가 되는 포인트컷을 빈으로 등록한다.

```xml
<bean id="pointcut" class="springbook.service.NameMatchClassMethodPointcut">
    <property name="mappedClassName" value="*ServiceImpl"/>
    <property name="mappedName" value="upgrade*"/>
</bean>
```

#### 어드바이스와 어드바이저

이제는 ProxyFactoryBean 으로 등록한 빈에서처럼 trasactionAdvisor 를 명시적으로 DI 하는 빈은 존재하지 않는다.

대신 어드바이저를 이용하는 자동 프록시 생성기인 DefaultAutoProxyCreator 에 의해 자동수집되고, 프록시 대상 선정 과정에 참여하며, 자동생성된 프록시에 다이내믹하게 DI 되어 동작하는 어드바이저가 된다.

#### ProxyFactoryBean 제거와 서비스 빈의 원상복구

이제 더이상 명시적인 프록시를 등록할 필요가 없기 때문에 userService 는 다시 원래의 id 인 userService 로 돌아올 수 있다.

```xml
<bean id="userService" class="springbook.user.service.UserServiceImpl">
    <property name="userDao" ref="userDao"/>
    <property name="mailSender" ref="mailSender"/>
</bean>
```

#### 자동 프록시 생성기를 사용하는 테스트

이제 @Autowired 를 통해 컨텍스트에서 가져오는 UserService 타입 오브젝트는 트랜잭션이 적용된 프록시이어야 한다.

upgradeAllOrNothing() 테스트를 위해 강제 예외 발생용 TestUserService 클래스를 빈으로 등록할 필요가 있다.

TestUserService 의 이름을 TestUserServiceImpl 로 변경한다.

그리고 예외를 발생시킬 대상인 네 번째 사용자 아이디를 클래스에 넣어서 고정한다.

```java
static class TestUSerServiceImpl extends UserServiceImpl {
    private String id = "madnite1";
    
    protected void upgradeLevel(User user) {
        if (user.getId().equals(this.id)) throw new TestUserServiceException();
        super.upgradeLevel(user);
    }
}
```

이제 TestUserServiceImpl 을 빈으로 등록한다.

```xml
<bean id="testUserService" class="springbook.user.service.UserServiceTest$TestUserServiceImpl" parent="userService"/>
```

클래스 이름에 사용한 $ 기호는 스태틱 멤버 클래스를 지정할 때 사용하는 것이다.

parent 애트리뷰트를 사용하면 다른 빈 설정의 내용을 상속받을 수 있다.

클래스는 물론이고, 프로퍼티 설정도 모두 상속받는다.

이제 테스트코드에서 upgradeAllOrNothing() 메소드에서는 testUserService 빈을 사용하도록 수정하자

```java
public class UserServiceTest {
    @Autowired
    UserService userService;
    
    @Autowired
    UserService testUserService;
    
    @Test
    public void upgradeAllOrNothing() {
        userDao.deleteAll();
        for(User user : users) userDao.add(user);
        
        try {
            this.testUserService.upgradeLevels();
            fail("TestUserServiceException expected");
        } catch (TestUserServiceException e) {
            // TestUserService 는 업그레이드 작업 중에 예외가 발생해야 함
        }
        checkLelveUpgraded(users.get(1), false);
    }
}
```

설정과 테스트 코드가 깔끔해졌다.

#### 자동생성 프록시 확인

트랜잭션 어드바이스를 적용한 프록시 자동생성기를 빈 후처리기 메커니즘을 통해 적용했다.

첫째, 트랜잭션이 필요한 빈에 트랜잭션 부가기능이 적용되었는가이다.

upgradeAllOrNothing() 테스트를 통해 검증이 가능하다.

둘째, 아무 빈에나 트랜잭션 부가기능이 적용된 것이 아닌지 확인해야 한다.

포인트컷 빈의 클래스 이름 패턴을 변경해서 testUserService 에 트랜잭션이 적용되지 않게 해보자.

마지막으로 DefaultAdvisorAutoProxyCreator 에 의해 userService 빈이 프록시로 바꿔치기 되었다면 getBean("userService") 로 가져온 오브젝트는 TestUserService 타입이 아니라 JDK 의 Proxy 타입일 것이다.

```java
@Test
public void advisorAdutoProxyCreator() {
    assertThat(this.userService, is(java.lang.reflect.Proxy.class));
}
```

### 6.5.3 포인트컷 표현식을 이용한 포인트컷


