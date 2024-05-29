---
layout: single
title: "3장 템플릿 - 3.2 변하는 것과 변하지 않는 것"
categories:
  - "토비의 스프링 3.1"
tags:
  - "템플릿"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 3장 템플릿

## 3.2 변하는 것과 변하지 않는 것

### 3.2.1 JDBC try/catch/finally 코드의 문제점

try/catch 로 예외 시에 리소스를 반환하지 않고 종료되는 것은 해결했지만 코드가 지저분해졌다.

테스트를 통해 DAO 마다 예외상황에서 리소스를 반납하는지 체크할 수 도 있지만 예외상황을 강제로 만드는 것도 쉽지 않다.

### 3.2.2 분리와 재사용을 위한 디자인 패턴 적용

재사용할 수 있는 부분과 바뀌는 부분을 찾아보자

```java
// highlight-start
Connection c = null;
PreparedStatement ps = null;
try {
  c = dataSource.getConnection();
  // highlight-end
  
  ps = c.prepareStatement("delete from users");
  
  // highlight-start
  ps.executeUpdate();
} catch (SQLException e) {
  throw e; 
} finally {
  if (ps != null) { try { ps.close()); } catch (SQLException e) {} }
  if (c != null) { try {c.close()); } catch (SQLException e) {} } )
}
// highlight-end
```

하이라이트 된 부분은 재사용할 수 있다.

예컨데 add() 메소드라고 해도 하이라이트된 부분이 아닌 곳만 변경하면 된다.

메소드 추출하기로 재사용을 시도해본다.

변하지 않는 부분이 변하는 부분을 감싸고 있기 때문에 변하지 않는 부분을 메소드 추출하기에 여의치 않다.

그래서 바뀌는 부분을 메소드 추출하기를 적용했더니, 재사용할 수가 없다.

템플릿 메소드 패턴을 적용해본다.

템플릿 메소드 패턴은 상속을 통해 기능을 확장해서 사용하는 부분이다.

변하지 않는 부분은 슈퍼클래스에 두고 변하는 부 분은 추상 메소드로 정의해둬서 서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 하는 것이다.

makeStatement() 메소드를 추상메소드로 선언하고 UserDaoDeleteAll 자식 클래스를 만들어 구현한다.

```java
// ...

abstract protected PreparedStatement makeStatement(Connection c) throws SQLException;

// ...
```

```java
public class UserDaoDeleteAll extends UserDao {
  protected PreparedStatement makeStatement(Connection c) throws SQLException { 
    PreparedStatement ps = c.prepareStatement("delete from users");
    return ps;
  }
}
```

템플릿 메소드 패턴은 개방 폐쇄 원칙을 지킬 수 있는 괜찮은 방법이다.

하지만 DAO 로직마다 상속을 통해 새로운 클래스를 만들어야 한다는 문제가 있다.

만약 UserDao 의 JDBC 메소드가 4개인 경우 4개의 서브클래스가 필요하다.

또 다른 문제가 확장구조가 이미 클래스를 설계하는 시점부터 고정되어서 관계에 대한 유연성이 떨어진다는 것이다.

전략패턴을 적용해볼 수 있다.

전략패턴은 템플릿 메소드 패턴보다 유연하고 확장성이 뛰어나다.

오브젝트를 아예 둘로 분리하고 클래스 레벨에서는 인터페이스를 통해서만 의존하도록 만드는 것이다.

변하지 않는 부분이 컨텍스트 영역이고 특정한 기능은 Strategy 인터페이스를 통해 외부의 독립된 전략 클래스에 위임하는 것이다.

[![application-context-work](/assets/images/posts/2022-11-25-toby-spring-01-3-2-changes-and-does-not/strategy-pattern.png)](/assets/images/posts/2022-11-25-toby-spring-01-3-2-changes-and-does-not/strategy-pattern.png)

deleteAll() 의 기능을 다시 살펴보자

- DB 커넥션 가져오기
- PreparedStatement 를 만들어줄 외부 기능 호출하기
- 전달받은 PreparedState 실행하기
- 예외가 발생하면 이를 다시 메소드 밖으로 던지기
- 모든 경우에 만들어진 PreparedStatement 와 Connection 적절히 닫아주기

눈여겨 볼 것은 PreparedStatement 를 생성할 때 Connection 을 전달해줘야 한다는 점이다.

전략 인터페이스를 구현하는 클래스를 만들어보자.

```java
public class DeleteAllStatement implements StatementStrategy {

  public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
    PreparedStatement ps = c.prepareStatement("delete from users");
    return ps;
  }
  
}
```

확장된 PreparedStrategy 전략인 DeleteAllStatement 가 만들어졌다.

이것을 사용하여 deleteAll 메소드를 만들어본다.

```java
// ...
public void deleteAll() throws SQLException {
  try (
    c = dataSource,getConnection();
    StatementStrategy strategy = new DeleteAllStatement();
    // highlight-start
    ps = strategy.makePreparedStatement(c);
    // highlight-end
    ps.executeUpdate();
  } catch (SQLException e) {
)
```

하지만 여기서도 특정 구현 클래스인 DeleteAllStatement 를 만들기 때문에 OCP 에 잘 맞다고 볼 수 없다.

이제는 DI 를 적용해보자

전략 패턴에서 컨텍스트가 어떤 전략을 사용할지는 컨텍스트를 사용하는 클라이언트에서 정해주어야 한다.

전략 객체의 생성과 컨텍스트로의 전달을 맡았던 것이 ObjectFactory 이다.

DI 는 이런 전략 패턴의 장점을 일반적으로 활용할 수 있도록 만든 구조이다.

컨텍스트를 메소드로 분리시키고 StatementStrategy 를 받을 수 있도록 하자

```java
// highlight-start
public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
// highlight-end

  Connection c = null;
  PreparedStatement ps = null;
  
  try {
    c = dataSource.getConnection();
    // highlight-start
    ps = stmt.makePreparedStatement(c);
    // highlight-end
    ps.executeUpdate();
  } catch (SQLException e) {
    throw e; 
  } finally {
    if (ps != null) { try { ps.close(); } catch (SQLException e) {} } 
    if (c != null) { try {c.close(); } catch (SQLException e) {} }
  }
  
}
```

컨텍스트 메소드를 호출할 클라이언트를 만든다.

클라이언트는 전략을 생성하고 컨텍스트 메소드를 호출한다.

```java
public void deleteAll() throws SQLException {
  StatementStrategy st = new DeleteAllStatement();
  jdbcContextWithStatementStrategy(st);
}
```

이제 구조상 전략패턴이 되었다.

> DI 의 가장 중요한 개념은 제3자의 도움을 통해 두 오브젝트 사이의 유연한 관계가 설정되도록 만든다는 것이다.
>
> 일반적으로 DI 는 의존관계에 있는 두 개의 오브젝트와 이 관계를 다이내믹하게 설정해주는 오브젝트 팩토리(DI 컨테이너), 그리고 이를 사용하는 클라이언트라는 4개의 오브젝트 사이에서 일어난다.
>
> DI의 장점을 단순화해서 loC 컨테이너의 도움 없이 코드 내에서 적용한 경우를 마이크로 DI라고 한다.
