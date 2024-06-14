---
# layout: single
title: "6장 AOP - 6.1 트랜잭션 코드의 분리"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

AOP 는 Ioc/DI, 서비스 추상화와 더불어 스프링의 3대 기반기술의 하나다.

스프링에 적용된 가장 인기 있는 AOP 의 적용 대상은 바로 선언적 트랜잭션 기능이다.

## 6.1 트랜잭션 코드의 분리

비즈니스 로직이 주인이어야 할 메소드 안에 이름도 길고 무시무시하게 생긴 트랜잭션 코드가 더 많은 자리를 차지하고 있는 모습이 못마땅하다.

트랜잭션의 경계는 분명 비즈니스 로직의 전후에 설정돼야 하는 것이 분명하니 UserService 의 메소드에 두는 것을 거부할 명분이 없다.

### 6.1.1 메소드 분리

트랜잭션이 적용된 코드의 특징은 비즈니스 로직 코드를 사이에 두고 트랜잭션 시작과 종료를 담당하는 코드가 앞뒤에 위치하고 있다는 점과 트랜잭션 경계설정의 코드와 비즈니스 로직 코드 간에 서로 주고 받는 정보가 없다는 점이다.

트랜잭션 코드와 비즈니스 코드는 성격이 다를 뿐 아니라 서로 주고받는 것도 없는, 완벽하게 독립적인 코드다.

트랜잭션 코드와 비즈니스 코드를 분리해보자

```java
public void upgradeLevels() throws Exception {
    Transactionstatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        upgradeLevelsInternal();
        this.transactionManager.commit(status);
    } catch (Exception e) {
        this.transactionManager.rollback(status);
        throw e;
    }
}

private void upgradeLevelsInternal() {
    List<User> users = userDao.getAll();
    for (User user : users) {
        if (canUpgradeLevel(user)) {
            upgradeLevel(user);
        }
    }
}
```

### 6.1.2 DI를 이용한 클래스의 분리

여전히 트랜잭션 코드가 UserService 안에 있다.

트랜잭션 코드를 UserService 밖으로 뽑아내 보자.

#### DI 적용을 이용한 트랜잭션 분리

하지만 트랜잭션 코드를 UserService 밖으로 빼버리면 UserService 를 직접 사용하는 클라이언트 코드에서는 트랜잭션 기능이 빠진 UserService 를 사용하게 된다.

이것은 구체적인 구현 클래스를 직접 참조하는 경우의 전형적인 단점이다.

DI 를 이용해서 간접적으로 사용하게 변경해본다.

DI 의 기본 아이디어는 실제 사용할 오브젝트의 클래스 정체는 감춘 채 인터페이스를 통해 간접적으로 접근하는 것이다.

현재 구조는 UserService 와 그것을 사용하는 클라이언트가 강한 결합도로 고정되어 있다.

이 경우 UserService 를 인터페이스로 만들고 기존 코드는 UserService 인터페이스의 구현 클래스 UserServiceImpl 을 만들었다.

지금 해결하려고 하는 문제는 UserService 에는 순수하게 비즈니스 로직을 담고, 트랜잭션 코드를 외부로 빼내려고 하는 것이다.

UserService 인터페이스를 구현한 또 다른 구현 클래스를 만든다.

이 클래스가 트랜잭션 경계설정이라는 책임을 맡는다.

그리고 비즈니스 코드를 가진 UserService 구현 클래스인 UserServiceImpl 에 실제적인 로직 처리 작업을 위임한다.

#### UserService 인터페이스 도입

```java
public interface UserService {
    void add(User user);
    void upgradeLevels();
}
```

UserServiceImpl 에서는 트랜잭션 코드를 제거한다.

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao;
    private UserLevelUpgradePolicy userLevelUpgradePolicy;

    // ...

    public void upgradeLevels() {
        List<User> users = userDao.getAll();
        for (User user : users) {
            if (userLevelUpgradePolicy.canUpgradeLevel(user)) {
                userLevelUpgradePolicy.upgradeLevel(user);
            }
        }
    }
}
```

#### 분리된 트랜잭션 기능

이제 트랜잭션 경계설정 기능을 담은 UserServiceTx 를 만들어보자.

UserServiceTx 는 UserService 를 구현한 다른 오브젝트에게 비즈니스 작업을 위임한다.

```java
public class UserServiceTx implements UserService {

    UserService userService;

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public void add(User user) {
        userService.add(user);
    }

    public void upgradeLevels() {
        userService.upgradeLevels();
    }

}
```

UserServiceTx 는 UserService 인터페이스를 구현했으니, 클라이언트에 대해 UserService 타입 오브젝트의 하나로 행세할 수 있다.

이제 트랜잭션 코드를 부여할 수 있다.

```java
public class UserServiceTx implements UserService {

    UserService userService;

    PlatformTransactionManager transactionManager;

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    // ...

    public void upgradeLevels() {
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            userService.upgradeLevels();
            this.transactionManager.commit(status);
        } catch (RuntimeException e) {
            this.transactionManager.rollback(status);
            throw e;
        }
    }

}
```

#### 트랜잭션 적용을 위한 DI 설정

기존의 userService 빈이 의존하고 있던 transactionManager 는 userServiceTx 가 의존하게 하고, userDao 와 mailSender 는 userServiceImpl 빈이 의존하도록 변경한다.

#### 트랜잭션 분리에 따른 테스트 수정

스프링의 설정파일에는 UserService 라는 인터페이스 타입을 가진 빈이 2개 존재한다.

같은 타입의 빈이 두 개라면 @Autowired 를 적용한 경우 어떤 빈을 가져올까? @Autowired 는 기본적으로 타입을 이용해 빈을 찾지만 만약 타입으로 하나의 빈을 결정할 수 없는 경우에는 필드 이름을 이용해 빈을 찾는다.

MailSender 를 DI 해줄 대상을 구체적으로 알고 있어야 하기 때문에 UserServiceImpl 클래스의 오브젝트를 가져올 필요가 있다.

목 오브젝트를 이용해 수동 DI 를 적용하는 테스트라면 어떤 클래스의 오브젝트인지 분명하게 알 필요가 있다.

UserServiceImpl 클래스 타입의 변수를 선언하고 @Autowired 를 지정해서 해당 클래스로 만들어진 빈을 주입받도록 한다.

upgradeLevels() 테스트 메소드의 MailSender 의 목 오브젝트를 userServiceImpl 빈에 설정해준다.

TestService 오브젝트를 UserServiceTx 오브젝트에 수동 DI 시킨 후 트랜잭션 기능까지 포함된 UserServiceTx 의 메소드를 호출해서 테스트를 수행한다.

```java
@Test
public void upgradeAllOrNothing() throws Exception {
    TestUserService testUSerService = new TestUserService(users.get(3).gretId());
    testUSerService.setUserDao(userDao);
    testUserService.setMailSender(mailSender);
    
    UserServiceTx userServiceTx = new UserServiceTx();
    userServiceTx.setTransactionManager(transactionManager);
    userServiceTx.setUserService(testUserService);
    
    userDao.deleteAll();
    for(User user : users) userDao.add(user);
    
    try {
        userServiceTx.upgradeLevels();
        fail("TestUserServiceException expected");
    }
    ...
}
```

TestUserService 는 이제 UserServiceImpl 클래스를 상속하면 된다.

#### 트랜잭션 경계설정 코드 분리의 장점








