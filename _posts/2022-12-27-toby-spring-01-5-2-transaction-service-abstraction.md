---
# layout: single
title: "5장 서비스 추상화 - 5.2 트랜잭션 서비스 추상화"
# excerpt: "DAO 에 트랜잭션을 적용해보면서 스프링이 어떻게 성격이 비슷한 여러 기술을 추상화하고, 일관된 방법으로 사용할 수 있도록 지원하는지 알아봅니다."
categories:
  - "토비의 스프링 3.1"
tags:
  - "서비스 추상화"
---

## 5.2 트랜잭션 서비스 추상화

사용자 레벨 조정 작업 중간에 문제가 발생하면 그때까지 진행되었던 모든 변경 작업을 모두 취소시키도록 하는 기능을 추가합니다.

### 5.2.1 모 아니면 도

작업 수행 중 예외가 던져지는 상황을 의도적으로 만들어서 테스트해 봅니다.

#### 테스트용 UserService 대역

테스트를 위해 UserService 의 서브클래스를 만듭니다.

테스트용 UserService 서브클래스는 UserService 의 일부 메소드를 특정 시점에 강제로 예외를 발생하도록 구현합니다.

테스트용이기 때문에 테스트클래스 내부에서 static 클래스로 만듭니다.

테스트하려는 UserService 의 upgradeLevel() 메소드의 접근권한을 'protected' 로 변경하고, 서브클래스에서 오버라이딩 합니다.

```java
public class UserServiceTest {

  // ...
  
  // highlight-next-line
  static class TestUserService extends UserService {
    private String id;
    
    private TestUserService(String id) {  
      this.id = id;
    }
  
    // highlight-next-line
    protected void upgradeLevel(User user) {
      if (user.getId().equals(this.id)) throw new TestUserServiceException();  
      super.upgradeLevel(user);  
    }
  }
  
  static class TestUserServiceException extends RuntimeException {
  }
}
```

#### 강제 예외 발생을 통한 테스트

UserServiceTest 에 테스트케이스를 만듭니다.

fail() 메소드는 만약 upgradeLevels() 메소드가 정상종료되면 강제로 실패하게 만드는 메소드입니다.

```java
import static org.junit.Assert.fail;

// ...

public class UserServiceTest {

  // ...
  
  @Test
  public void upgradeAllOrNothing() throws Exception {
    // highlight-next-line
    UserService testUserService = new TestUserService(users.get(3).getId());  
    testUserService.setUserDao(this.userDao); 
    testUserService.setDataSource(this.dataSource);
     
    userDao.deleteAll();			  
    for(User user : users) userDao.add(user);
    
    try {
      // highlight-next-line
      testUserService.upgradeLevels();   
      fail("TestUserServiceException expected"); 
    } catch(TestUserServiceException e) { 
    }
    
    // 기존 User 의 등급이 변경되었는지 확인
    checkLevelUpgraded(users.get(1), false);
  }
}
```

하지만 이 테스트는 실패합니다.

#### 테스트 실패의 원인

upgradeLevels() 메소드가 아직 트랜잭션으로 처리되지 않았기 때문에 중간에 예외가 발생하더라도 기존의 변경이 취소되지 않습니다.

### 5.2.2 트랜잭션 경계설정

하나의 SQL 명령은 DB 자체에서 트랜잭션을 보장합니다.

하지만 이번 요구사항처럼 여러 SQL 명령을 하나의 트랜잭션으로 묶어야 하는 상황이 있습니다.

#### JDBC 트랜잭션의 트랜잭션 경계설정

트랜잭션은 시작하는 지점과 끝나는 지점이 있습니다.

시작하는 방법은 한 가지이지만 끝내는 방법은 무효화하는 Transaction Rollback, 확정하는 Transaction Commit 이 있습니다.

트랜잭션이 시작하고 끝나는 지점을 트랜잭션 경계라고 합니다.

