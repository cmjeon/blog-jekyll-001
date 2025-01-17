---
layout: single
title: "1장 오브젝트와 의존관계 - 1.2 DAO 의 분리"
categories:
  - "토비의 스프링 3.1"
tags:
  - "오브젝트와 의존관계"
---

## 1.2 DAO 의 분리

### 1.2.1 관심사의 분리

<div class="notice--primary" markdown="1">
객체지향의 세계에서는 모든 것이 변한다.<br>
여기서 변한다는 것은 변수나 오브젝트 필드의 값이 변한다는 게 아니다. 오브젝트에 대한 설계와 이를 구현한 코드가 변한다는 뜻이다.<br>
...<br>
사용자의 비즈니스 프로세스와 그에 따른 요구사항은 끊임없이 바뀌고 발전한다.<br>
...<br>
그래서 개발자가 객체를 설계할 때 가장 염두에 둬야 할 사항은 바로 미래의 변화를 어떻게 대비할 것인가이다.<br>
<br>
1장_ 오브젝트와 의존관계, 60.<br>
</div>

<div class="notice--primary" markdown="1">
객체지향 기술은 흔희 실세계를 최대한 가깝게 모델링해낼 수 있기 때문에 의미가 있다고 여겨진다.<br>
하지만 그보다는 객체지향 기술이 만들어내는 가상의 추상세계 자체를 효과적으로 구성할 수 있고, 이를 자유롭고 편리하게 변경, 발전, 확장시킬 수 있다는 데 더 의미가 있다.<br>
<br>
1장_ 오브젝트와 의존관계, 60.<br>
</div>

미래를 준비하는 데 있어 가장 중요한 과제는 변화에 대비하는 것이다.

가장 좋은 대책은 변화의 폭을 최소한으로 줄여주는 것이다.

변경이 일어날 때 필요한 작업을 최소화하고, 그 변경이 다른 곳에 문제를 일으키지 않게 하는 것은 `분리와 확장`을 고려한 설계로 가능하다.

변화는 대체로 집중된 한 가지 관심에 대해 일어난다. 하지만 그에 따른 작업은 한곳에 집중되지 않는 경우가 많다.

변화가 한 번에 한 가지 관심에 집중돼서 일어난다면, 우리가 준비해야 할 일은 한 가지 관심이 한 군데에 집중되게 만드는 것이다.

프로그래밍의 기초 개념 중에 `관심사의 분리` 라는 게 있다.

이를 객체지향에 적용해보면, 관심이 같은 것끼리는 하나의 객체 안으로 모이게 하고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것이다.

### 1.2.2 커넥션 만들기의 추출

현재 UserDao 의 add() 메소드 하나에서 적어도 세 가지 관심사항을 확인할 수 있다.

#### UserDao의 관심사항

1. DB 와 연결을 위한 커넥션을 가져오기
2. 사용자 등록을 위해 statement 를 만들고 실행하기
3. 작업이 끝나면 statement 와 connection 오브젝트를 닫고 리소스를 돌려주기

#### 중복 코드의 메소드 추출

우선 UserDao 에서 DB 와 연결을 위해 커넥션을 가져오는 중복코드를 getConnection() 메소드로 추출한다.

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
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?", "spring", "book");
    return c;
  }

  // ...

}
```

커넥션에 대한 관심 내용이 독립적으로 존재하게 되서 수정이 간단해졌다.

#### 변경사항에 대한 검증: 리팩토링과 테스트

방금 한 작업은 UserDao 의 기능에는 아무런 변화를 주지 않았다.

하지만 특정 관심사항이 담긴 코드를 별도의 메소드로 분리하는 구조의 변화가 있었다.

기능에는 변화를 주지 않으면서 이전보다 소스코드를 깔끔하게 하는 것을 `리팩토링`이라고 한다.

그리고 이렇게 중복된 코드를 뽑아내는 것을 리팩토링에서 `메소드 추출` 기법이라고 한다.

### 1.2.3 DB 커넥션 만들기의 독립

변화에 유연하게 대처할 수 있는 코드가 되었지만 더 나아가서 변화를 반기는 DAO 를 만들어보자.

만약 UserDao 에 다른 종류의 DB 커넥션이 필요하다면 어떻게 해야할까?

UserDao 의 코드를 수정하지 않고 다른 DB 커넥션으로 UserDao 을 사용하게 할 수 있을끼?

#### 상속을 통한 확장

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

  abstract protected Connection getConnection() throws ClassNotFoundException, SQLException;
    
}
```

