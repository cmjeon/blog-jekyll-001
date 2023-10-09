---
layout: single
title: "3장 템플릿 - 3.4 컨텍스트와 DI"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 3장 템플릿

## 3.4 컨텍스트와 DI

### 3.4.1 jdbcContext 의 분리

전략 패턴의 구조로 보면

- UserDao() 메소드 : 클라이언트
- 익명 내부 클래스 : 개별적인 전략
- jdbcContextWithStatementStrategy() 메소드 : 컨텍스트

이다.

jdbcContextWithStatementStrategy() 메소드를 다른 Dao 에서도 사용하기 위해 UserDao 클래스 밖으로 독립시켜보자.

JdbcContext 클래스를 만들고 workWithStatementStrategy() 를 생성한다.

```java
public class JdbcContext {

  private DataSource dataSource;
  
  public void setDataSource(DataSource dataSource) {
    this.dataSource = dataSource;
  }

  public void workWithStatementStrategy(Statementstrategy stmt) throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;
    
    try {
      c = this.dataSource.getConnection();
      ps = stmt.makePreparedStatement(c);
      ps.executeUpdateO;
    } catch (SQLException e) {
      throw e; 
    } finally {
      if (ps != null) { try { ps.closeO; } catch (SQLException e) {} } 
      if (c != null) { try {c.closeO; } catch (SQLException e) {} }
    }
  }
  
}
```

UserDao 는 JdbcContext 를 주입받는다.

```java
public class UserDao {
  
  // highlight-start
  private JdbcContext jdbcContext;
  
  public void setJdbcContext(JdbcContext jdbcContext) {
    this. jdbcContext = jdbcContext;
  }
  // highlight-end
  
  public void add(final User user) throws SQLException {
    // highlight-next-line
    this.jdbcContext.workWithStatementStrategy(
      new Statementstrategy() {
        ...
      }
    );
  }
  
  public void deleteAll() throws SQLException {
    // highlight-next-line
    this.jdbcContext.workWithStatementStrategy(
      new StatementStrategy() {
        ...
      }
    );
  }
  
}
```

JdbcContext 는 그 자체로 독립적인 JDBC 컨텍스트를 제공해주는 서비스 오브젝트이기 때문에 인터페이스를 구현하도록 만들지 않았다는 것을 확인한다.

### 3.4.2 JdbcContext 의 특별한 DI

UserDao 는 인터페이스를 거치치 않고 JdbcContext 를 사용하고 있어서 구현 클래스를 변경할 수 없다.

의존관계 주입 DI 개념을 충실히 따르자면 인터페이스를 둬서 클래스 레벨의 의존관계를 피하고, 런타임 시에 의존할 오브젝트와의 관계를 다이나믹하게 주입해주는 것이 맞다.

하지만 스프링의 DI 를 넓게 보자면 제어권한을 외부로 위임했다는 IoC 개념을 포괄하고 있기 때문에 JdbcContext 를 스프링을 이용해서 UserDao 에 주입했다는 건 DI 의 기본을 따르고 있다고 봐야 한다.

JdbcContext 는 스프링 컨테이너의 싱글톤 레지스트리에서 관리되는 싱글톤 빈이다.

따라서 여러 오브젝트에서 공유해 사용이 가능하다.

JdbcContext 가 DI 를 통해 DataSource 에 의존하고 있다.

DI 를 위해서는 주입되는 오브젝트, 주입받는 오브젝트 양쪽 모두 스프링 빈으로 등록되어야 한다.

JdbcContext 와 UserDao 처럼 매우 긴밀한 관계를 가지고 강하게 결합되어 있다면 JdbcContext 에 대한 DI 필요성을 위해 스프링의 빈으로 등록해서 UserDao 에 DI 되도록 만들어도 좋다.

UserDao 내부에서 직접 DI 를 적용할 수도 있다.

다만 JdbcContext 를 싱글톤으로 만들려는 것은 포기해야 된다.

대신 DAO 마다 하나의 JdbcContext 를 두게 해야 한다.

스프링 컨테이너 대신 JdbcContext 의 생성과 초기화를 UserDao 가 맡는다.

JdbcContext 가 주입받아야 하는 DataSource 는 UserDao 가 대신 주입받는다.

JdbcContext 는 이제 빈이 아니라 주입받을 수 없기 때문이다.

```java
public class UserDao {

  private JdbcContext jdbcContext;
  
  public void setDataSource(DataSource dataSource) {
    this.jdbcContext=newJdbcContext();
    // highlight-next-line
    this.jdbcContext.setDataSource(dataSource);
    this.dataSource = dataSource;
  }
  
}
```

수동으로 DI 를 하는 방법으로 굳이 인터페이스를 두지 않아도 될 만큼 긴밀한 DAO 와 JdbcContext 라면 내부에서 직접 만들어 사용하면서도 다른 오브젝트에 대한 DI 를 적용할 수 있다.
