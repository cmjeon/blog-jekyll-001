---
layout: single
title: "1장 오브젝트와 의존관계 - 1.3 DAO 의 확장"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

# 1장 오브젝트와 의존관계

## 1.3 DAO 의 확장

관심사에 따라 분리한 오브젝트들은 제각기 독특한 변화의 특징을 가진다.

변화의 특징이 있다는 것은 변화의 이유와 시기 등이 다르다는 뜻이다.

상속은 여러가지 단점이 많이 때문에 독립적인 클래스로 분리하는 것을 고민해야 한다.

### 1.3.1 클래스의 분리

DB 커넥션을 생성하는 기능을 가진 SimpleConnectionMaker 클래스를 만든다.

UserDao 는 new 키워드를 사용해 SimpleConnectionMaker 클래스의 객체를 만들고 add(), get() 메소드에서 필요할 때 가져다 쓰면 된다.

```java
public abstract class UserDao {

  private SimpleConnectionMaker simpleConnectionMaker;

  public UserDao() {
    this.simpleConnectionMaker = new SimpleConnectionMaker();
  }

  public void add(User user) throws ClassNotFoundException, SQLException {
    Connection c = this.simpleConnectionMaker.makeNewConnection();
    // ...
  }

  public User get(String id) throws ClassNotFoundException, SQLException {
    Connection c = this.simpleConnectionMaker.makeNewConnection();
    // ...
  }

  // ...

}
```

SimpleConnectionMaker 클래스는 아래와 같이 구현한다.

```java
public class SimpleConnectionMaker {

  public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
    return c;
  }

}
```

성격이 다른 코드를 분리한 것은 좋은데, 이렇게 되면 이제는 DB 커넥션 기능의 확장이 어렵다.

더이상 NUserDao, DUserDao 같이 다른 getConnection() 메소드의 기능을 구현할 수가 없다.

왜냐하면 UserDao 가 SimpleConnectionMaker 클래스에 종속되어 있기 때문이다.

```java
simpleConnectionMaker = new SimpleConnectionMaker();
```

N 사와 D 사의 getConnection 을 쓰기 위해서는 UserDao 의 수정이 반드시 필요해졌다.

클래스를 분리한 경우에도 자유로운 확장이 가능하게 하려면 두 가지 문제를 해결해야 한다.

첫번째 문제는 SimpleConnectionMaker 의 메소드가 문제다. 

우리는 makeNewConnection() 메소드를 사용했는데, D 사의 DB 커넥션 제공 클래스가 openConnection() 메소드를 사용하면 UserDao 내의 커넥션을 가져오는 코드를 모두 수정해야 한다.

두번째 문제는 DB 커넥션을 제공하는 클래스가 어떤 것인지 UserDao 가 알고 있어야 한다.

UserDao 내부에 SimpleConnectionMaker 클래스를 생성하는 코드가 있기 때문에, 다른 DB 커넥션 제공 클래스를 구현하려면 UserDao 를 수정해야 한다.

이런 문제의 근본적인 원인은 UserDao 가 DB 커넥션을 가져오는 클래스에 대해 구체적으로 알고 있기 때문이다.

### 1.3.2 인터페이스의 도입

이를 해결하기 위해 중간에 추상적인 느슨한 연결고리를 만들어 줄 수 있다.

추상화란 어떤 것들의 공통적인 성격을 뽑아내어 이를 따로 분리해내는 작업이다.

자바가 추상화를 위해 제공하는 가장 유용한 도구는 인터페이스다.

SimpleConnectionMaker 를 추상화한 ConnectionMaker 인터페이스를 만들어 본다.

```java
public interface ConnectionMaker {

  public abstract Connection makeConnection() throws ClassNotFoundException, SQLException;

}
```

UserDao 가 ConnectionMaker 인터페이스를 사용하게 한다면 ConnectionMaker 인터페이스 타입의 오브젝트라면 무엇이든 makeConnection() 메소드를 호출해서 Connection 타입의 오브젝트를 돌려줄 것이라 기대할 수 있다. 

그리고 D 사와 N 사에서는 ConnectionMaker 인터페이스를 구현한 DConnectionMaker 클래스를 만들어서 사용하면 된다.

```java
public class DConnectionMaker implements ConnectionMaker {

  public Connection makeConnection() throws ClassNotFoundException, SQLException {
    // ...
  }

}
```

