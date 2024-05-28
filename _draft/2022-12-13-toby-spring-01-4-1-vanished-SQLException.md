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

기능이 비정상적으로 동작하거나, 메모리나 리소스가 소진되거나 예상치 못한 다른 문제를 일으킬 수 있다.

또한, 예외가 화면에 출력시키는 것만으로도 부족하다.

예외는 처리되어야 하기 때문에 catch 블록을 이용해 화면에 메시지를 출력한 것은 예외를 처리한 게 아니다.

예외를 처리할 때 반드시 지켜야 할 핵심 원칙은 한 가지다. 

모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보되어야 한다.

굳이 예외를 잡아서 뭔가 조치를 취할 방법이 없다면 잡지 말아야 한다.

메소드에 throws 선언해서 메소드 밖으로 던지고 자신을 호출한 코드에 예외처리 책임을 전가해버려라.

#### 무의미하고 무책임한 throws




**Info:**<br>
코드는 이늄으로 관리되어야 합니다.<br>
소스코드에서 코드를 문자나 숫자로 받게하면 엉뚱한 값이나 범위를 벗어나는 값을 넣게 될 수도 있습니다.
{: .notice--info}

#### User 필드 추가

User 클래스에도 Level 을 추가해 줍니다.

```java
public class User {
	
  // ...
	
  Level level; // 현재 레벨
  int login; // 로그인 횟수
  int recommend; // 추천 횟수
	
  public User(String id, String name, String password, Level level, int login, int recommend) {
    this.id = id;
    this.name = name;
    this.password = password;
    // highlight-start
    this.level = level;
    this.login = login;
    this.recommend = recommend;
    // highlight-end
  }

  // ...

  public Level getLevel() {
    return level;
  }

  public void setLevel(Level level) {
    this.level = level;
  }
	
  public int getLogin() {
    return login;
  }

  public void setLogin(int login) {
    this.login = login;
  }

  public int getRecommend() {
    return recommend;
  }

  public void setRecommend(int recommend) {
    this.recommend = recommend;
  }

}
```

#### UserDaoTest 테스트 수정

UserDaoTest.java 도 수정해 줍니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserDaoTest {

  @Autowired 
  UserDao dao; 
  
  @Autowired 
  DataSource dataSource;
	
  private User user1;
  private User user2;
  private User user3; 
	
  @Before
  public void setUp() {
    // highlight-start
    this.user1 = new User("gyumee", "박성철", "springno1", Level.BASIC, 1, 0);
    this.user2 = new User("leegw700", "이길원", "springno2", Level.SILVER, 55, 10);
    this.user3 = new User("bumjin", "박범진", "springno3", Level.GOLD, 100, 40);
    // highlight-end
  }
	
  // ...
	
  private void checkSameUser(User user1, User user2) {
    assertThat(user1.getId(), is(user2.getId()));
    assertThat(user1.getName(), is(user2.getName()));
    assertThat(user1.getPassword(), is(user2.getPassword()));
    // highlight-start
    assertThat(user1.getLevel(), is(user2.getLevel()));
    assertThat(user1.getLogin(), is(user2.getLogin()));
    assertThat(user1.getRecommend(), is(user2.getRecommend()));
    // highlight-end
  }
	
}
```

#### UserDaoJdbc 수정

UserDaoJdbc.java 도 수정합니다.

```java
private RowMapper<User> userMapper = new RowMapper<User>() {
  public User mapRow(ResultSet rs, int rowNum) throws SQLException {
    User user = new User();
    user.setId(rs.getString("id"));
    user.setName(rs.getString("name"));
    user.setPassword(rs.getString("password"));
    // highlight-start
    user.setLevel(Level.valueOf(rs.getInt("level")));
    user.setLogin(rs.getInt("login"));
    user.setRecommend(rs.getInt("recommend"));
    // highlight-end
    return user;
  }
};

