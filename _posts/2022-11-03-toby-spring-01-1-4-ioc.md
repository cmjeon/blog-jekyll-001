---
layout: single
title: "1장 오브젝트와 의존관계 - 1.4 제어의 역전(IoC)"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

## 1.4 제어의 역전(IoC)

### 1.4.1 오브젝트 팩토리

#### 팩토리

UserDaoTest 가 하던 특정 ConnectionMaker 를 만들고, UserDao 에 ConnectionMaker 을 연결하는 일을 대신할 UserDaoFactory 를 만든다.

```java
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

이제 UserDaoTest 에서는 UserDaoFactory 에서 나온 UserDao 객체를 사용하기만 하면 된다.

```java
public class UserDaoTest {

  public static void main(String[] args) throws ClassNotFoundException, SQLException {
    UserDao dao = new UserDaoFactory().userDao();
    // ...
  }

}
```

#### 설계도로서의 팩토리

분리된 UserDao 와 ConnectionMaker 는 어플리케이션의 구성요소로 데이터로직 및 기술로직을 담당하게 된다.

그리고 UserDaoFactory 는 이 오브젝트간의 구조와 관계를 정의하는 역할을 맡게 된다.

UserDaoFactory 를 만듦으로써 애플리케이션의 구성요소 역할의 오브젝트와 구조를 결정하는 역할의 오브젝트가 구분되게 되었다.

### 1.4.2 오브젝트 팩토리의 활용

만약 다른 Dao 가 추가된다면 DaoFactory 에 해당 Dao 용 ConnectionMaker 를 생성하는 코드가 중복된다.

```java
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

이런 경우는 ConnectionMaker 를 생성하는 코드를 추출하여 중복을 피할 수 있다.

```java
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
