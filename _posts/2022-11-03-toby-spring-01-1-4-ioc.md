---
layout: single
title: "1장 오브젝트와 의존관계 - 1.4 제어의 역전(IoC)"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 1장 오브젝트와 의존관계

## 1.4 제어의 역전(IoC)

### 1.4.1 오브젝트 팩토리

UserDaoTest 가 하던 UserDao 에 ConnectionMaker 을 등록하는 일을 맡길 UserDaoFactory 를 만든다.

```java title="UserDaoFactory.java"
public class UserDaoFactory {

  public UserDao userDao() {
    UserDao dao = new UserDao(connectionMaker());
    return dao;
  }

  public ConnectionMaker connectionMaker() {
    ConnectionMaker connectionMaker = new DConnectionMaker();
    return connectionMaker;
  }

}
```

<!--truncate-->

이제 UserDaoTest 에서는 UserDao 를 사용하기만 하면 된다.

```java title="UserDaoTest.java"
public class UserDaoTest {

  public static void main(String[] args) throws ClassNotFoundException, SQLException {
    UserDao dao = new UserDaoFactory().userDao();
    // ...
  }

}
```

UserDao 와 ConnectionMaker 는 각각 데이터로직와 기술로직을 담당하게 된다.

UserDaoFactory 는 오브젝트간의 구조와 관계를 정의하는 역할을 맡게 된다.

UserDaoFactory 를 만듦으로써 애플리케이션의 구성요소 역할을 하는 오브젝트와 구조를 결정하는 오브젝트를 분리해냈다.

### 1.4.2 오브젝트 팩토리의 활용

만약 DaoFactory 에 다른 Dao 가 추가된다면 ConnectionMaker 를 생성하는 코드가 반복된다.

```java {4,8,12} title="DaoFactory.java"
public class DaoFactory {

	public UserDao userDaoO {
		return new UserDao(new DConnectionMaker());
	}

	public AccountDao accountDaoO {
		return new AccountDao(new DConnectionMaker());
	}

	public MessageDao messageDaoO {
		return new MessageDao(new DConnectionMaker());
	}

}
```

따라서 ConnectionMaker 를 생성하는 코드를 추출한다.

```java {15-17} title="DaoFactory.java"
public class DaoFactory {

	public UserDao userDaoO {
		return new UserDao(connectionMaker());
	}

	public AccountDao accountDaoO {
		return new AccountDao(connectionMaker());
	}

	public MessageDao messageDaoO {
		return new MessageDao(connectionMaker());
	}

	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker();
	}

)
```

### 1.4.3 제어권의 이전을 통한 제어관계 역전

이제 제어의 역전이라는 개념을 알아보자.

일반적인 프로그램의 흐름은 프로그램 시작점에서 다음에 사용할 오브젝트를 결정하고, 생성하고, 사용한다.

이런 구조에서는 각 오브젝트들이 프로그램의 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여한다.

초기 UserDao 를 보면 main() 메소드가 UserDao 를 직접 생성하고 사용하였다.

UserDao 또한 ConnectionMaker 를 직접 생성하고 사용하였다.

제어의 역전이란 이런 제어 흐름의 개념을 거꾸로 뒤집은 것이다.

제어의 역전에서는 main() 메소드같은 엔트리 포인트를 제외하고 모든 오브젝트는 제어 권한을 위임받은 특별한 오브젝트에 의해 생성된다.

UserDao 와 DaoFactory 에 제어의 역전이 적용되어 있다.

UserDao 가 어떤 ConnectionMaker 를 생성하고 사용할지를 DaoFactory 에 넘겼기 때문이다.

IoC 를 적용함으로써 설계가 깔끔해지고 유연성이 증가하며 확장성이 좋아지기 때문에 필요하면 IoC 스타일의 설계와 코드를 만드는 것이 좋다.

제어의 역전에서는 프레임워크/컨테이너 같은 애플리케이션 컴포넌트의 생성, 관계설정, 생명주기를 관장할 주체가 필요하다.