트랜잭션의 시작과 종료는 Connection 오브젝트를 통해 이루어 집니다.

JDBC 의 기본설정은 DB 작업 수행이 완료되면 트랜잭션 커밋을 하도록 되어 있습니다.

setAutoCommit(false) 으로 시작하고, commit(), rollback() 을 선언하여 수동으로 처리하는 작업을 트랜잭션의 경계설정 transaction demarcation 이라고 합니다.

#### UserService 와 UserDao의 트랜잭션 문제

일반적으로 트랜잭션은 Connection 보다 수명이 짧을 수 밖에 없습니다.

UserDaoJdbc 오브젝트는 JdbcTemplate 을 사용하고, JdbcTemplate 메소드(ex. update())들은 DataSource 의 getConnection() 메소드를 호출하여 작업하고, 작업이 끝나면 Connection 을 닫아줍니다.

결국 JdbcTemplate 메소드 호출 한번에 하나의 Connection 을 만들고 닫기 때문에, 각 메소드들은 독립적인 트랜잭션으로 실행될 수 밖에 없습니다.

#### 비즈니스 로직 내의 트랜잭션 경계설정

여러 SQL 을 한번에 묶기 위해서는 UserService 에서 트랜잭션의 경계설정 작업을 해야합니다.

UserService 에서 Connection 객체를 생성하고, Connection 객체를 upgradeLevel() 메소드나 UserDaoJdbc 객체의 메소드를 통해 전달해줘야 합니다.

#### UserService 트랜잭션 경계설정의 문제점

UserService 와 UserDao 에 Connection 객체를 생성하고, 전달받아 처리하는 식의 수정은 다른 문제를 발생시킵니다.

1. JdbcTemplate 사용이 불가하고 JDBC API 을 직접 사용해야 함
2. Connection 객체가 UserService 와 UserDao 에 파라미터로 추가되어야 하고 메소드마다 계속 전달
3. Connection 파라미터가 UserDao 인터페이스 메소드에 추가되면 UserDao 는 더 이상 데이터 액세스 기술에 독립적일 수가 없음
4. 테스트 코드에서 직접 Connection 오브젝트를 일일이 만들어서 DAO 메소드를 호출하도록 모두 변경

### 5.2.3 트랜잭션 동기화

#### Connection 파라미터 제거

upgradeLevels() 메소드가 트랜잭션 경계설정을 해야하는 상황에서 Connection 을 파라미터로 전달하는 문제를 해결해봅니다.

스프링에서는 트랜잭션 동기화 transaction synchronization 방식을 제안합니다.

UserService 에서 만든 Connection 객체를 특별한(?) 저장소에 보관해두고 필요할 때 사용하는 방식입니다.

트랜잭션 동기화를 사용한 경우의 작업 흐름을 살펴봅니다.

[![서비스 추상화-3](/assets/images/posts/2022-12-27-toby-spring-01-5-2-transaction-service-abstraction/05-3.png)](/assets/images/posts/2022-12-27-toby-spring-01-5-2-transaction-service-abstraction/05-3.png)

1. UserService 가 Connection 을 생성한다.
2. 트랜잭션 동기화 저장소에 저장하고, Connection 의 setAutoCommit(false) 를 호출해 트랜잭션을 시작한다.
3. 첫 번째 update() 메소드가 호출되면 update() 메소드 내부에서 JdbcTemplate 의 update() 메소드가 작동된다.
4. JdbcTemplate 의 update() 메소드는 트랜잭션 동기화 저장소에 현재 시작된 트랜잭션을 가진 Connection 객체가 있는지 확인한다.
5. Connection 객체를 이용해서 PreparedStatement 를 만들어 SQL 을 실행한다. JdbcTemplate 는 Connection 객체를 트랜잭션 동기화 저장소에서 가져온 경우에는 Connection 을 닫지 않고 작업을 마친다.
6. 두 번째 update() 메소드가 호출된다.
7. 트랜잭션 동기화 저장소에 Connection 을 가져온다.
8. JdbcTemplate 는 Connection 객체를 사용해서 SQL 을 실행한다.
9. 마지막 update() 메소드가 호출된다.
10. 트랜잭션 동기화 저장소에 Connection 을 가져온다.
11. JdbcTemplate 는 Connection 객체를 사용해서 SQL 을 실행한다.
12. UserService 는 트랜잭션 내의 모든 작업이 끝나면 Connection 의 commit() 을 호출해서 트랜잭션을 완료시킨다.
13. UserService 는 트랜잭션이 완료되면 트랜잭션 저장소에서 Connection() 객체를 삭제한다.