public void add(User user) {
  this.jdbcTemplate.update("insert into users (id, name, password, level, login, recommend) values (?, ?, ?, ?, ?, ?)", 
    user.getId(), 
    user.getName(), 
    user.getPassword(),
    // highlight-start 
    user.getLevel().intValue(), 
    user.getLogin(), 
    user.getRecommend()
    // highlight-end
  );
}
```

JDBC 가 사용하는 SQL 은 컴파일 과정에서는 자동으로 검증되지 않는 문자열입니다.

그래서 SQL 문장이 실행되기 전까지는 문법 오류나 오타를 발견하기 어렵습니다.

만약 userMapper 에 추가한 내용에 오타가 있다면 컴파일 에러가 발생합니다.

### 5.1.2 사용자 수정 기능 추가

수정할 정보가 담긴 User 객체를 전달하면 id 를 참고해서 사용자를 찾아서 정보를 갱신하는 메소드를 만들어 봅니다.

#### 수정 기능 테스트 추가

update 테스트를 추가합니다.

```java
@Test
public void update() {
  dao.deleteAll();
  
  dao.add(user1);
  
  user1.setName("오민규");
  user1.setPassword("springno6");
  user1.setEmail("user6@ksug.org");
  user1.setLevel(Level.GOLD);
  user1.setLogin(1000);
  user1.setRecommend(999);
  
  // highlight-next-line
  dao.update(user1);
  
  User user1update = dao.get(user1.getId());
  checkSameUser(user1, user1update);
}
```

#### UserDao 와 UserDaoJdbc 수정

UserDao 에 update() 메소드를 추가합니다.

UserDao 를 구현한 UserDaoJdbc 에도 update() 메소드가 필요합니다.

```java
public void update(User user) {
  this.jdbcTemplate.update("update users set name = ?, password = ?, email = ?, level = ?, login = ?, recommend = ? where id = ? ", 
    user.getName(), 
    user.getPassword(), 
    user.getEmail(), 
    user.getLevel().intValue(), 
    user.getLogin(), 
    user.getRecommend(),
    user.getId()
  );
}
```

#### 수정 테스트 보완

SQL 문에서는 종종 테스트로는 검증하지 못하는 오류가 발생할 수도 있습니다.

예컨데  UPDATE 문에서 WHERE 절을 빼먹는 경우입니다.

테스트 코드를 보완하여 UPDATE 문의 실수를 발견할 수 있도록 해봅니다.

```java
@Test
public void update() {
  dao.deleteAll();
  
  dao.add(user1);		// 수정할 사용자
  dao.add(user2);		// 수정하지 않을 사용자
  
  user1.setName("오민규");
  user1.setPassword("springno6");
  user1.setEmail("user6@ksug.org");
  user1.setLevel(Level.GOLD);
  user1.setLogin(1000);
  user1.setRecommend(999);
  
  dao.update(user1);
  
  User user1update = dao.get(user1.getId());
  checkSameUser(user1, user1update);
  // highlight-start
  User user2same = dao.get(user2.getId());
  checkSameUser(user2, user2same);
  // highlight-end
}
```

사용자를 두 명 등록해놓고, 그 중 하나만 수정한 뒤에 수정된 사용자와 수정하지 않은 사용자의 정보를 확인하는 식으로 테스트를 보완합니다.

### 5.1.3 UserService.upgradeLevels()

이제 레벨을 변경하는 비즈니스 로직을 추가해 봅니다.

Dao 는 데이터를 다루는 영역이기 때문에 비즈니스 로직을 Dao 에 두는 것은 적당하지 않습니다.

사용자 관리 로직을 추가할 UserService 클래스를 생성합니다.

UserService 는 UserDao 인터페이스 타입으로 userDao 빈을 DI 받아 사용합니다.

#### UserService 클래스와 빈 등록

```java
public class UserService {

  // highlight-next-line
  private UserDao userDao;

  public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
  }
  
  // ...

}
```

그리고 UserService 를 빈으로 등록하기 위해 XML 에 추가해줍니다.

#### UserServiceTest 테스트 클래스

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserServiceTest {

  @Autowired UserService userService;	
	
  @Test
  public void bean() {
    // highlight-next-line
    assertThat(this.userService, is(notNullValue()));
  }
  
}
```

