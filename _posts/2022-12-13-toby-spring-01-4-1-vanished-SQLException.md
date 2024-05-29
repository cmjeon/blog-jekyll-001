---
# layout: single
title: "4장 예외 - 4.1 사라진 SQLException"
# excerpt: "DAO 에 트랜잭션을 적용해보면서 스프링이 어떻게 성격이 비슷한 여러 기술을 추상화하고, 일관된 방법으로 사용할 수 있도록 지원하는지 알아봅니다."
categories:
  - "토비의 스프링 3.1"
tags:
  - "서비스 추상화"
---

## 4.1 사라진 SQLException

deleteAll() 메소드의 정의를 들여다보면 JdbcTemplate 적용 이전에 있었던 throws SQLException 이 사라진 것을 알 수 있다.

```java
// JdbcTemplate 적용 전
public void deleteAll() throws SQLException {
    this. jdbcContext.executeSql("delete from users");
}
```

```java
// JdbcTemplate 적용 후
public void deleteAll() {
    this.jdbcTemplate.update("delete from users");
}
```

### 4.1.1 초난감 예외처리

#### 예외 블랙홀

예외가 발생하면 그것을 catch 블록을 써서 잡아내는 것까지는 좋은데 그러곤 아무것도 하지 않고 별문제 없는 것처럼 넘어가 버리는 건 정말 위험한 일이다.

예외를 처리할 때 반드시 지켜야 할 핵심 원칙은 모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보되어야 한다는 것이다.

예외를 잡아서 뭔가 조치를 취할 방법이 없다면 잡지 말아야 한다.

메소드에 throws SQLException 을 선언해서 메소드 밖으로 던지고 자신을 호출한 코드에 예외처리 책임을 전가해버려라.

#### 무의미하고 무책임한 throws

메소드 선언에 throws Exception 을 기계적으로 붙이는 개발자도 있다.

자신이 사용하려고 하는 메소드에 throws Exception 이 선언되어 있다면 그 메소드 선언에서는 의미 있는 정보를 얻을 수 없다.

이런 메소드를 사용하는 메소드에서는 정보부족으로 예외를 처리할 수 없기 때문에 메소드 선언에 throws Exception 을 따라 붙이는 수밖에 없다.

### 4.1.2 예외의 종류와 특징

자바 개발자들 사이에 예외처리에 관한 큰 이슈는 체크 예외 checked exception 라고 불리는 명시적인 처리가 필요한 예외를 다루는 방법에 관한 것이다.

자바에서 throw 를 통해 발생시킬 수 있는 예외는 크게 세 가지가 있다.

- Error

첫째는 java.lang.Error 클래스의 서브클래스이다. 에러는 시스템에 비정상적인 상황이 발생했을 때 사용된다.

애플리케이션에서는 이런 에러에 대한 처리는 신경쓰지 않는다.

- Exception 과 체크 예외

java.lang.Exception 클래스와 그 서브클래스이다. 애플리케이션 코드의 작업 중에 예외상황이 발생했을 경우에 사용된다.

Exception 클래스는 다시 체크 예외와 언체크 예외 unchecked exception 으로 구분된다.

체크 예외는 Exception 클래스의 서브클래스이지만 RuntimeException 을 상속하지 않은 클래스이다.

언체크 예외는 Exception 클래스의 서브클래스이고 RuntimeException 클래스을 상속한 클래스이다.

일반적으로 예외라고 하면 체크 예외라고 생각해도 된다.

체크 예외가 발생할 수 있는 메소드를 사용할 때는 catch 문으로 잡든 throws 로 던지든 반드시 예외를 처리하는 코드를 함께 작성해야 한다.

그렇지 않으면 컴파일 에러가 발생한다.

- RuntimeException 과 언체크/런타임 예외

java.lang.RuntimeException 클래스와 그 서브클래스이다.

명시적인 예외처리를 강제하지 않기 때문에 언체크 예외, 런타임 예외라고 불린다.

런타임 예외는 주로 애플리케이션의 오류가 있을 때 발생하도록 의도된 것들이다.

대표적으로 NullPointerException, IllegalArgumentException 등이 있다.

런타임 예외는 어플리케이션이 예상하지 못했던 예외상황에서 발생한 게 아니기 때문에 굳이 catch 나 throws 를 사용하지 않아도 된다.























