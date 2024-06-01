---
title: "4장 예외 - 4.2 예외 전환"
categories:
  - "토비의 스프링 3.1"
tags:
  - "예외"
---

## 4.2 예외 전환

예외를 다른 것으로 바꿔서 던지는 예외 전환의 목적은 두 가지이다.

하나는 런타임 예외로 포장해서 굳이 필요하지 않는 catch/throws 를 줄여주는 것이다.

다른 하나는 로우레벨의 예외를 좀 더 의미 있고 추상화된 예외로 바꿔서 던져주는 것이다.

### 4.2.1 JDBC의 한계

JDBC 는 자바를 이용해 DB 에 접근하는 방법을 추상화된 API 형태로 정의해놓고, 각 DB 업체가 JDBC 표준을 따라 만들어진 드라이버를 제공하게 해준다.

인터페이스를 사용하는 객체지향 프로그래밍 방법의 장점을 잘 경험할 수 있는 것이 바로 JDBC 이다.

하지만 JDBC API 가 현실적으로 DB 를 자유롭게 바꾸어 사용할 수 있게 하는데는 두 가지 걸림돌이 있다.

#### 비표준 SQL

첫째 문제는 JDBC 코드에서 사용하는 SQL 이다.

SQL 이 어느정도 표준화되었다곤 해도 대부분의 DB 는 표준을 따르지 않는 비표준 문법과 기능도 제공한다.

비표준 SQL 은 결국 DAO 코드에 들어가고, 해당 DAO 는 특정 DB 에 대해 종속적인 코드가 되고 만다.

표준 SQL 만을 사용하는 방법은 현실성이 없고, DAO 를 DB 별로 만들어 사용하거나 SQL 을 외부에서 독립시켜 바꿔 쓸 수 있게 하는 것을 생각해 볼 수 있다.

#### 호환성 없는 SQLException 의 DB 에러정보

두 번째 문제는 SQLException 이다.

SQLException 은 DB 에러 정보를 담고 있는데, DB 에러는 DBMS 마다 다르게 정의되어 있다.

게다가 에러의 종류와 원인도 제각각이다.

그래서 JDBC 는 데이터 처리 중에 발생하는 다양한 예외를 그냥 SQLException 하나에 모두 담아버린다.

예외가 발생한 원인은 SQLException 안에 담긴 에러 코드와 SQL 상태정보를 참조해야 한다.

그런데 SQLException 안에 담긴 에러코드와 SQL 상태정보는 DB 별로 모두 다르다.

상태정보 또한 DB 의 JDBC 드라이버에서 SQLException 에 담을 상태코드를 정확하게 만들어주지 않는다.

결국 호환성없는 에러 코드와 표준을 잘 따르지 않는 상태 코드를 가진 SQLException 만으로 DB 에 독립적인 유연한 코드를 작성하는 건 불가능에 가깝다.

### 4.2.2 DB 에러 코드 매핑을 통한 전환

SQLException 에 담긴 DB 별 에러 코드를 참고해서 발생한 예외의 원인이 무엇인지 해석해 주는 기능을 만들어 본다.

스프링은 DataAccessException 이라는 SQLException 을 대체할 수 있는 런타임 예외를 정의하고, DataAccessException 의 서브클래스로 세분화된 예외 클래스들을 정의하고 있다.

BadSqlGrammarException, DataAccessResourceFailureException, DuplicateKeyException, DataIntegrityViolationException 등이 있다.

스프링은 스프링이 정의한 예외 클래스와 DB별 에러 코드를 분류해서 매핑해놓은 테이블을 만들어두고 이를 이용한다.

JdbcTemplate 은 SQLException 의 DB 에러 코드로 DataAccessException 의 서브클래스 하나로 매핑해준다.

JdbcTemplate 을 이용한다면 JDBC 에서 발생하는 DB 관련 예외는 거의 신경 쓰지 않아도 된다.

만약 에러가 발생했을 때 애플리케이션에서 직접 정의한 예외를 발생시키고 싶을 수 있다.

예컨데 중복키 에러 DuplicatedKeyException 를 애플리케이션에서 정의한 DuplicateUserIdException 으로 발생시키고 싶다고 한다면, 예외를 전환해주는 코드를 DAO 에 넣으면 된다.