#### upgradeLevels() 메소드

이제 UserService 에 upgradeLevels() 메소드를 추가하고 레벨을 변경하는 비즈니스 로직을 추가합니다.

```java
public class UserService {
  
  // ...
	
  public void upgradeLevels() {
    List<User> users = userDao.getAll();  
    for(User user : users) {  
      Boolean changed = null;
      // highlight-next-line
      if (user.getLevel() == Level.BASIC && user.getLogin() >= 50) {
        user.setLevel(Level .SILVER);
        changed = true;
      // highlight-next-line
      } else if (user.getLevel() == Level.SILVER && user.getRecommend() >= 30) {
        user.setLevel(Level .GOLD);
        changed = true;
      // highlight-next-line
      } else if (user.getLevel() == Level.GOLD) {
        changed = false; 
      } else { 
        changed = false; 
      }
      
      if (changed) { 
        userDao.update(user); 
      }
    }
  }
  
}
```

#### upgradeLevels() 테스트

테스트 픽스처를 등록합니다.

```java
public class UserServiceTest {
  
  // ...
  
  @Before
  public void setUp() {
    users = Arrays.asList(
      new User("bumjin", "박범진", "p1", Level.BASIC, 49, 0),
      new User("joytouch", "강명성", "p2", Level.BASIC, 50, 0),
      new User("erwins", "신승한", "p3", Level.SILVER, 60, 29),
      new User("madnite1", "이상호", "p4", Level.SILVER, 60, 30),
      new User("green", "오민규", "p5", Level.GOLD, 100, 100)
    );
  }
  
}
```

테스트를 할 때는 데이터의 경계가 되는 값의 전후로 테스트 하는 것이 좋습니다.

여기서는 로그인 수 50 과 추천수 30 이 경계가 되는 값 입니다.

```java
@Test
public void upgradeLevels() {
  userDao.deleteAll();
  for(User user : users) userDao.add(user);
  
  // highlight-next-line
  userService.upgradeLevels();
  
  checkLevel(users.get(0), Level.BASIC);
  checkLevel(users.get(1), Level.SILVER);
  checkLevel(users.get(2), Level.SILVER);
  checkLevel(users.get(3), Level.GOLD);
  checkLevel(users.get(4), Level.GOLD);
}

// highlight-start
private void checkLevel(User user, boolean expectedLevel) {
  User userUpdate = userDao.get(user.getId());
  assertThat(userUpdate.getLevel(), is(expectedLevel));
}
// highlight-end
```

사용자 정보를 저장한 후 upgradeLevels() 메소드를 실행합니다.

그리고 checkLevel() 메소드로 기대하는 레벨로 변경되었는지 확인합니다.

### 5.1.4 UserService.add()

처음 가입하는 사용자는 기본적으로 BASIC 레벨이어야 합니다.

이 비즈니스 로직도 사용자 관리에 대한 비즈니스 로직을 담고 있는 UserService 에 담는 것이 좋습니다.

```java
@Test 
public void add() {
  userDao.deleteAll();
  
  User userWithLevel = users.get(4);	  // GOLD 레벨  
  User userWithoutLevel = users.get(0);  
  userWithoutLevel.setLevel(null);
  
  // highlight-start
  userService.add(userWithLevel);	  
  userService.add(userWithoutLevel);
  // highlight-end
  
  User userWithLevelRead = userDao.get(userWithLevel.getId());
  User userWithoutLevelRead = userDao.get(userWithoutLevel.getId());
  
  // highlight-start
  assertThat(userWithLevelRead.getLevel(), is(userWithLevel.getLevel())); 
  assertThat(userWithoutLevelRead.getLevel(), is(Level.BASIC));
  // highlight-end
}
```

위 테스트케이스는 2가지를 테스트합니다.

Level 이 비어있는 경우는 BASIC 을 부여해주고, 이미 설정된 Level 이 있다면 그대로 놔두는 것입니다.