트랜잭션 동기화 저장소는 작업 스레드마다 독립적으로 Connection 객체를 저장하고 관리하기 때문에 멀티스레드 환경에서도 충돌이 날 염려는 없습니다.

트랜잭션 동기화 기법을 사용하면 파라미터를 통해 일일이 Connection 객체를 전달할 필요가 없습니다.

#### 트랜잭션 동기화 적용

멀티스레드 환경에서 안전한 트랜잭션 동기화 방법을 구현하는 일은 간단하지 않습니다.

다행히 스프링에서는 트랜잭션 동기화 기능을 지원하는 유틸리티 메소드를 제공합니다.

```java
// ...

import org.springframework.jdbc.datasource.DataSourceUtils;
import org.springframework.transaction.support.TransactionSynchronizationManager;

public class UserService {

  // highlight-next-line
  private DataSource dataSource;
  
  public void setDataSource(DataSource dataSource) {
    this.dataSource = dataSource;
  }
	
  public void upgradeLevels() throws Exception {
	  // highlight-start
    TransactionSynchronizationManager.initSynchronization();  
    Connection c = DataSourceUtils.getConnection(dataSource); 
    c.setAutoCommit(false);
		// highlight-end
		
    try {									   
      List<User> users = userDao.getAll();
      for (User user : users) {
        if (canUpgradeLevel(user)) {
          upgradeLevel(user);
        }
      }
      // highlight-next-line
      c.commit();
    } catch (Exception e) {
      // highlight-next-line    
      c.rollback();
      throw e;
    } finally {
      DataSourceUtils.releaseConnection(c, dataSource);
      // highlight-start	
      TransactionSynchronizationManager.unbindResource(this.dataSource);  
      TransactionSynchronizationManager.clearSynchronization();
			// highlight-end  
    }
  }
	
  // ...
	
}
```

먼저 Connection 을 생성할 때 사용할 DataSource 를 DI 받습니다.

스프링이 제공하는 트랜잭션 동기화 관리 클래스는 TransactionSynchronizationManager 입니다.

먼저 트랜잭션 동기화 작업을 초기화하고, DataSourUtils 를 통해 DB 커넥션을 생성합니다.

DataSourceUtils 의 getConnection() 메소드를 사용하는 이유는 Connection 객체를 생성해주기도 하지만 트랜잭션 동기화에 사용하도록 저장소에 바인딩해주기 때문입니다.

#### 트랜잭션 테스트 보완

UserServiceTest 의 upgradeAllOrNothing() 테스트 메소드에 dataSource 빈을 주입해줍니다.

```java

// ...

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserServiceTest {

  // highlight-start
  @Autowired 
  DataSource dataSource;
  // highlight-end

  @Test
  public void upgradeAllOrNothing() throws Exception {
    UserService testUserService = new TestUserService(users.get(3).getId());  
    testUserService.setUserDao(this.userDao); 
    // highlight-next-line
    testUserService.setDataSource(this.dataSource);
     
    userDao.deleteAll();			  
    for(User user : users) userDao.add(user);
    
    try {
      testUserService.upgradeLevels();   
      fail("TestUserServiceException expected"); 
    } catch(TestUserServiceException e) { 
    }
    
    checkLevelUpgraded(users.get(1), false);
  }
  
}
```

