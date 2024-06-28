---
# layout: single
title: "6장 AOP - 6.3 다이내믹 프록시와 팩토리 빈"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

## 6.3 다이내믹 프록시와 팩토리 빈

### 6.3.1 프록시와 프록시 패턴, 데코레이터 패턴

단순히 확장성을 고려해서 한 가지 기능을 분리한다면 전형적인 전략 패턴을 사용하면 된다.

전략 패턴을 통해 트랜잭션 기능의 구현을 분리해냈지만 트랜잭션 코드는 그대로 남아있었다.

트랜잭션이라는 기능은 사용자 관리 비즈니스 로직과는 성격이 다르기 때문에 아예 그 적용 사실 자체를 밖으로 분리할 수 있었다.

그 결과, UserServiceTx 를 만들었고 UserServiceImpl 에는 트랜잭션 관련 코드가 하나도 남지 않게 되었다. 

UserServiceTx 같은 부가기능을 담은 클래스는 부가기능 외의 나머지 기능은 원래의 핵심기능을 가진 클래스로 위임해주어야 한다는 특징이 있다.

부가기능을 가진 클래스가 핵심기능을 사용하는 구조가 되는 것이다.

부가기능은 마치 자신이 핵심기능을 가진 클래스인양 꾸며서, 클라이언트가 자신을 거쳐서 핵심기능을 사용하도록 만들어야 한다.

그러기 위해서는 클라이언트는 인터페이스를 통해서만 기능을 호출해야 하고, 부가기능은 인터페이스를 구현한 뒤 자신이 그 사이에 끼어들어야 한다.

비즈니스 로직에 트랜잭션 기능을 부여해주는 것처럼 부가기능 코드에서는 핵심기능으로 요청을 위임하는 과정에서 자신이 가진 부가기능을 적용할 수 있다.

자신을 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 프록시 proxy, 즉 대리인이라고 부른다.

프록시를 통해 실제로 요청을 위임받아 처리하는 오브젝트는 타깃 target 또는 실체 real subject 라고 부른다.

따라서 프록시의 특징을 타깃과 같은 인터페이스를 구현했다는 점, 프록시가 타깃을 호출할 수 있는 위치에 있다는 점으로 볼 수 있다.

프록시는 사용 목적에 따라 두 가지로 데코레이터 패턴, 프록시 패턴으로 불린다.

#### 데코레이터 패턴

데코레이터 패턴은 타깃에 부가적인 기능을 런타임 시 다이내믹하게 부여하기 위해 프록시를 사용하는 패턴을 말한다.

여기서 다이내믹하다는 뜻은 컴파일 시점, 즉 코드상에서는 어떤 방법과 순서로 프록시와 타깃을 연결할지 정해져있지 않다는 뜻이다.

데코레이터 패턴에서는 동일한 인터페이스를 구현한 타겟과 여러 개의 프록시를 사용할 수 있다.

자바 IO 패키지의 InputStream 과 OutputStream 구현 클래스는 데코레이터 패턴이 사용된 대표적인 예이다.

인터페이스를 통한 데코레이터 정의와 런타임 시의 다이내믹한 구성 방법은 스프링의 DI 를 이용하면 아주 편리하다.

데코레이터 빈의 프로퍼티로 같은 인터페이스를 구현한 다른 데코레이터 혹은 타깃 빈을 설정하면 된다.

데코레이터 패턴은 타깃의 코드를 손대지 않고, 클라이언트가 호출하는 방법도 변경하지 않은 채로 새로운 기능을 추가할 때 유용한 방법이다.

#### 프록시 패턴

일반적으로 프록시라는 용어는 클라이언트와 타깃 사이에 대리 역할을 맡은 오브젝트를 말한다.

프록시 패턴은 프록시를 사용하는 방법 중에 타깃에 대한 접근 방법을 제어하려는 목적을 가진 경우를 말한다.

프록시 패턴의 프록시는 타깃의 기능을 확장하거나 추가하지 않고, 클라이언트가 타깃에 접근하는 방식을 변경해준다.

클라이언트에게 타깃에 대한 레퍼런스를 넘겨야 하는데, 실제 타깃 오브젝트를 만드는 대신 프록시를 넘겨주는 것이다.

그리고 프록시의 메소드를 통해 타깃을 사용하려고 하면, 그 때 프록시가 타깃 오브젝트를 생성하고 요청을 위임해주는 식이다.

