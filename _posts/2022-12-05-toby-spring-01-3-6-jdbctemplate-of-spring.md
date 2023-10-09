---
layout: single
title: "3장 템플릿 - 3.6 스프링의 JdbcTemplate"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 3장 템플릿

## 3.6 스프링의 JdbcTemplate

스프링의 템플릿/콜백 기술을 살펴보자.

스프링이 제공하는 JDBC 코드용 기본 템플릿은 JdbcTemplate 이다.

UserDao 가 JdbcTemplate 을 받을 수 있도록 변경한다.

```java
public class UserDao {
    
    // ...
    
    // highlight-next-line
    private JdbcTemplate jdbcTemplate;
    
    public void setDataSource(DataSource dataSource) {
        // highlight-next-line
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        this.dataSource = dataSource;
    }
    
    // ...
    
}
```

### 3.6.1 update()

deleteAll() 메소드를 변경한다.

makePreparedStatement() 를 createPreparedStatement() 메소드로 변경한다.

이는 둘다 템플릿으로부터 Connection 을 제공받아서 PreparedStatement 를 돌려준다.

```java
public void deleteAll() {
    this.jdbcTemplate.update(
        new PreparedStatementCreator() {
            public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
                return con.prepareStatement("delete from users");
            }
        }
    );
}
```

JdbcTemplate 의 내장 콜백을 사용하는 update() 메소드로 변경하였다.

```java
public void deleteAll() {
    this.jdbcTemplate.update("delete from users");
}
```

add() 메소드의 콜백에서 수행하는 PreparedStatement 를 만들고, 파라미터를 바인딩하는 작업도 update() 메소드로 가능하다.

```java
public void add() {
    this.jdbcTemplate.update("insert into users(id, name, password) values(?, ?, ?)", 
            user.getId(), 
            user.getName(),
            user.getPassword()
    );
}
```

### 3.6.2 queryForInt()

getCount() 메소드에 JdbcTemplate 의 query() 메소드를 적용한다.

```java
public int getCount() {
    return this.jdbcTemplate.query(
            new PreparedStatementCreator() {
                public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
                    return con.prepareStatement("select count(*) from users");
                }
            },
            new ResultSetExtractor<Integer>() {
                public Integer extractData(ResultSet rs) throws SQLException, DataAccessException {
                    rs.next();
                    return rs.getInt(1);
                }
            }
    );
}
```

2개의 콜백이 있어 혼란스럽다.

원래 getCount() 메소드에 있던 코드 중에서 변하는 부분만 콜백으로 만들어져서 제공된다고 생각하면 된다.

클라이언트 -> 템플릿 -> 콜백의 관계를 기억하자.

두 번째 콜백은 Generic 을 사용하였다.

이를 통해 query() 템플릿의 리턴 타입도 바뀌게 된다.

위 두 개의 콜백을 내장하고 있는 query() 템플릿을 JdbcTemplate 의 queryForInt() 로 변경할 수 있다.

```java
public int getCount () {
    return this.jdbcTemplate.queryForInt("select count(*) from users");
}
```

### 3.6.3 queryForObject()

get() 메소드에도 JdbcTemplate 을 적용해보자.

get() 메소드에는 바인딩이 필요한 치환자가 있다.

그리고 ResultSet 이 User 객체로 리턴되어야 한다.

```java
public User get(String id) {
    return this.jdbcTemplate.queryForObject(
            "select * from users where id = ?",
            new Object[] {id},
            new RowMapper<User>() {
                public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                    User user = new User();
                    user.setId(rs.getString("id"));
                    user.setName(rs.getString("name"));
                    user.setPassword(rs.getString("password"));
                    return user;
                }
            }
    );
}
```

첫 번째 파라미터는 PreparedStatement 를 만들기 위한 SQL, 두 번째는 바인딩할 값이다.

queryForObject() 메소드 내부에서 이 두 파라미터를 사용하는 PreparedStatement 콜백이 만들어진다.

RowMapper 에서는 ResultSet 의 로우 내용을 User 오브젝트에 담아서 리턴한다.

만약 queryForObject() 메소드의 실행결과가 비어있다면 EmptyResultDataAccessException 예외를 던지도록 만들어져 있다.

### 3.6.4 query()

getAll() 메소드를 추가해서 사용자 목록을 가져올 수 있도록 한다.

먼저 테스트를 만든다.

```java
// ...
public class UserDaoTest {

  // ...
  
  @Test
	public void getAll()  {
		dao.deleteAll();
		
		List<User> users0 = dao.getAll();
		assertThat(users0.size(), is(0));
		
		dao.add(user1); // Id: gyumee
		List<User> users1 = dao.getAll();
		assertThat(users1.size(), is(1));
		checkSameUser(user1, users1.get(0));
		
		dao.add(user2); // Id: leegw700
		List<User> users2 = dao.getAll();
		assertThat(users2.size(), is(2));
		checkSameUser(user1, users2.get(0));  
		checkSameUser(user2, users2.get(1));
		
		dao.add(user3); // Id: bumjin
		List<User> users3 = dao.getAll();
		assertThat(users3.size(), is(3));
		checkSameUser(user3, users3.get(0));  
		checkSameUser(user1, users3.get(1));  
		checkSameUser(user2, users3.get(2));  
	}
	
}
```

getAll() 메소드를 만든다.

get() 메소드와 다른 점은 JdbcTemplate 의 query() 메소드를 사용한다는 것이다.

