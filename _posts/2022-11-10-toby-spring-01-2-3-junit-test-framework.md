---
layout: single
title: "2장 테스트 - 2.3 개발자를 위한 테스팅 프레임워크 JUnit"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

# 2장 테스트

## 2.3 개발자를 위한 테스팅 프레임워크 JUnit

JUnit 프레임워크는 폭넓게 사용되는 사실상 자바의 표준 테스팅 프레임워크이다.

### 2.3.1 JUnit 테스트 실행 방법

가장 좋은 JUnit 테스트 실행 방법은 IDE 에 내장된 JUnit 테스트 지원 도구를 사용하는 것이다.

이클립스의 Run As -> JUnit Test 를 사용하면 main() 메소드에서 JUnitCore 를 실행하는 것을 대체한다.

선택한 테스트를 바로 실행하는 단축키

- Eclipse + Window: Alt + Shift + X, T
- IntelliJ + Mac: Control + Shift + R

### 2.3.2 테스트 결과의 일관성

테스트는 외부 상태에 따라 성공하기도 하고 실패하기도 해서는 안된다.

코드의 변경이 없다면 테스트는 항상 동일한 결과를 내야 한다.

UserDao 에 deleteAll(), getCount() 메소드를 추가한다.

```java
public class UserDao {

  // ...
  
  public void deleteAll() throws SQLException {
    Connection c = dataSource.getConnection();
  
    PreparedStatement ps = c.prepareStatement("delete from users");
    ps.executeUpdate();

    ps.close();
    c.close();
  }
  
  public int getCount() throws SQLException  {
    Connection c = dataSource.getConnection();
  
    PreparedStatement ps = c.prepareStatement("select count(*) from users");

    ResultSet rs = ps.executeQuery();
    rs.next();
    int count = rs.getInt(1);

    rs.close();
    ps.close();
    c.close();
  
    return count;
  }

}
```

deleteAll() 메소드를 실행하고, getCount() 메소드를 이용해서 테스트에 맞는 상태가 되었는지 확인한다.

```java
public class UserDaoTest {
  
  @Test 
  public void andAndGet() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
    UserDao dao = context.getBean("userDao", UserDao.class);
    
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));
    
    User user = new User();
    user.setId("gyumee");
    user.setName("박성철");
    user.setPassword("springno1");

    dao.add(user);
    assertThat(dao.getCount(), is(1));
    
    User user2 = dao.get(user.getId());
    
    assertThat(user2.getName(), is(user.getName()));
    assertThat(user2.getPassword(), is(user.getPassword()));
  }
}
```

이제는 테스트를 반복해서 실행해도 계속 성공할 것 이다.

단위 테스트는 항상 일관성 있는 결과가 보장되어야 한다.

외부 환경에 영향을 받지 말아야하고, 테스트를 실행하는 순서가 바뀌어도 결과가 보장되어야 한다.

### 2.3.3 포괄적인 테스트

getCount() 메소드의 기능을 테스트하기 위한 테스트 메소드를 만들어본다.

테스트 메소드는 한 번에 한 가지 검증 목적에만 충실한 것이 좋다.

User 클래스에 생성자를 추가한다.

```java
public class User {
  
  // ...
  
  public User(String id, String name, String password) {
    this.id = id;
    this.name = name;
    this.password = password;
  }
  
}
```

UserDaoTest 에 count() 테스트 메소드를 추가한다.

```java
public class UserDaoTest {
  
  // ...
  
  @Test
  public void count() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
    
    UserDao dao = context.getBean("userDao", UserDao.class);
    User user1 = new User("gyumee", "박성철", "springno1");
    User user2 = new User("leegw700", "이길원", "springno2");
    User user3 = new User("bumjin", "박범진", "springno3");
        
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));
        
    dao.add(user1);
    assertThat(dao.getCount(), is(1));
    
    dao.add(user2);
    assertThat(dao.getCount(), is(2));
    
    dao.add(user3);
    assertThat(dao.getCount(), is(3));
  }

}
```

addAndGet() 메소드도 주어진 id 가 정확한 User 정보를 가져오는지 확인해야 한다.

addAndGet() 메소드를 보완한다.

```java
public class UserDaoTest {

  @Test 
  public void andAndGet() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
    UserDao dao = context.getBean("userDao", UserDao.class);

    User user1 = new User("gyumee", "박성철", "springno1");
    User user2 = new User("leegw700", "이길원", "springno2");
    
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    dao.add(user1);
    dao.add(user2);
    assertThat(dao.getCount(), is(2));
    
    User userget1 = dao.get(user1.getId());
    assertThat(userget1.getName(), is(user1.getName()));
    assertThat(userget1.getPassword(), is(user1.getPassword()));
    
    User userget2 = dao.get(user2.getId());
    assertThat(userget2.getName(), is(user2.getName()));
    assertThat(userget2.getPassword(), is(user2.getPassword()));
  }
  
  // ...
  
}
```

