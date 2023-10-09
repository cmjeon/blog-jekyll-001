---
layout: single
title: "2장 테스트 - 2.2 UserDaoTest 개선"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 2장 테스트

## 2.2 UserDaoTest 개선

### 2.2.1 테스트 검증의 자동화

테스트를 눈으로 확인해야하는 부분을 개선해본다.

모든 테스트는 성공과 실패의 두 가지 결과를 가질 수 있다.

UserDaoTest 클래스의 마지막 부분을 수정한다.

```java
public class UserDaoTest {
  public static void main(String[] args) throws SQLException {
    
    // ...

    User user2 = dao.get(user.getId());
    System.out.println(user2.getName());
    System.out.println(user2.getPassword());

    System.out.println(user2,getId() + " 조회 성공");
  }
}
```

처음의 user 오브젝트와 두번째 user 오브젝트를 비교해서 조건이 맞으면 테스트를 성공, 아니면 실패로 고친다.

```java
public class UserDaoTest {
  public static void main(String[] args) throws SQLException {
  
    // ...
    
    if (!user.getName().equals(user2.getName())) { 
      System.out.println("테스트 실패 (name)");
    } else if (!user.getPassword().equals(user2.getPassword())) {
      System.out.println("테스트 실패 (password)"); 
    } else {
      System.out.println("조회 테스트 성공");
    }
  }
}
```

이제 테스트가 실행되면 조건을 비교해서 '테스트 실패', 혹은 '테스트 성공' 이라고 결과를 보여준다.

테스트를 수행하고 결과를 검증하는 것까지 자동화된 것이다.

'테스트 실패' 로 나오면 그 원인을 찾아서 코드를 수정하면 되고, 최종적으로 '테스트 성공' 이라고 나오면 기능이 완성되었다고 확신을 하면 된다.

자동화된 테스트를 위한 xUnit 프레임워크를 만든 켄트 벡은 "테스트란 개발자가 마음 편하게 잠자리에 들 수 있게 해주는 것" 이라고 했다.

개발 과정에서 기존 코드를 개선하면서 문제가 없는지 확인할 수 있는 가장 좋은 방법은 자동화된 테스트를 만들어두는 것이다.

### 2.2.2 테스트의 효율적인 수행과 결과 관리

자바 테스팅 프레임워크라고 불리는 JUnit 테스트로 전환해본다.

UserDaoTest 의 main() 메소드를 JUnit 프레임워크를 이용해 다시 작성해본다.

여기서도 제어의 역전을 이용해 프레임워크가 개발자가 만든 클래스의 오브젝트를 생성하고 실행한다.

JUnit 프레임워크로 만드는 테스트 메소드는 public 선언되어야 하고 @Test 애노테이션이 붙어야 한다.

```java
public class UserDaoTest {
  
  @Test
  public void addAndGet() throws SQLException {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml"); 
    UserDao dao = context.getBean("userDao", UserDao.class);
  }

}
```

위의 if 문의 기능을 assertThat 이라는 스태틱 메소드로 변경한다.

```java
// 기존 if 문
if (!user.getName().equals(user2.getName())) { 

// assertThat
assertThat(user2.getName(), is(user.getName()));
```

assertThat() 메소드의 첫 번째 파라미터의 값을 뒤에 나오는 매처 matcher 라고 불리는 조건으로 비교해서 일치하면 다음으로 넘어가고, 아니면 실패한다.

JUnit 은 예외가 발생하거나 assertThat() 에서 실패하지 않고 테스트 메소드의 실행이 완료되면 테스트가 성공했다고 인식한다.

UserDaoTest.java 를 JUnit 프레임워크에서 실행될 수 있게 수정하였다.

```java
public class UserDaoTest {
  
  @Test 
  public void andAndGet() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
    UserDao dao = context.getBean("userDao", UserDao.class);
    
    User user = new User();
    user.setId("gyumee");
    user.setName("박성철");
    user.setPassword("springno1");

    dao.add(user);
      
    User user2 = dao.get(user.getId());
    
    assertThat(user2.getName(), is(user.getName()));
    assertThat(user2.getPassword(), is(user.getPassword()));
  }
  
}
```

JUnit 테스트 실행하기 위해 main() 메소드를 추가하고 실행해 준다.

```java
public class UserDaoTest {
  
  @Test 
  public void andAndGet() throws SQLException {
    // ...
  }
  
  public static void main(String[] args) {
    JUnitCore.main("springbook.user.dao.UserDaoTest");
  }
  
}
```

클래스를 실행하면 테스트하는데 걸린 시간과 테스트 결과, 그리고 몇 개의 테스트 메소드가 실행되었는지 알려준다.

JUnit 은 assertThat() 메소드를 이용해 검증을 했을 때 기대한 결과가 아니면 이 AssertionError 를 던진다.

따라서 assertThat() 의 조건을 만족시키지 못하면 테스트는 더 이상 진행되지 않고 JUnit 은 테스트가 실패했음을 알린다.

> 본문의 JUnit 은 4 버전이고, 현재 나온 JUnit 은 5 버전으로 사용방법이 다소 상이하다.
> JUnit 5 버전의 작성법은 아래를 참고한다.
> [https://junit.org/junit5/docs/current/user-guide/#writing-tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests)