아래는 인터페이스를 사용해서 DB 커넥션을 가져와 사용하도록 수정한 UserDao 코드이다.

```java
public class UserDao {
  private ConnectionMaker connectionMaker;

  public UserDao() {
    this.connectionMaker = new DConnectionMaker();
  }

  public void add(User user) throws ClassNotFoundException, SQLException {
    Connection c = this.connectionMaker.makeConnection();
    // ...
  }

  public User get(String id) throws ClassNotFoundException, SQLException {
    Connection c = this.connectionMaker.makeConnection();
    // ...
  }

  // ...
}
```

언뜻 괜찮아 보이는데, UserDao 를 찬찬히 보면 문제가 있다.

이번에는 UserDao 가 DConnectionMaker 클래스에 종속되어 버렸다.

이렇게 되면 다시 문제가 원점이다.

### 1.3.3 관계설정 책임의 분리

인터페이스까지 써가면서 거의 완벽하게 분리했는데, 왜 UserDao 가 구체적인 클래스를 알아야 할까?

왜냐하면 UserDao 에는 어떤 ConnectionMaker 구현 클래스를 사용할지 결정하는 코드가 아직도 남아 있기 때문이다.

`new DConnectionMaker();` 라는 코드는 짧지만 독립적인 관심사를 담고 있다.

그 관심사는 바로 UserDao 가 어떤 ConnectionMaker 구현 클래스의 오브젝트를 이용하게 할지를 결정하는 것이다.

다시 말하면 UserDao 와 UserDao 가 사용할 ConnectionMaker 의 구현 클래스 사이의 관계를 설정해주는 것에 대한 관심이다.

오브젝트와 오브젝트 사이의 관계를 만들어주기 위해서는 생성자를 직접호출하는 방법도 있지만 외부에서 만든 것을 가져오는 방법도 있다.

외부에서 만든 오브젝트를 전달받으려면 메소드 파라미터나 생성자 파라미터를 이용하면 된다.

UserDao 의 클라이언트 오브젝트에서 UserDao 가 사용할 ConnectionMaker 의 구현 클래스를 결정하고 UserDao 에 제공해 주도록 만들어보자.

이를 위해 UserDao 생성자를 조금 고쳐본다.

UserDao 의 생성 시점에 ConnectionMaker 의 종류를 결정할 수 있도록 하였다.

```java
public class UserDao {

  private ConnectionMaker connectionMaker;

  public UserDao(ConnectionMaker connectionMaker) {
    this.connectionMaker = connectionMaker;
  }

  // ...
}
```

이제 ConnectionMaker 의 생성을 UserDao 를 사용하는 쪽인 UserDaoTest 클래스에 맡긴다.

```java
public class UserDaoTest {
  public static void main(String[] args) throws ClassNotFoundException, SQLException {
    ConnectionMaker connectionMaker = new DConnectionMaker();
    UserDao dao = new UserDao(connectionMaker);

    User user = new User();
    // ...
  }
}
```

이제 UserDao 와 ConnectionMaker 가 잘 분리된 것 같다.

UserDao 는 ConnectionMaker 라는 인터페이스를 통해 구현된 구현체만 받으면 된다.

### 1.3.4 원칙과 패턴

객체지향의 원칙에 대해서 소개한다.

> 객체지향 설계 원칙 (SOLID)
>
> [http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)
>
>- SRP(The Single Responsibility Principle): 단일 책임 원칙
>- OCP(The Open Closed Principle): 개방 폐쇄 원칙
>- LSP(The Liskov Substitution Principle): 리스코프 치환 원칙
>- ISP(The Interface Segregation Principle): 인터페이스 분리 원칙
>- DIP(The Dependency Inversion Principle): 의존관계 역전 원칙

- 개방 폐쇄 원칙: 확장에는 열려있어야 하고, 변경에는 닫혀있어야 한다.
- 높은 응집도: 응집도가 높다는 것은 변화가 일어날 때 해당 모듈에서 변하는 부분이 크다는 뜻
- 낮은 결합도: 책임과 관심사가 다른 오브젝트와는 느슨하게 연결된 형태를 유지해야 한다.
- 전략패턴: 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고, 필요에 따라 바꿔 사용할 수 있게 하는 디자인 패턴이다.
