---
layout: single
title: "3장 템플릿 - 3.1 다시 보는 초난감 DAO"
categories:
  - "토비의 스프링 3.1"
tags:
  - "템플릿"
---

# 3장 템플릿

객체지향 설계의 핵심 원칙인 개방 폐쇄 원칙 OCP 을 다시 한번 생각해보자

템플릿이란 이렇게 바뀌는 성질이 다른 코드 중에서 변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로 독립시켜서 효과적으로 활용할 수 있도록 하는 방법이다.

## 3.1 다시 보는 초난감 DAO

UserDao 의 코드는 예외상황에 대한 처리가 어렵다는 문제가 있다.

### 3.1.1 예외처리 기능을 갖춘 DAO

JDBC 코드는 중간에 어떤 이유로든 예외가 발생했을 때도 사용한 리소스를 반드시 반환해야 한다.

```java
public void deleteAll() throws SQLException {
  Connection c = dataSource.getConnection();
  
  PreparedStatement ps = c.prepareStatement("delete from users");
  ps.executeUpdate();
  
  ps.close();
  c.close();
```

하이라이트된 부분에서 예외가 발생하면 close() 메소드가 실행되지 않아서 제대로 리소스가 반환되지 않게 된다.

미처 반환되지 못한 Connection 이 쌓이면 서버가 중단될 수도 있기 때문에 치명적인 위험을 가진 오류이다.

> Connection 과 PreparedStatement 는 보통 풀 pool 을 만들어두고, 필요할 때 할당하고 반환하면 다시 풀에 넣는 방식으로 운영된다.

오류가 발생해도 리소스를 반환할 수 있도록 try/catch/finally 구문을 사용해보자

```java
public void deleteAll() throws SQLException {
  Connection c = null;
  PreparedStatement ps = null;
  
  try {
    c = dataSource.getConnection();
    ps = c.prepareStatement("delete from users");
    ps.executeUpdate();
  } catch (SQLException e) {
    throw e;
  } finally {
    // ...
    ps.close();
    // ...
    c.close();
    // ...
  }
}  
```

JDBC 조회 기능의 예외처리도 try/catch/finally 로 감싸준다.

```java
public int getCount() throws SQLException  {
	Connection c = dataSource.getConnection();
	PreparedStatement ps = null;
    ResultSet rs = null;
    
    try {
    	c= dataSource.getConnection();
        ps = c.prepareStatement("select count(*) from users");
        
        rs = ps.executeQuery();
        rs.next();
        return rs.getInt(1);
	} catch (SQLException e) {
    	throw e;
    } finally {
    	if (rs != null) {
        	try {
            	rs.close();
            } catch (SQLException e) {
            }
        }
        if (ps != null) {
        	try {
            	ps.close();
            } catch (SQLException e) {
            }
        }
        if (c != null) {
        	try {
            	c.close();
            } catch (SQLException e) {
            }
        }
	}
}  
```
