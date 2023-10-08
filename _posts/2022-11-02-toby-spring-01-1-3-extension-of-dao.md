---
layout: single
title: "1장 오브젝트와 의존관계 - 1.3 DAO 의 확장"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 1장 오브젝트와 의존관계

## 1.3 DAO 의 확장

관심사에 따라 분리한 오브젝트들은 제각기 독특한 변화의 특징을 가진다.

변화의 특징이 있다는 것은 변화의 이유와 시기 등이 다르다는 뜻이다.

상속은 여러가지 단점이 많이 때문에 독립적인 클래스로 분리하는 것을 고민해야 한다.

### 1.3.1 클래스의 분리

DB 커넥션을 생성하는 SimpleConnectionMaker 클래스를 만든다.

UserDao 를 생성할 때 SimpleConnectionMaker 클래스로 객체를 만들고 필요할 때 가져다 쓰도록 한다.

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

```java
public class SimpleConnectionMaker {

  public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
    return c;
  }

}
```

이렇게 되면 이제는 DB 커넥션 기능의 확장이 어렵다.

더이상 NUserDao, DUserDao 같이 다른 getConnection() 메소드의 기능을 구현할 수가 없다.

왜냐하면 UserDao 가 SimpleConnectionMaker 클래스에 종속되어 있기 때문이다.

N 사와 D 사의 getConnection 을 쓰기 위해서는 UserDao 의 수정이 반드시 필요해졌다.

### 1.3.2 인터페이스의 도입

이를 해결하기 위해 SimpleConnectionMaker 를 추상화한 ConnectionMaker 인터페이스를 만들어 본다.

```java
public interface ConnectionMaker {

  public abstract Connection makeConnection() throws ClassNotFoundException, SQLException;

}
```

그리고 D 사와 N 사에서는 ConnectionMaker 인터페이스를 구현한 DConnectionMaker 클래스를 만들어서 사용하면 된다.

```java
public class DConnectionMaker implements ConnectionMaker {

  public Connection makeConnection() throws ClassNotFoundException, SQLException {
    // ...
  }

}
```

괜찮아 보인다.

그런데 UserDao 에서 쓸려고 보면 문제가 발생한다.

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

이번에는 UserDao 가 DConnectionMaker 클래스에 종속되어 버렸다.

이렇게 되면 다시 문제가 원점이다.

### 1.3.3 관계설정 책임의 분리

문제를 해결하기 위해 UserDao 생성자를 조금 고쳐본다.

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