```java
public void add() throws DuplicateUserldException {
  try {
    // jdbcTemplate을 이용해 User를 add 하는 코드
  } catch (DuplicateKeyException e) {
    // 로그를 남기는 등의 필요한 작업
    throw new DuplicatellserldException(e);
  }
}
```

### 4.2.3 DAO 인터페이스와 DataAccessException 계층구조

DataAccessException 은 JDBC 의 SQLException 을 전환하는 용도로만 만들어진 건 아니다.

DataAccessException 은 의미가 같은 예외라면 데이터 액세스 기술의 종류와 상관없이 일관된 예외가 발생하도록 만들어준다.

#### DAO 인터페이스와 구현의 분리

DAO 를 굳이 따로 만들어서 사용하는 이유는 무엇일까?

가장 중요한 이유는 데이터 액세스 로직을 담은 코드를 성격이 다른 코드에서 분리해놓기 위해서다.

그리고 전략 패턴을 적용해 DAO 구현 방법을 변경할 수 있게 만들기 위해서이기도 하다.

DAO 의 사용 기술과 구현 코드는 클라이언트로부터 감출 수 있지만 메소드 선언에 나타나는 예외정보가 문제가 된다.

UserDao 의 인터페이스를 분리해서 기술에 독립적인 인터페이스로 만드려면 메소드 선언에 예외가 없어야 한다.

DAO 에서 사용하는 데이터 액세스 기술의 API 가 예외를 던지기 때문에 메소드 선언에 예외가 필요하다.

게다가 데이터 액세스 기술의 API 는 저마다 독자적인 예외를 던지기 때문에 메소드 선언에 SQLException 을 사용할 수 없다.

이 경우, 메소드 선언에 모든 예외를 다 받아주는 throws Exception 을 사용하는 것을 생각해볼 수 있지만 이는 무책임한 방식이다.

JDBC 외에 JDO, Hibernate, JPA 등의 기술은 런타임 예외를 사용한다.

그렇다면 JDBC API 를 직접 사용하는 DAO 만 메소드 내에서 런타임 예외로 포장해서 던져줄 수 있다면 메소드 선언에서 예외를 없앨 수 있다.

이제 DAO 에서 사용하는 기술에 완전히 독립적인 인터페이스 선언이 가능해진다.

메소드 선언에서 예외가 사라지더라도 비즈니스 로직에서 의미 있게 다루어야 하는 예외는 분명 있다.

문제는 데이터 액세스 기술에 따라 다른 런타임 예외가 발생한다는 점이다.

결국 클라이언트가 DAO 의 기술에 의존적이 된다.

단지 인터페이스를 추상화하고, 런타임 예외로 전환하는 것만으로는 불충분하다는 뜻이다.

#### 데이터 액세스 예외 추상화와 DataAccessException 계층구조

스프링은 데이터 액세스 기술을 사용할 때 발생하는 예외들을 추상화해서 DataAccessException 계층구조 안에 정리해두었다.

스프링의 DataAccessException 은 일부 기술에서만 나타나능 예외를 포함해서 데이터 액세스 기술에서 발생 가능한 대부분의 예외를 계층구조로 분류해놓았다.

예를 들어 JPA, 하이버네이트처럼 오브젝트/엔티티 단위로 정보를 업데이트 하기 때문에 낙관적인 락킹 optimistic locking 이 발생한다.

이 경우에 스프링의 예외 전환 방법을 적용하면 기술에 상관없이 ObjectOptimisticLockingFailureException 이라는 예외를 던진다.

<div class="notice--primary" markdown="1">
낙관적인 락킹은 같은 정보를 두 명 이상의 사용자가 동시에 조회하고 순차적으로 업데이트를 할 때, 뒤늦게 업데이트 한 것이 먼저 업데이트한 것을 덮어쓰지 않도록 막아주는 데 쓸 수 있는 편리한 기능이다.<br>
<br>
4장_ 예외, 306.<br>
</div>

JdbcTemplate 과 같이 스프링의 데이터 액세스 지원 기술을 이용해 DAO 를 만들면 사용 기술에 독립적인 일관성 있는 예외를 던질 수 있다.

결국 인터페이스 사용, 런타임 예외 전환과 함께 DataAccessException 예외 추상화를 적용하면 데이터 액세스 기술과 구현 방법에 독립적인 이상적인 DAO 를 만들 수가 있다.

### 4.2.4 기술에 독립적인 UserDao 만들기


