각종 리모팅 기술을 이용해 다른 서버에 존재하는 오브젝트를 사용해야 한다면, 원격 오브젝트에 대한 프록시를 만들어두고, 클라이언트는 마치 로컬에 존재하는 오브젝트를 쓰는 것처럼 프록시를 사용하게 할 수 있다.

또는 타깃에 대한 접근권한을 제어하기 위해 프록시 패턴을 사용할 수 있다.

만약 수정 가능한 오브젝트가 있는데, 특정 레이어로 넘어가서는 읽기전용으로만 동작하게 강제해야 할 때, 오브젝트의 프록시를 만들어서 접근을 제어할 수 있다.

Collections 의 unmodifiableCollection() 을 통해 만들어지는 오브젝트가 전형적인 접근권한 제어용 프록시라고 볼 수 있다.

이렇듯, 프록시 패턴은 타깃은 기능 자체에는 관여하지 않으면서 접근하는 방법을 제어해주는 프록시를 이용하는 것이다.

앞으로는 타깃과 동일한 인터페이스를 구현하고 클라이언트와 타깃 사이에 존재하면서 기능의 부가 또는 접근 제어를 담당하는 오브젝트를 모두 프록시라고 부르겠다.

### 6.3.2 다이내믹 프록시

프록시는 기존 코드에 영향을 주지 않으면서 타깃의 기능을 확장하거나 접근 방법을 제어할 수 있는 유용한 방법이다.

프록시를 일일이 모든 인터페이스를 구현해서 클래스를 새로 정의하지 않고도 편리하게 만들어서 사용할 방법이 없을까?

자바의 java.lang.reflect 패키지 안에 프록시를 손쉽게 만들 수 있도록 지원해주는 클래스들이 있다.

기본적인 아이디어는 목 프레임워크와 비슷하다. 몇 가지 API 를 이용해 프록시 클래스를 정의하지 않고도 프록시처럼 동작하는 오브젝트를 다이내믹하게 생성하는 것이다.

#### 프록시의 구성과 프록시 작성의 문제점

프록시는 다음의 두 가지 기능으로 구성된다.

- 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
- 지정된 요청에 대해서는 부가기능을 수행한다.

UserServiceTx 를 살펴보면 프록시의 역할은 타깃으로 요청을 위임하는 것과 부가작업이라는 두 가지로 구분해볼 수 있다.

```java
public class UserServiceTx implements UserService {
    // ...
    public void upgradeLevels() {
        // 부가기능 수행
      TransactionStatus status = this.transactionManager
              .getTransaction(new DefaultTransactionDefinition());
        try {
            userService.upgradeLevels(); // 타깃으로 요청을 위임
            this.transactionManager.commit(status);
        } catch(RuntimeException e) {
            // 부가기능 수행
        }
    }
}
```

프록시를 만들기 어려운 이유는 아래와 같다.

- 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다.
  - 타깃 인터페이스의 메소드가 추가되거나 변경될 때마다 함께 수정해줘야 한다.
  - 부가기능이 필요없는 메소드도 프록시 클래스에 타깃으로 위임하는 코드를 만들어줘야 한다.
- 부가기능 코드가 중복될 가능성이 많다.
  - 예컨데, 여러 클래스에 트랜잭션 기능이 필요해면 해당 부가기능 코드가 여러 클래스에 중복되어 나타난다.

#### 리플렉션

다이내믹 프록시는 리플렉션 기능을 이용해서 프록시를 만들어준다.

리플렉션은 자바의 코드 자체를 추상화해서 접근하도록 만든 것이다.

자바의 모든 클래스는 그 클래스 자체의 구성정보를 담은 Class 타입의 오브젝트를 가지고 있다.

getClass() 메소드를 호출하면 클래스의 정보를 담은 Class 타입의 오브젝트를 가져올 수 있다.

클래스 오브젝트를 이용하면 클래스 코드에 대한 메타정보를 가져올 수 있고, 심지어 오브젝트의 값을 변경할 수도 있다.

리플렉션 API 중에서 메소드에 대한 정의를 담은 Method 라는 인터페이스를 이용해 메소드를 호출하는 방법을 보자.

```java
String name = "String";

// length() 를 이용한 방법
int length1 = name.length();

// Method.invoke() 를 이용한 방법
Method lengthMethod = String.class.getMethod("length");
int length2 = lengthMethod.invoke(name);
```

#### 프록시 클래스

다이내믹 프록시를 이용한 프록시를 만들어보자.