query() 의 리턴 타입은 `List<T>` 이므로 `RowMapper<T>` 콜백 오브젝트에서 결정할 수 있다.

```java
public List<User> getAll() {
  return this.jdbcTemplate.query(
    "select * from users order by id",
    new RowMapper<User>() {
      public User mapRow(ResultSet rs, int rowNum) throws SQLException {
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));
        return user;
      }
    }
  );
}
```

현명한 개발자가 되려면 네거티브 테스트에 익숙해지는 것이 좋다.

테스트에 데이터가 없는 경우에 대한 검증 코드를 추가한다.

```java
// ...

public class UserDaoTest {

  // ...
  
  @Test
	public void getAll()  {
		dao.deleteAll();
		
		// ...
		
		// highlight-start
		List<User> users0 = dao.getAll();
		assertThat(users0.size(), is(0));
		// highlight-end
		
	}
	
}
```

### 3.6.5 재사용 가능한 콜백의 분리

#### DI 를 위한 코드 정리

JdbcTemplate 를 사용하면 dataSource 가 필요하지 않으니 DataSource 인스턴스 변수를 제거한다.

JdbcTemplate 을 생성할 때 직접 DI 해주기 위해 setter 메소드는 남겨둔다.

#### 중복 제거

get(), getAll() 메소드에서 사용하는 RowMapper 를 재사용가능하도록 분리한다.

```java
// ...

public class UserDao {

  // ...
  
  private RowMapper<User> userMapper = new RowMapper<User>() {
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
      User user = new User();
      user.setId(rs.getString("id"));
      user.setName(rs.getString("name"));
      user.setPassword(rs.getString("password"));
      return user;
    }
  };
  
  public User get(String id) {
		return this.jdbcTemplate.queryForObject(
		  "select * from users where id = ?",
			new Object[] {id},
			// highlight-next-line 
			this.userMapper
    );
	} 
	
	public List<User> getAll() {
		return this.jdbcTemplate.query(
		  "select * from users order by id",
		  // highlight-next-line
		  this.userMapper
    );
	}
}
```

#### 템플릿/콜백 패턴과 UserDao

UserDao 에는 User 정보를 DB 에 넣거나 가져오거나 조작하는 방법에 대한 핵심적인 로직만 담겨 있다.

JdbcTemplate 에는 JDBC API 를 사용하는 방식, 예외처리, 리소스의 반납, DB 연결과 관련된 책임과 관심을 담고 있다.

UserDao 는 다른 코드와 낮은 결합도를 유지하고 있다.

JdbcTemplate 를 직접 이용한다는 측면에서 특정 템플릿/콜백 구현에 강한 결합을 가지고 있다.

만약 더 낮은 결합을 유지하고 싶다면 인터페이스를 통해 DI 받아 사용하게 만들어도 된다.

이 외에도 userMapper 를 별도의 빈으로 만들어버리는 방법, SQL 문장을 UserDao 코드가 아닌 외부 리소스에 담고 이를 읽어와 사용하게 하는 방법이 있겠다.

## 3.7 정리

- JDBC와 같은 예외가 발생할 가능성이 있으며 공유 리소스의 반환이 필요한 코드는 반드시 try/catch/finally 블록으로 관리해야 한다.
- 일정한 작업 흐름이 반복되면서 그중 일부 기능만 바뀌는 코드가 존재한다면 전략 패턴을 적용한다. 바뀌지 않는 부분은 컨텍스트로, 바뀌는 부분은 전략으로 만들고 인터페이스를 통해 유연하게 전략을 변경할 수 있도록 구성한다.
- 같은 애플리케이션 안에서 여러 가지 종류의 전략을 다이내믹하게 구성하고 사용해야 한다면 컨텍스트를 이용하는 클라이언트 메소드에서 직접 전략을 정의하고 제공하게 만든다.
- 클라이언트 메소드 안에 익명 내부 클래스를 사용해서 전략 오브젝트를 구현하면 코드도 간결해지고 메소드의 정보를 직접 사용할 수 있어서 편리하다.
- 컨텍스트가 하나 이상의 클라이언트 오브젝트에서 사용된다면 클래스를 분리해서 공유하도록 만든다.
- 컨텍스트는 별도의 빈으로 등록해서 DI 받거나 클라이언트 클래스에서 직접 생성해서 사용한다. 클래스 내부에서 컨텍스트를 사용할 때 컨텍스트가 의존하는 외부의 오브젝트가 있다 면 코드를 이용해서 직접 DI 해줄 수 있다.
- 단일 전략 메소드를 갖는 전략 패턴이면서 익명 내부 클래스를 사용해서 매번 전략을 새로 만들어 사용하고, 컨텍스트 호출과 동시에 전략 DI를 수행하는 방식을 템플릿/콜백 패턴이라고 한다.
- 콜백의 코드에도 일정한 패턴이 반복된다면 콜백을 템플릿에 넣고 재활용하는 것이 편리하다.
- 템플릿과 콜백의 타입이 다양하게 바뀔 수 있다면 제네릭스를 이용한다.
- 스프링은 JDBC 코드 작성을 위해 Jdbc Temolate을 기반으로 하는 다양한 템플릿과 콜백을 제공한다.
- 템플릿은 한 번에 하나 이상의 콜백을 사용할 수도 있고, 하나의 콜백을 여러 번 호출할 수도 있다.
- 템플릿/콜백을 설계할 때는 템플릿과 콜백 사이에 주고받는 정보에 관심을 둬야 한다.