이제 UserService 에 add() 메소드를 추가합니다.

```java
public void add(User user) {
  if (user.getLevel() == null) user.setLevel(Level.BASIC);
  userDao.add(user);
}
```

테스트는 통과합니다.

### 5.1.5 코드 개선

작성된 코드를 살펴볼 때는 다음과 같은 질문을 해볼 필요가 있습니다.

- 코드에 중복된 부분은 없는가?
- 코드가 무엇을 하는 것인지 이해하기 불편하지 않은가?
- 코드가 자신이 있어야 할 자리에 있는가?
- 앞으로 변경이 일어난다면 어떤 것이 있을 수 있고, 그 변화에 쉽게 대응할 수 있게 작성되어 있는가?

#### upgradeLevels() 메소드 코드의 문제점

upgradeLevels() 메소드를 다시 살펴봅니다.

```java
public class UserService {
  
  // ...
	
  public void upgradeLevels() {
    List<User> users = userDao.getAll();  
    for(User user : users) {  
      Boolean changed = null;
      if (user.getLevel() == Level.BASIC && user.getLogin() >= 50) {
        user.setLevel(Level .SILVER);
        changed = true;
      } else if (user.getLevel() == Level.SILVER && user.getRecommend() >= 30) {
        user.setLevel(Level .GOLD);
        changed = true;
      } else if (user.getLevel() == Level.GOLD) { 
        changed = false; 
      } else { 
        changed = false; 
      }
      
      if (changed) { 
        userDao.update(user); 
      }
    }
  }
  
}
```

upgradeLevels() 메소드에는 아래와 같은 문제점이 있습니다.

- 새로운 레벨이 추가되면 Level 이늄이 수정되어야 하고, upgradeLevels() 의 if 조건식이 추가되어야 한다.
- 업그레이드 조건이 복잡해질수록 메소드가 길어진다.
- 조건으로 기존 레벨을 확인하고 각 레벨별로 조건을 판단하는 식으로 코드가 복잡해 질 수 있다. (if 중첩)

#### upgradeLevels() 리팩토링

코드를 리팩토링을 하기 위해서 추상적인 레벨에서부터 로직을 작성해 봅니다.

기존의 upgradeLevels() 메소드는 자주 변경될 가능성이 있는 구체적인 내용이 추상적인 로직의 흐름과 함께 섞여 있습니다.

upgradeLevels() 메소드를 리팩토링하기 위해서 레벨을 업그레이드하는 기본 흐름을 만들어 봅니다.

**Info:**<br>
비즈니스 로직을 추상적인 것과 구체적인 것으로 나눌 수 있는 기준은 구현의 변경가능성입니다.
{: .notice--info}

```java
public void upgradeLevels() {
  List<User> users = userDao.getAll(); 
  // highlight-start
  for(User user : users) {
    if (canUpgradeLevel(user)) { 
      upgradeLevel(user);
    }
  }
  // highlight-end
}
```

추상적인 기본 흐름은 '사용자 정보를 가져와 한 명씩 순회하면서 업그레이드 가능여부를 확인하고, 가능하면 업그레이드 한다.' 입니다.

구체적인 내용은 각 메소드에서 구현합니다.

먼저 canUpgradeLevel() 메소드를 구현해봅니다.

여기서는 레벨별로 업그레이드 조건을 확인합니다.

```java
private boolean canUpgradeLevel(User user) {
  Level currentLevel = user.getLevel();
  switch(currentLevel) {
    case BASIC: return (user.getLogin() >= 50); 
    case SILVER: return (user.getRecommend() >= 30);
    case GOLD: return false;
    default: throw new IllegalArgumentException("Unknown Level: " + currentLevel); 
  }
}
```

만약 업그레이드 조건이 확인되면 upgradeLevel() 메소드로 업그레이드 작업을 진행합니다.

```java
public class UserService {

  // ...
  
  private void upgradeLevel(User user) {
    if (user.getLevel() == level.BASIC) user.setLevel(Level.SILVER);
    else if (user.getLevel() == level.SILBER) user.setLevel(Level.GOLD);
    userDao.update(user);
  }
}
```