```java
// Hello 인터페이스
interface Hello {

  String sayHello(String name);

  String sayHi(String name);

  String sayThankYou(String name);

}
```

```java
// Hello 타깃 클래스
public class HelloTarget implements Hello {

  public String sayHello(String name) {
    return "Hello " + name;
  }

  public String sayHi(String name) {
    return "Hi " + name;
  }

  public String sayThankYou(String name) {
    return "Thank You " + name;
  }

}
```

```java
// 테스트
@Test
public void simpleProxy() {
  Hello hello = new HelloTarget();
  assertThat(hello.sayHello("Toby"), is("Hello Toby"));
  assertThat(hello.sayHi("Toby"), is("Hi Toby"));
  assertThat(hello.sayThankYou("Toby"), is("Thank You Toby"));
}
```

타깃인 HelloTarget 에 부가기능을 추가하는 HelloUppercase 프록시를 만들어 본다.

HelloUppercase 프록시는 Hello 인터페이스를 구현하고, Hello 타입의 타깃 오브젝트를 받아서 저장해둔다.

프록시의 구현 메소드에서는 타깃 오브젝트의 메소드를 호출한 뒤에 결과를 대문자로 바꿔주는 부가기능을 적용한다.

```java
public class HelloUppercase implements Hello {

  Hello hello;

  public HelloUppercase(Hello hello) {
    this.hello = hello;
  }

  public String sayHello(String name) {
    String helloString = hello.sayHello(name); // 위임
    return helloString.toUpperCase(); // 부가기능
  }

  public String sayHello(String name) {
    String hiString = hello.sayHi(name); // 위임
    return hiString.toUpperCase(); // 부가기능
  }

  public String sayHello(String name) {
    String thankYouString = hello.sayThankYou(name); // 위임
    return thankYouString.toUpperCase(); // 부가기능
  }

}
```

테스트 코드를 추가하고 프록시가 동작하는지 확인해보자

```java
@Test
public void proxiedHello() {
    Hello proxiedHello = new HelloUppercase(new HelloTarget());
    assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
    assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
    assertThat(proxiedHello.sayThankYou("Toby"), is("THANK YOU TOBY"));
}
```

이 프록시는 프록시 적용의 일반적인 문제점 두 가지를 모두 갖고 있다.

인터페이스의 모든 메소드를 구현해 위임하도록 코드를 만들어야 하고, 대문자로 바꿔주는 부가기능이 모든 메소드에 중복되어 나타난다.

#### 다이내믹 프록시 적용

HelloUppercase 를 다이내믹 프록시를 이용해 만들어보자.

다이내믹 프록시는 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트다.

프록시 팩토리에게 인터페이스 정보만 제공해주면 해당 인터페이스를 구현한 프록시 오브젝트를 만들어준다.

이 덕분에 프록시를 만들 때 인터페이스를 모두 구현해가면서 클래스를 정의하는 수고를 덜 수 있다.

클라이언트는 여전히 인터페이스를 통해 다이내믹 프록시 오브젝트로 부가기능을 사용할 수 있다.

다이내믹 프록시가 인터페이스 구현 클래스의 오브젝트를 만들어주지만, 필요하나 부가기능을 제공하는 코드는 직접 작성해야 한다.

부가기능은 InvocationHandler 를 구현한 오브젝트에 작성한다.

InvocationHandler 는 메소드 한 개만 가진 인터페이스이다.

```java
public Object invoke(Object proxy, Method method, Object[] args);
```

invoke() 메소드는 Method 인터페이스와 파라미터를 method 와 args 로 각각 받는다.

Hello 인터페이스를 제공하면서 프록시 팩토리에게 다이내믹 프록시를 만들어달라고 요청하면 Hello 인터페이스의 모든 메소드를 구현한 오브젝트를 생성해준다.

다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 의 invoke() 메소드로 넘기는 역할을 맡는다.

이렇게 하면 인터페이스의 모든 요청이 하나의 메소드로 집중되기 때문에 중복을 제거할 수 있다.

InvocationHandler 를 만들어보자

```java
public class UppercaseHandler implements InvocationHandler {

  Hello target;

  public UppercaseHandler(Hello target) {
    this.target = target;
  }

  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    String ret = (String) method.invoke(target, args);
    return ret.toUpperCase();
  }

}
```

다이내믹 프록시를 통해 요청이 전달되면 리플렉션 API 를 이용해 타깃 오브젝트의 메소드를 호출한다.

