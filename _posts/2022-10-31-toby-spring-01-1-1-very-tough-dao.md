---
layout: single
title: "1장 오브젝트와 의존관계 - 1.1 초난감 DAO"
categories:
  - "토비의 스프링 3.1"
tags:
  - "오브젝트와 의존관계"
---

스프링이 가장 관심을 많이 두는 대상은 오브젝트다.

오브젝트에 대한 관심은 오브젝트 설계 방법인 디자인 패턴, 구조를 개선하는 리팩토링, 검증하는데 쓰이는 테스트를 발전시켰다.

## 1.1 초난감 DAO

### 1.1.1 User

사용자 정보를 저장하기 위해 자바빈 규약을 따르는 User 클래스를 만든다.

```java
public class User {

  String id;
  String name;
  String password;

  public String getId {
    return id;
  }
  public void setId(String id) {
    this.id = id;
  }
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  )
  public String getPassword() {
    return password;
  }
  public void setPassword(String password) {
    this.password = password;
  }

}
```

<div class="notice--primary" markdown="1">
자바빈(JavaBean)은 두 가지 관례를 따라 만들어진 오브젝트를 가리킨다.<br>
디폴트 생성자 : 프레임워크에서 리플렉션을 이용해 오브젝트를 생성하기 위해 필요하다.<br> 
프로퍼티 : 자바빈이 노출하는 이름을 가진 속성을 프로퍼티라고 한다. 프로퍼티는 set, get 메소드를 이용해 수정, 조회할 수 있다.<br>
<br>
1장_ 오브젝트와 의존관계, 55.<br>
</div>

### 1.1.2 UserDao

UserDao 는 User 객체를 DB 에 저장하고 불러올 수 있는 클래스이다.

DB 를 사용하기 위해서는 JDBC 를 이용해야 됙, 이를 위해 UserDao 를 구현해 보자.

UserDao 는 User 데이터를 DB 에 저장하는 add 메소드, 저장된 User 데이터를 가져오는 get 메소드를 가지고 있어야 한다.

```java
public class UserDao {

  public void add(User user) throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?", "spring", "book");

    PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
    ps.setString(1, user.getId());
    ps.setString(2, user.getName());
    ps.setString(3, user.getPassword());

    ps.executeUpdate();

    ps.close();
    c.close();
  }

  public User get(String id) throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");

    PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
    ps.setString(1, id);

    ResultSet rs = ps.executeQuery();
    rs.next();

    User user = new User();
    user.setId(rs.getString("id"));
    user.setName(rs.getString("name"));
    user.setPassword(rs.getString("password"));

    rs.close();
    ps.close();
    c.close();

    return user;
  }

}
```

### 1.1.3 main() 을 이용한 DAO 테스트 코드

만들어진 코드의 기능을 검증하기 위해 UserDao 에 main() 메소드를 만들어서 UserDao 를 테스트 해본다.

```java
public class UserDao {

  // ...

  public static void main(String[] args) throws ClassNotFoundException, SQLException {
    UserDao dao = new UserDao();

    User user = new User();
    user.setId("whiteship");
    user.setName("백기선");
    user.setPassword("married");

    dao.add(user);

    System.out.println(user.getId() + " 등록 성공");

    User user2 = dao.get(user.getId());
    System.out.println(user2.getName());
    System.out.println(user2.getPassword());

    System.out.println(user2.getId() + " 조회 성공");
  }

}
```

테스트용 main 메소드를 실행해보면 잘 작동하는 것을 확인할 수 있다.

하지만 안타깝게도 위의 UserDao 는 객체지향적으로 만들어진 코드는 아니다.

이제부터 이 UserDao 를 고쳐가면서 객체지향 기술의 원리에 충실한 코드를 만들어 보겠다.

<div class="notice--primary" markdown="1">
일단 UserDao 클래스 코드의 문제점은 무엇일까 스스로 생각해보자.<br>
문제점이라고 하면 뭔가 기능이 정상적으로 동작하지 않아야 할 텐데, 사실 위의 DAO 는 우리가 기대했던 기능을 충실하게 동작한다는 사실을 초간단 main() 메소드 테스트 방식을 이용해 이미 두 눈으로 검증했다<br>
그렇다면, 왜 이 모드에 문제가 많다고 하는 것일까?<br>
잘 동작하는 코드를 굳이 수정하고 개선해야 하는 이유는 뭘까?<br>
그렇게 DAO 코드를 개선했을 때의 장점은 무엇일까?<br>
그런 장점들이 당장에, 또는 미래에 주는 유익은 무엇인가? 또, 객체지향 설계의 원칙과는 무슨 상관이 있을까?
이 DAO 를 개선하는 경우와 그대로 사용하는 경우, 스프링을 사용하는 개발에서 무슨 차이가 있을까?<br>
스프링을 공부한다는 건 바로 이런 문제 제기와 의문에 대한 답을 찾아나가는 과정이다.<br>
<br>
1장_ 오브젝트와 의존관계, 59.<br>
</div>