upgradeLevel() 메소드는 약간 문제가 있습니다.

- 레벨간의 관계가 노골적으로 드러난다는 것
- 레벨이 늘어나면 if 문이 점점 길어진다는 것
- 예외상황에 대한 처리가 없다는 것

먼저 레벨간의 관계는 Level 이늄으로 이동시킵니다.

```java
public enum Level {

  // highlight-start
  GOLD(3, null), 
  SILVER(2, GOLD), 
  BASIC(1, SILVER); 
  // highlight-end 
	
  private final int value;
  // highlight-next-line
  private final Level next; 
	
  Level(int value, Level next) {  
    this.value = value;
    this.next = next; 
  }
	
  // ...
	
  public Level nextLevel() { 
    return this.next;
  }
	
  // ...
	
}
```

이렇게 하면 UserService 가 비즈니스 로직에서 일일이 조건식으로 다음 레벨을 지정할 필요가 없습니다.

사용자의 레벨이 바뀌는 로직도 UserService 보다 User 에서 처리하도록 합니다.

**Info:**<br>
객체의 내부 정보가 변경되는 것은 객체 스스로 다루는 것이 적절합니다.
{: .notice--info}

```java
public class User {

  // ...
  
  // highlight-start
  public void upgradeLevel() {
    Level nextLevel = this.level.nextLevel();	
    if (nextLevel == null) { 								
      throw new IllegalStateException(this.level + "은  업그레이드가 불가능합니다");
    } else {
      this.level = nextLevel;
    }
  }
  // highlight-end
  	
}
```

upgradeLevel() 메소드를 잘못 사용하는 코드가 있을 수 있으니 예외처리도 포함하도록 합니다.

리팩토링 후 UserService 는 다음과 같이 변경됩니다.

```java
public class UserService {

  // ...
  
  public void upgradeLevels() {
    List<User> users = userDao.getAll(); 
    for(User user : users) {
      if (canUpgradeLevel(user)) { 
        upgradeLevel(user);
      }
    }
  }
  
  private boolean canUpgradeLevel(User user) {
    Level currentLevel = user.getLevel();
    switch(currentLevel) {
      case BASIC: return (user.getLogin() >= 50); 
      case SILVER: return (user.getRecommend() >= 30);
      case GOLD: return false;
      default: throw new IllegalArgumentException("Unknown Level: " + currentLevel); 
    }
  }
  
  private void upgradeLevel(User user) {
    // highlight-next-line
    user.upgradeLevel();
    userDao.update(user);
  }

}
```

이제 UserService, User, Level 이 각자의 내부 정보를 다루는 자신의 책임에 충실한 기능을 가지고 있으면서 필요한 일이 생기면 수행을 요청하는 구조를 가지고 있습니다.

코드 간결하기 때문에 변경이 필요할 때 수정할 지점을 쉽게 찾을 수 있습니다.

독립적으로 테스트하도록 만든다면 테스트 코드도 단순해 집니다.

**Info:**<br>
객체지향적 코드는 다른 객체의 데이터를 가져와서 작업하지 않고 그 객체에 작업을 해달라고 요청합니다.<br>
오브젝트에게 데이터를 요구하지 말고 작업을 요청하라는 것이 객체지향 프로그래밍의 가장 기본이 되는 원리입니다.
{: .notice--info}

#### User 테스트

User 에 대한 테스트도 만들어 봅니다.

앞으로 계속 새로운 기능과 로직이 추가될 가능성이 있으니 테스트를 만들어두면 도움이 될 것 입니다.

```java
public class UserTest {

  User user;
	
  @Before
  public void setUp() {
    user = new User();
  }
	
  @Test
  public void upgradeLevel() {
    Level[] levels = Level.values();
    for(Level level : levels) {
      if (level.nextLevel() == null) continue;
      user.setLevel(level);
      user.upgradeLevel();
      assertThat(user.getLevel(), is(level.nextLevel()));
    }
  }
	
  @Test(expected=IllegalStateException.class)
  public void cannotUpgradeLevel() {
    Level[] levels = Level.values();
    for(Level level : levels) {
      if (level.nextLevel() != null) continue;
      user.setLevel(level);
      user.upgradeLevel();
    }
  }

}
```