예외의 경우에 대해서도 테스트 할 수 있다.

```java
public class UserDaoTest {

  // ...  
  
  @Test(expected=EmptyResultDataAccessException.class)
  public void getUserFailure() throws SQLException {
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));
    
    dao.get("unknown_id");
  }

}
```

위의 테스트를 통과시키기 위해 소스코드를 수정하였다.

```java
public class UserDao {

  // ...
  
  public User get(String id) throws SQLException {
    Connection c = this.dataSource.getConnection();
    PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
    ps.setString(1, id);
    
    ResultSet rs = ps.executeQuery();

    User user = null;
    if (rs.next()) {
      user = new User();
      user.setId(rs.getString("id"));
      user.setName(rs.getString("name"));
      user.setPassword(rs.getString("password"));
    }

    rs.close();
    ps.close();
    c.close();
    
    if (user == null) throw new EmptyResultDataAccessException(1);

    return user;
  }
}
```

단순하고 간단한 테스트가 치명적인 실수를 피할 수 있게 해주기도 한다.

개발자가 테스트를 직접 만들 때 하는 실수가 바로 성공하는 테스트만 골라서 만드는 것이다.

스프링의 창시자인 로드 존슨은 "항상 네거티브 테스트를 먼저 만들라"는 조언을 했다.

### 2.3.4 테스트가 이끄는 개발

get() 메소드의 예외 케이스 테스트를 만드는 과정을 돌아보면 테스트 케이스를 먼저 만들고 이후 UserDao 를 수정하기 시작했다.

우리는 먼저 '존재하지 않는 id로 get() 메소드를 실행하면 특정한 예외가 던져져야 한다' 라는 기능을 먼저 설계하였다.

추가하고 싶은 기능을 코드로 표현하로 했다.

getUserFailure() 테스트 코드의 내용은 아래와 같다.

| - | 단계 | 내용 | 코드 |
|---|-----|-----|-----|
| 조건 | 어떤 조건을 가지고 | 가져올 사용자 정보가 존재하지 않는 경우에 |dao.deleteAII( ); <br/>assertThat(dao.getCount(), is(0));|
| 행위 | 무엇을 할 때 | 존재하지 않는 id로 get() 를 실행하면 |get("unknownjd");|
| 결과 | 어떤 결과가 나온다 | 특별한 예외가 던져진다. |@Test(expected= EmptyResultDataAccessException.class)|

테스트 코드가 마치 하나의 기능정의서처럼 보인다.

만약 테스트가 실패하면 이때는 설계한 대로 코드가 만들어지지 않았음을 바로 알 수 있다.

결국 테스트가 성공한다면, 코드의 구현과 테스트가 동시에 완료되게 된다.

테스트 코드를 먼저 만드록, 테스트를 성공하게 해주는 코드를 작성하는 방식을 테스트 주도 개발 TDD, Test Driven Development 라고 한다.

테스트를 먼저 만들고 그 테스트가 성공하도록 코드를 만들면 테스트를 빼먹지 않을 수 있다.

테스트를 작성하는 시간과 애플리케이션 코드를 작성하는 시간의 간격이 짧아진다.

TDD 에서는 테스트를 작성하고 이를 성공시키는 코드를 만드는 작업의 주기를 가능한 짧게 가져가도록 권장한다.

개발한 코드의 오류는 빨리 발견할수록 좋다.

### 2.3.5 테스트 코드 개선

테스트 코드도 언제 든지 내부구조와 설계를 개선해서 좀 더 깔끔하고 이해하기 쉬우며 변경이 용이한 코드 로 만들 필요가 있다.

메소드에 @Before 애노테이션을 붙이고 각 테스트마다 중복되는 코드를 모아놓는다.

```java {3,6} title="UserDaoTest.java"
public class UserDaoTest {

  @Before
  public void setUp() {
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
  }
  
  // ...
  
}
```

JUnit 이 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식은 다음과 같다.

1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다.
2. 테스트 클래스의 오브젝트를 하나 만든다.
3. @Before가 붙은 메소드가 있으면 실행한다.
4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다.
5. @After가 붙은 메소드가 있으면 실행한다.
6. 나머지 테스트 메소드에 대해 2〜5번을 반복한다.
7. 모든 테스트의 결과를 종합해서 돌려준다.

JUnit 은 @Test 가 붙은 메소드를 실행하기 전과 후에 각각 @Before 와 @After 가 붙은 메소드를 자동으로 실행한다.

다만 @Before 와 @After 메소드를 통해 만들어지는 정보는 인스턴스 변수를 이용해야 한다.

JUnit 은 각 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만든다.

테스트가 서로 영향을 주지 않고 독립적으로 실행됨을 확실히 보장하기 위해서다.

테스트를 수행하는 데 필요한 정보나 오브젝트를 픽스처 fixture 라고 한다.

일반적으로 픽스처는 여러 테스트에서 반복적으로 사용되기 때문에 @Before 메소드를 이용해 생성해둔다.