UserDao 를 상속하는 DUserDao 와 NUserDao 를 만들고, getConnection() 메소드를 구현한다.

```java
public class DUserDao extends UserDao {

  protected Connection getConnection() throws ClassNotFoundException, SQLException {
    // D 사의 DB connection 생성코드
  }

}
```

```java
public class NUserDao extends UserDao {

  protected Connection getConnection() throws ClassNotFoundException, SQLException {
      // N 사의 DB connection 생성코드
  }

}
```

부모클래스에 기본적인 흐름과 추상 메소드를 구현하고 자식클래스에서 추상 메소드를 구현하여 사용하는 방법을 `템플릿 메소드 패턴 template method pattern` 이라고 한다.

<div class="notice--primary" markdown="1">
슈퍼클래스에 기본적인 로직의 흐름(커넥션 가져오기 SQL 생성, 실행 반환)을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법을 디자인 패턴에서 템플릿 메소드 패턴 template method pattern 이라고 한다.<br>
<br>
1장_ 오브젝트와 의존관계, 67.<br>
</div>

UserDao 에서는 getConnection() 으로 Connection 타입의 객체를 사용하는 것에만 관심이 있을 수 있다.

그렇게 되면 getConnection() 으로 실제 Connection 객체를 생성하는 것은 자식클래스의 관심사항이다.

자식클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 `팩토리 메소드 패턴 factory method pattern` 이라고 한다.

오브젝트를 생성하는 방식이 다르다면 이는 팩토리 메소드 패턴으로 이해할 수 있다.

<div class="notice--primary" markdown="1">
서브클래스 에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 팩토리 메소드 패턴 factory method pattern 이라고 부르기도 한다.<br>
<br>
1장_ 오브젝트와 의존관계, 67.<br>
</div>

상속구조를 통해서 사용과 생성이라는 관심사를 분리한 코드를 만들었고, 서로 영향을 덜 주도록 했다는 것으로 이해하면 된다.

<div class="notice--primary" markdown="1">
서브클래스에서 오브젝트 생성 방법과 클래스를 결정할 수 있도록 미리 정의해둔 메소드를 팩토리 메소드라고 하고, 이 방식을 통해 오브젝트 생성 방법을 나머지 로직, 즉 슈퍼클래스의 기본 코드에서 독립시키는 방법을 팩토리 메소드 패턴이라고 한다.<br>
<br>
1장_ 오브젝트와 의존관계, 70.<br>
</div>

템플릿 메소드 패턴 또는 팩토리 메소드 패턴으로 관심사항이 다른 코드를 분리해내고, 서로 독립적으로 변경 또는 확장할 수 있도록 만든 것은 효과적인 방법일 수 있다.

상속을 이용한 방법은 간단하고 사용하기 편리하지만 많은 한계점이 있다.

자바는 클래스의 다중상속을 허용하지 않기 때문에 만약 상속을 사용하고 있는 부모클래스라면 다른 상속을 적용하지 못한다.

또한 상속을 통한 상하위 클래스의 관계는 생각보다 밀접하고, 두 가지 다른 관심사에 대한 긴밀한 결합을 허용한다.

예컨데 자식클래스가 부모클래스의 기능을 직접 사용할 수도 있다.

부모클래스 내부의 변경이 있을 때 서브클래스를 함께 수정해야 할 수도 있다. 혹은 부모클래스가 변경되지 않도록 제약해야 할 수도 있다.

DB 커넥션을 생성하는 코드를 다른 DAO 에서 재사용하지 못하는 것도 단점이다. UserDao 외에 Dao 클래스가 만들어지면 getConnection() 의 구현 코드를 Dao 클래스에 중복해줘야 하는 문제가 발생한다.