타깃 오브젝트의 메소드 호출이 끝났으면 프록시가 제공하려는 부가기능인 리턴 값을 대문자로 바꾸는 작업을 수행하고 결과를 리턴한다.

이제 UppercaseHandler 를 사용하고 Hello 인터페이스를 구현한 다이내믹 프록시를 만들어보자.

아래는 다이내믹 프록시를 생성하는 코드이다.

```java
Hello proxiedHello = (Hello) Proxy.newProxyInstance(
        getClass().getClassLoader(),
        new Class[] { Hello.class },
        new UppercaseHandler(new helloTarget())
);
```

첫 번째 파라미터는 다이내믹 프록시가 정의되는 클래스 로더를 제공해야 한다.

두 번째 파라미터는 다이내믹 프록시가 구현해야 할 인터페이스다.

마지막 파라미터는 부가기능과 위임 관련 코드를 담고 있는 InvocationHandler 구현 오브젝트를 제공해야 한다.

newProxyInstance() 에 의해 만들어지는 다이내믹 프록시 오브젝트는 두 번째 파라미터로 제공한 Hello 인터페이스를 구현한 오브젝트이기 때문에 Hello 타입으로 캐스팅이 가능하다.

따라서 proxiedHello 는 Hello 인터페이스로 사용이 가능하다.

#### 다이내믹 프록시의 확장

UppercaseHandler 를 이용한 다이내믹 프록시 오브젝트는 Hello 인터페이스가 바뀌더라도 전혀 손댈 게 없다.

다이내믹 프록시가 만들어 질 때 메소드가 모두 포함될 것이고, 부가기능은 invoke() 메소드에서 처리되기 때문이다.

InvocationHandler 의 구현을 변경하여 타깃의 종류에 관계없이 적용이 가능하도록 하고, 리턴 타입이 문자열인 경우에만 대문자로 바꿔주도록 할 수 있다.

```java
public class UppercaseHandler implements InvocationHandler {

    Object target;

    private UppercaseHandler(Object target) {
        this.target = target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object ret = method.invoke(target, args);
        if (ret instanceof String) {
            return ((String) ret).toUpperCase();
        } else {
            return ret;
        }
    }

}
```

리턴 타입뿐 아니라 메소드 이름을 조건으로 "say" 로 시작하는 메소드일 경우에 대문자로 바꿔주도록 할 수 있다.

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    Object ret = method.invoke(target, args);
    if (ret instanceof String && method.getName().startWith("say")) {
        return ((String) ret).toUpperCase();
    } else {
        return ret;
    }
}
```

### 6.3.3 다이내믹 프록시를 이용한 트랜잭션 부가기능

트랜잭션 기능을 부가해주는 InvocationHandler 를 정의하고 다이내믹 프록시와 연동하여 UserServiceTx 를 다이내믹 프록시 방식으로 변경해보자.

#### 트랜잭션 InvocationHandler

```java
public class TransactionHandler implements InvocationHandler {

    private Object target;

    private PlatformTransactionManager transactionManager;

    private String pattern; // 트랜잭션 적용 대상 메소드의 이름 패턴

    public void setTargret(Object target) {
        this.target = target;
    }

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    public void setPattern(String pattern) {
        this.pattern = pattern;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (method.getName().startWith(pattern)) { // 적용할 메서드를 선별
            return invokeInTransaction(method, args);
        } else {
            return method.invoke(target, args);
        }
    }

    private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            Object ret = method.invoke(target, args);
            this.transactionManager.commit(status);
            return ret;
        } catch (InvocationTargetException e) {
            this.transactionManager.rollback(status);
            throw e.getTargetException(); // 중첩되어 있는 예외를 반환
        }
    }

}
```

#### TransactionHandler 와 다이내믹 프록시를 이용하는 테스트

TrasactionHandler 가 UserServiceTx 를 대신할 수 있는지 테스트케이스를 변경한다.

```java
@Test
public void upgradeAllOrNothing() throws Exception {
  ...
  TransactionHandler txHandler = new TransactionHandler();
  txHandler.setTarget(testUserService);
  txHandler.setTransactionManager(transactionManager);
  txHandler.setPattern("upgradeLevels");
  UserService txUserService = (UserService) Proxy.newProxyInstance(
          getClass().getClassLoader(),
          new Class[] { UserService.class },
          txHandler
  );
}
```

다이내믹 프록시로 만든 txUserService 로도 테스트는 통과한다.

### 6.3.4 다이내믹 프록시를 위한 팩토리 빈

TransactionHandler 와 다이내믹 프록시를 스프링의 DI 를 통해 사용할 수 있도록 만들어야 한다.

스프링이 빈은 지정된 클래스 이름을 가지고 리플렉션을 이용해서 해당 클래스의 오브젝트를 만든다.

Class 의 newInstance() 메소드는 해당 클래스의 기본 생성자를 호출하여 오브젝트를 반환하는 리플렉션 API 이다.

```java
Date new (Date) Class.forName("java.util.Date").newInstance();
```

하지만 다이내믹 프록시 오브젝트는 이런 식으로 프록시 오브젝트가 생성되지 않는다.

#### 팩토리 빈

스프링은 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에도 빈을 만들 수 있는 다른 방법을 제공한다.

대표적으로 팩토리 빈을 이용한 빈 생성 방법이다.

팩토리 빈이란 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈이다.

스프링의 FactoryBean 의 인터페이스를 구현해서 팩토리 빈을 만들 수 있다.

FactoryBean 인터페이스는 세 가지 메소드로 구성되어 있다.

```java
public interface FactoryBean<T> {
    