이제 레벨 업그레이드 작업에 트랜잭션이 적용됩니다.

사용자의 레벨 업그레이드 작업을 완료하지 못하면 이미 변경된 사용자의 레벨도 롤백됩니다.

#### JdbcTemplate 과 트랜잭션 동기화

JdbcTemplate 는 영리하게 동작합니다.

JdbcTemplate 는 트랜잭션 동기화 저장소에 등록된 Connection 객체가 없는 경우에 직접 DB 커넥션을 만들고 트랜잭션을 시작해서 JDBC 작업을 진행합니다.

반면 upgradeLevels() 메소드처럼 트랜잭션 동기화를 시작해놓았다면 트랜잭션 동기화 저장소에 들어 있는 DB 커넥션을 가져와서 기존의 트랙잭션에 참여합니다.

### 5.2.4 트랜잭션 서비스 추상화

#### 기술과 환경에 종속되는 트랜잭션 경계설정 코드

여러 개의 DB 를 사용하고 있는 환경에서는 JDBC Connection 을 이용한 트랜잭션 방식인 로컬 트랜잭션으로는 불가능합니다.

로컬 트랜잭션은 하나의 DB Connection 에 종속되기 때문입니다.

별도의 트랜잭션 관리자를 통해 트랜잭션을 관리하는 글로벌 트랜잭션 Global Transaction 방식을 사용해야 합니다.

자바는 글로벌 트랜잭션을 지원하는 트랜잭션 매니저를 지원하기 위한 API 인 JTA Java Transaction API 를 제공하고 있습니다.

아래는 JTA 를 이용한 트랜잭션 처리 코드의 전형적인 구조를 보여줍니다.

```java
InitialContext ctx = new InitialContext();
UserTransaction tx = (UserTransaction) ctx.lookup(USER_TX_JNDI_NAME);

tx.begin();
// highlight-next-line
Connection c = dataSource.getConnection();

try {
  tx.commit();
} catch (Exception e) {
  tx.rollback();
  throw.e;
} finally {
  c.close();
}
```

문제는 JDBC 로컬 트랜잭션을 JTA 를 이용하는 글로벌 트랜잭션으로 변경하려면 UserService 의 코드를 수정해야 한다는 부분입니다.

UserService 의 입장에서는 자신의 로직말고 기술환경이 바뀌었는데 코드가 변경이 되어 버립니다.

#### 트랜잭션 API의 의존관계 문제와 해결책

원래 UserService 는 UserDao 인터페이스에만 의존하고 있었기 때문에 구현 클래스의 변경에 영향을 받지 않았습니다.

하지만 UserService 에서 트랜잭션의 경계설정을 해야 할 필요가 생기면서 다시 특정 데이터 액세스 기술에 종속되는 구조가 되었습니다.

다행히 트랜잭션의 경계설정을 담당하는 코드는 일정한 패턴을 갖는 구조이기 때문에 추상화가 가능합니다.

<div class="notice--info" markdown="1">
추상화란 하위 시스템의 공통점을 뽑아내서 분리시키는 것을 말합니다.<br>
추상화를 하면 하위 시스템이 어떤 것인지 알지 못해도 일관된 방법으로 접근할 수 있습니다.<br>
</div>

각 DB 에서 제공하는 클라이언트 라이브러리와 API 는 서로 전혀 호환되지 않지만 SQL 을 이용한다는 공통의 방식이 있습니다.

이 공통 방식을 추상화 한 것이 JDBC 입니다.

이 추상화 방식을 응용하면 트랜잭션 처리 코드에도 추상화를 도입하여 트랜잭션 경계설정 코드를 만들 수 있습니다.

#### 스프링의 트랜잭션 서비스 추상화

스프링이 제공하는 트랜잭션 추상화 계층구조입니다.

