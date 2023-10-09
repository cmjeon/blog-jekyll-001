---
layout: single
title: "3장 템플릿 - 3.3 JDBC 전략 패턴의 최적화"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 3장 템플릿

## 3.3 JDBC 전략 패턴의 최적화

### 3.3.1 전략 클래스의 추가 정보

add() 메소드에는 부가정보인 User 가 필요하다.

AddStatement 클래스를 만들고 생성자를 통해 User 를 주입받도록 한다.

```java
public class AddStatement implements StatementStrategy {
  User user;
  
  public AddStatement(User user) {
    this.user = user;
  }
  
  // ...
  
}
```

이제 deleteAll(), add() 메소드가 PreparedStatement 를 실행하는 컨텍스트를 공유하게 되었다.

이렇게 공유할 수 있는 코드를 통해 코드를 간결하게 가져갈 수 있다.

### 3.3.2 전략과 클라이언트의 동거

코드가 깔끔해지긴 했지만 여전히 개선할 부분이 있다.

현재 구현으로는 메소드마다 새로운 StatementStrategy 구현 클래스를 만들어야 한다.

또한 StatementStrategy 에 전달할 User 와 같은 부가적인 정보가 있는 경우, 생성자와 인스턴스 변수가 추가로 필요하다.

StatementStrategy 구현 클래스를 UserDao 클래스 안에 내부 클래스로 정의해 보자

특정 메소드에서만 사용되는 것이라면 로컬 클래스로 만들 수 있다.

```java
public void add(final User user) throws SQLException {

  // highlight-start
  class AddStatement implements StatementStrategy {
  
    User user;
    
    public AddStatement(User user) {
      this.user = user;
    }
  
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
      PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)"); 
      ps.setString(1, user.getId());
      ps.setString(2, user.getName());
      ps.setString(3, user.getPassword());
      return ps;
    }
    
  }
  // highlight-end

  // highlight-next-line
  StatementStrategy st = new AddStatement(user);
  jdbcContextWithStatementStrategy(st);
  
}
```

> 중첩 클래스의 종류
> 
> 다른 클래스 내부에 정의되는 클래스를 중첩 클래스(nested class)라고 한다.
>
> 중첩 클래스는 두 가지로 구분된다.
>- static class : 독립적으로 오브젝트로 만들어질 수 있음
>- inner class : 자신이 정의된 클래스의 오브젝트 안에서만 만들어질 수 있음
>
>내부 클래스는 다시 범위(scope)에 따라 세 가지로 구분된다.
>- member inner class : 멤버 필드처럼 오브젝트 레벨에 정의됨
>- local class : 메소드 레벨에 정의됨
>- anonymous inner class : 이름을 갖지 않음
>
>익명 내부 클래 스의 범위는 선언된 위치에 따라서 다르다.
:::

좀 더 간결하게 하기 위해서 익명 내부 클래스로 변경할 수 있다.

```java
public void add(final User user) throws SQLException { 
  jdbcContextWithStatementStrategy(
    new StatementStrategy() {
      public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)"); 
        ps.setString(1, user.getId())); 
        ps.setString(2, user.getName()); 
        ps.setString(3, user.getPassword());
        return ps;
      }
    }
  );
}
```

>익명 내부 클래스
> 
>익명 내부 클래스(anonymous inner class)는 이름을 갖지않는 클래스다.
>
>클래스 선언과 오브젝트 생성이 결합된 형태로 만들어지며, 상속할 클래스나 구현할 인터페이스를 생성자 대신 사용해서 다음과 같은 형태로 만들어 사용한다.
>
>클래스를 재사용할 필요가 없고, 구현한 인터페이스 타입으로만 사용할 경우에 유용하다.
>
>new 인터페이스이름() { 클래스 본문 };
>