    T getObject() throws Exception; // 빈 오브젝트를 생성하여 반환 

    Class<? extends T> getObjectType(); // 생성되는 오브젝트의 타입을 반환

    boolean isSingleton(); // getObject 가 반환하는 오브젝트가 항상 싱글톤 오브젝트인지 알려준다.

}
```

FactoryBean 인터페이스를 구현한 클래스를 스프링 빈으로 등록하면 팩토리 빈으로 동작한다.

생성자를 외부로 제공하지 않고 오브젝트를 반환하는 스태틱 메소드만 가지고 있는 클래스를 정의해보자.

```java
public class Message {

  String text;

  private Message(String text) {
    this.text = text;
  }

  public String getText() {
    return text;
  }

  public static Message newMessage(String text) {
    return new MEssage(text);
  }

}
```

물론 스프링은 private 생성자를 가진 클래스도 빈으로 등록해주면 리플렉션을 이용해 오브젝트로 만들어준다.

하지만 생성자를 private 으로 만들었다는 것은 스태틱 메소드를 통해 오브젝트가 만들어져야 하는 이유가 있기 때문이다.

따라서 이 Message 클래스는 직접 스프링 빈으로 등록해서 사용해서는 안된다.

이제 Message 클래스의 오브젝트를 생성해주는 팩토리 빈 클래스를 만들어보자.

```java
public class MessageFactoryBean implements FactoryBean<Message> {

  String text;

  public void setText(String text) {
    this.text = text; // 오브젝트를 생성할 때 필요한 정보를 팩토리 빈의 프로퍼티로 받을 수 있다.
  }

  public Message getObject() throws Exception {
    return Message.newMessage(this.text); // 오브젝트 생성, 팩토리 빈이 받아둔 정보를 사용한다.
  }

  public Class<? extends Message> getObjectType() {
    return Message.class;
  }

  public boolean isSingleton() {
    return false; // getObject 할 때마다 새로운 오브젝트를 만드므로 false
  }

}
```

스프링은 FactoryBean 인터페이스를 구현한 클래스가 빈의 클래스로 지정되면, 오브젝트의 팩토리 빈 클래스의 오브젝트의 getObject() 메소드를 이용해 오브젝트를 가져와 이를 빈 오브젝트로 사용한다.

#### 팩토리 빈의 설정 방법

팩토리 빈의 설정방법은 일반 빈과 다르지 않다.

```xml
<bean id="message" class="springbook.learningtest.spring.factorybean.MessageFactoryBean">
  <property name="text" value="Factory Bean" />
</bean>
```

다만 message 빈 오브젝트의 타입이 Message 타입이라는 점이 다르다.

Message 빈의 타입은 MessageFactoryBean 의 getObjectType() 메소드가 돌려주는 타입으로 결정된다.

그리고 getObjcet() 메소드가 생성해주는 오브젝트가 message 빈의 오브젝트가 된다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class FactoryBeanTest {
    @Autowired 
    ApplicationContext context;
    
    @Test 
    public void getMessageFromFactoryBean() {
        Object message = context.getBean("message");
        assertThat(message, is(Message.class)); // FactoryBean 이 아니라 FactoryBean 이 생성하는 오브젝트를 가져온다.
        assertThat(((Message) message).getText(), is("Factory Bean")); // FactoryBean 이 생성한 오브젝트의 동작을 확인한다.
    }
}
```