[![서비스 추상화-6](/assets/images/posts/2022-12-27-toby-spring-01-5-2-transaction-service-abstraction/05-6.png)](/assets/images/posts/2022-12-27-toby-spring-01-5-2-transaction-service-abstraction/05-6.png)

스프링이 제공하는 트랜잭션 추상화 방법을 UserService 에 적용해봅니다.

```java
public void upgradeLevels() {
  // highlight-start
  PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
  TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
  // highlight-end
  
  try {
    List<User> users = userDao.getAll();
    for (User user : users) {
      if (canUpgradeLevel(user)) {
        upgradeLevel(user);
      }
    }
    // highlight-next-line
    this.transactionManager.commit(status);
  } catch (RuntimeException e) {
    // highlight-next-line
    this.transactionManager.rollback(status);
    throw e;
  }
}
```

JDBC 를 이용할 때는 먼저 Connection 을 생성하고 트랜잭션을 시작했습니다.

PlatformTransactionManager 에서는 getTransaction() 메소드를 호출하면 됩니다.

시작된 트랜잭션은 TransactionStatus 타입의 변수에 저장되어 있다가 PlatformTransactionManager 의 commit(), rollback() 시에 파라미터로 전달됩니다.

테스트를 통해 트랜잭션이 정상적으로 동작함을 알 수 있습니다.

#### 트랜잭션 기술 설정의 분리

UserService 코드를 JTA 를 이용하는 글로벌 트랜잭션으로 변경하려면 DataSourceTransactionManager 구현 클래스를 JPATransactionManager 로 바꿔주면 됩니다.

모두 PlatformTransactionManager 인터페이스를 구현하였기 때문에 트랜잭션 경계설정을 위한 getTransaction(), commit(), rollback() 메소드를 수정할 필요가 없습니다.

하지만 트랜잭션 구현 클래스에 대해 UserService 가 알고 있는 것은 DI 원칙 위배입니다.

따라서 UserService 가 트랜잭션 매니저를 DI 를 통해 주입받을 수 있도록 합니다.

<div class="notice--info" markdown="1">
어떤 클래스든 스프링의 빈으로 등록할 때 먼저 검토해야 할 것은 싱글톤으로 만들어져 여러 스레드에서 동시에 사용해도 괜찮은가 하는 점입니다.<br>
상태를 갖고 있는 등, 멀티스레드 환경에서 안전하지 않은 클래스가 빈으로 등록되어서는 안됩니다.<br>
</div>

```java
public class UserService {

  // ...
	
  private PlatformTransactionManager transactionManager;
	
  // highlight-start
  public void setTransactionManager(PlatformTransactionManager transactionManager) {
    this.transactionManager = transactionManager;
  }
  // highlight-end

  public void upgradeLevels() {
    // highlight-next-line
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
      List<User> users = userDao.getAll();
      for (User user : users) {
        if (canUpgradeLevel(user)) {
          upgradeLevel(user);
        }
      }
      this.transactionManager.commit(status);
    } catch (RuntimeException e) {
      this.transactionManager.rollback(status);
      throw e;
    }
  }
	
}
```

테스트의 upgradeAllOrNothing() 메소드는 수정이 필요합니다.

스프링 컨테이너로부터 transactionManager 빈을 주입받아서 전달해주도록 변경합니다.

```java
public class UserServiceTest {

    // ...
    
    // highlight-start
    @Autowired
    PlatformTransactionManager transactionManager;
    // highlight-end
  
    @Test
    public void upgradeAllOrNothing() {
        UserService testUserService = new TestUserService(users.get(3).getId());  
        testUserService.setUserDao(this.userDao);
        // highlight-next-line
        testUserService.setTransactionManager(this.transactionManager);
         
        userDao.deleteAll();			  
        for(User user : users) userDao.add(user);
        
        try {
            testUserService.upgradeLevels();   
            fail("TestUserServiceException expected"); 
        }
        catch(TestUserServiceException e) { 
        }
        
        checkLevelUpgraded(users.get(1), false);
    }
```