#### UserServiceTest 개선

기존의 테스트 코드에서는 무엇을 테스트하는지 잘 보이지 않았던 문제가 있었기에 조금 수정을 합니다.

```java
@Test
public void upgradeLevels() {
  userDao.deleteAll();
  for(User user : users) userDao.add(user);
  
  userService.upgradeLevels();
  
  // highlight-start
  checkLevelUpgraded(users.get(0), false);
  checkLevelUpgraded(users.get(1), true);
  checkLevelUpgraded(users.get(2), false);
  checkLevelUpgraded(users.get(3), true);
  checkLevelUpgraded(users.get(4), false);
  // highlight-end
}

private void checkLevelUpgraded(User user, boolean upgraded) {
  User userUpdate = userDao.get(user.getId());
  if (upgraded) {
    // highlight-next-line
    assertThat(userUpdate.getLevel(), is(user.getLevel().nextLevel()));
  } else {
    // highlight-next-line
    assertThat(userUpdate.getLevel(), is(user.getLevel()));
  }
}
```

기존의 checkLevel() 메소드 호출 시 파라미터로 전달하는 Level 이늄은 어떻게 테스트하는 것인지 알 수가 없었습니다.

반면에 checkLevelUpgraded() 메소드의 true/false 는 레벨 업그레이드 여부를 확인하려는 의도가 드러납니다.

고정된 값도 상수로 변경해 줍니다.

숫자의 의미를 파악하기 쉽게 하기 위해서이고, 또한 여러 곳의 값을 한번에 수정할 수도 있습니다.

```java
public class UserService {
  // highlight-start
  public static final int MIN_LOGCOUNT_FOR_SILVER = 50;
  public static final int MIN_RECCOMEND_FOR_GOLD = 30;
	// highlight-end

  // ...

  private boolean canUpgradeLevel(User user) {
    Level currentLevel = user.getLevel(); 
    switch(currentLevel) {
      // highlight-start
      case BASIC: return (user.getLogin() >= MIN_LOGCOUNT_FOR_SILVER); 
      case SILVER: return (user.getRecommend() >= MIN_RECCOMEND_FOR_GOLD);
      // highlight-end
      case GOLD: return false;
      default: throw new IllegalArgumentException("Unknown Level: " + currentLevel); 
    }
  }

  // ...

}
```

UserServiceTest 도 변경합니다.

```java
// ...

// highlight-start
import static springbook.user.service.UserService.MIN_LOGCOUNT_FOR_SILVER;
import static springbook.user.service.UserService.MIN_RECCOMEND_FOR_GOLD;
// highlight-end

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserServiceTest {
  
  // ...
	
  @Before
  public void setUp() {
    users = Arrays.asList(
      new User("bumjin", "박범진", "p1", Level.BASIC, MIN_LOGCOUNT_FOR_SILVER-1, 0),
      new User("joytouch", "강명성", "p2", Level.BASIC, MIN_LOGCOUNT_FOR_SILVER, 0),
      new User("erwins", "신승한", "p3", Level.SILVER, 60, MIN_RECCOMEND_FOR_GOLD-1),
      new User("madnite1", "이상호", "p4", Level.SILVER, 60, MIN_RECCOMEND_FOR_GOLD),
      new User("green", "오민규", "p5", Level.GOLD, 100, Integer.MAX_VALUE)
    );
  }

  // ...
	
}
```

만약 레벨을 업그레이드 하는 정책을 유연하게 변경할 수 있도록 하고 싶다면 업그레이드 정책을 UserService 에서 분리하는 방법을 고려할 수 있습니다.

UserLevelUpgradePolicy 인터페이스를 만들고 그 구현 클래스를 UserService 에 주입하도록 만드는 방법입니다.
