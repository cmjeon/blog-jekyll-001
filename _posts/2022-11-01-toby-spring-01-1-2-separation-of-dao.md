---
layout: single
title: "1장 오브젝트와 의존관계 - 1.2 DAO 의 분리"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 1장 오브젝트와 의존관계

## 1.2 DAO 의 분리

### 1.2.1 관심사의 분리

> 객체지향의 세계에서는 모든 것이 변한다.오브젝트에 대한 설계와 이를 구현한 코드가 변한다는 뜻이다.
> 
> 1장_ 오브젝트와 의존관계, 60.

개발자가 객체를 설계할 때 가장 염두에 둬야 할 사항은 미래의 변화를 대비하는 것이다.

객체지향적인 코드를 위해서는 `관심사의 분리` 가 중요하다.

### 1.2.2 커넥션 만들기의 추출

현재 UserDao 의 현재 관심사항은 아래와 같다.

1. DB 와 연결을 위한 커넥션을 가져오기
2. 사용자 등록을 위해 statement 를 만들고 실행하기
3. 작업이 끝나면 statement 와 connection 오브젝트를 닫고 리소스를 돌려주기

우선 UserDao 에서 DB 커넥션을 가져오는 중복코드를 getConnection() 메소드로 추출한다.

```java
public class UserDao {

  public void add(User user) throws ClassNotFoundException, SQLException {
    Connection c = getConnection();
    // ...
  }

  public User get(String id) throws ClassNotFoundException, SQLException {
    Connection c = getConnection();
    // ...
    return user;
  }

  private Connection getConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
    return c;
  }

  // ...

}
```

코드를 변경했으니 다시 main() 메소드로 테스트한다.

기능에는 변화를 주지 않으면서 이전보다 소스코드를 깔끔하게 하는 것을 리팩토링이라고 한다.

DB 커넥션을 getConnection() 메소드로 추출한 것처럼 중복된 코드를 뽑아내는 것을 리팩토링에서 `메소드 추출` 기법이라고 한다.

### 1.2.3 DB 커넥션 만들기의 독립

이번엔 다른 기능을 생각해본다.

만약 UserDao 에 다른 종류의 DB 커넥션이 필요하다면 어떻게 해야할까?

상속을 통한 기능확장으로 문제를 해결해보자.

getConnection() 메소드를 추상메소드로 만들고, 자식클래스에서 getConnection() 메소드의 구체적인 기능을 구현하도록 한다.

```java
public abstract class UserDao {

  public void add(User user) throws ClassNotFoundException, SQLException {
    Connection c = getConnection();
    // ...
  }

  public User get(String id) throws ClassNotFoundException, SQLException {
    Connection c = getConnection();
    // ...
  }

  abstract protected Connection getConnection() throws ClassNotFoundException, SQLException ;

  // ...
}
```

UserDao 를 상속하는 DUserDao 와 NUserDao 를 만들고, getConnection() 메소드를 구현한다.

```java
public class DUserDao extends UserDao {

  protected Connection getConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
    return c;
  }

}
```

```java
public class NUserDao extends UserDao {

  protected Connection getConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
    return c;
  }

}
```

부모클래스에 기본적인 흐름과 추상 메소드를 구현하고 자식클래스에서 추상 메소드를 구현하여 사용하는 방법을 `템플릿 메소드 패턴 template method pattern` 이라고 한다.

> 슈퍼클래스에 기본적인 로직의 흐름(커넥션 가져오기 SQL 생성, 실행 반환)을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법을 디자인 패턴에서 템플릿 메소드 패턴 template method pattern 이라고 한다.
>
> 1장_ 오브젝트와 의존관계, 67.

UserDao 에서는 getConnection() 으로 Connection 타입의 객체를 사용하는 것에만 관심이 있을 수 있다.

그렇게 되면 getConnection() 으로 실제 Connection 객체를 생성하는 것은 자식클래스의 관심사항이다.

자식클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 `팩토리 메소드 패턴 factory method pattern` 이라고 한다.

> 서브클래스 에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 팩토리 메소드 패턴 factory method pattern 이라고 부르기도 한다.
> 
> 1장_ 오브젝트와 의존관계, 67.

상속구조를 통해서 이렇게 사용과 생성이라는 관심사를 분리한 코드를 만들었고, 서로 영향을 덜 주도록 했다.

하지만 상속은 많은 한계점이 있다.

자바는 클래스의 다중상속을 허용하지 않기 때문에 상속을 적용한 부모클래스는 다른 상속을 적용하지 못한다.

상속관계는 두 가지 다른 관심사에 대한 긴밀한 결합을 허용한다.

자식클래스에서 부모클래스의 기능을 직접 사용할 수도 있다.

UserDao 외에 Dao 클래스가 만들어질 때마다 getConnection() 의 구현 코드가 매 Dao 클래스마다 중복되서 나타나야 하는 문제가 발생한다.
