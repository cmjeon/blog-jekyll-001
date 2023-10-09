---
layout: single
title: "1장 오브젝트와 의존관계 - 1.8 XML 을 이용한 설정"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 1장 오브젝트와 의존관계

## 1.8 XML 을 이용한 설정

범용 DI 컨테이너를 사용하면서 오브젝트간 의존정보를 DaoFactory 같은 자바 코드로 만드는 것은 번거롭다.

스프링은 다양한 방법으로 DI 의존관계 설정정보를 만들 수 있고, 대표적인 것이 XML 이다.

### 1.8.1 XML 설정

하나의 @Bean 메소드를 통해 얻을 수 있는 빈의 DI 정보는 아래와 같다.

- 빈의 이름
- 빈의 클래스
- 빈의 의존 오브젝트

XML 로도 세 가지 정보를 정의할 수 있다.

예시

```xml
<beans>
    <!-- bean 의 id 는 property 의 ref 와 동일해야 한다. -->
    <bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />
    <bean id="userDao" class="springbook.user.dao.UserDao">
        <property name="connectionMaker" ref="connectionMaker" />
    </bean>
</beans>
```

bean 의 id 애트리뷰트는 생성자 메소드 명이다.

bean 의 class 애트리뷰트는 생성할 클래스의 위치이다.

property 의 name 애트리뷰트는 DI 에 사용할 수정자 setter 메소드의 프로퍼티 명이다.

프로퍼티 이름은 바뀔 수 있는 클래스 이름보다는 대표적인 인터페이스 이름을 따르는 편이 자연스럽다.

ref 애트리뷰트는 주입할 오브젝트를 정의한 빈의 DI 이다.

### 1.8.2 XML 을 이용하는 애플리케이션 컨텍스트

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi=Mhttp://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />
  <bean id="userDao" class="springbook.user.dao.UserDao">
    <property name="connectionMaker" ref="connectionMaker" />
  </bean>

</beans>
```

UserDaoTest 의 애플리케이션 컨텍스트 생성 부분을 수정한다.

```java
public class UserDaoTest {

	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ApplicationContext context = new GenericXmlApplicationContext "applicationcontext.xml");

		// ...
	}

}
```

GenericXmlApplicationContext 외에 ClassPathXmlApplicationContext 를 이용할 수도 있다.

### 1.8.3 DataSource 인터페이스로 변환

자바에서는 DB 커넥션을 가져오는 오브젝트의 기능을 추상화해서 만들어놓은 DataSource 라는 인터페이스가 이미 존재한다.

```java
public interface DataSource extends CommonDataSource,Wrapper {
  Connection getConnection() throws SQLException;
}
```

UserDao 가 DataSource 를 사용하도록 바꿔본다.

```java
public class UserDao {

  private DataSource dataSource;

  public void setDataSource(DataSource dataSource) {
    this.dataSource = dataSource;
  }

  public void add(User user) throws SQLException {
    Connection c = dataSource.getConnection();
  }

}
```

DaoFactory 의 설정 방식을 변경한다.

dataSource() 메소드를 생성하고 userDao 가 dataSource 를 받도록 수정한다.

```java
@Configuration
public class DaoFactory {

  @Bean
  public DataSource dataSource() {
    SimpleDriverDataSource dataSource = new SimpleDriverDataSource ();

    dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
    dataSource.setUrl("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8");
    dataSource.setUsername("spring");
    dataSource.setPassword("book");

    return dataSource;
  }

  @Bean
  public UserDao userDao() {
    UserDao userDao = new UserDao();
    userDao.setDataSource(dataSource());
    return userDao;
  }

}
```

XML 설정 방식으로 변경해서 작업해본다.

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost/springbook?characterEncoding=UTF-8" />
		<property name="username" value="spring" />
		<property name="password" value="book" />
</bean>
```

driveClass 는 java.lang.class 타입인데 문자열 String 인 텍스트형태로 쓰여져 있다.

스프링이 프로퍼티의 값을 수정자 setter 메소드의 파라미터 타입을 참고해서 적절한 형태로 변환해주기 때문이다.

아래와 같은 작업이 발생한다고 생각하면 된다.

```java
Class driverclass = Class.forName("com.mysql.jdbc.Driver");
dataSource.setDriverClass(driverClass);
```

## 1.9 정리

- 먼저 책임이 다른 코드를 분리해서 두 개의 클래스로 만들었다(관심사의 분리. 리팩토링).
- 그중에서 바뀔 수 있는 쪽의 클래스는 인터페이스를 구현하도록 하고, 다른 클래스에서 인터페이스를 통해서만 접근하도록 만들었다. 이렇게 해서 인터페이스를 정의한 쪽의 구현 방법이 달라져 클래스가 바뀌더라도, 그 기능을 사용하는 클래스의 코드는 같이 수정할 필요가 없도록 만들었다(전략 패턴).
- 이를 통해 자신의 책임 자체가 변경되는 경우 외에는 불필요한 변화가 발생하지 않도록 막아주고, 자신이 사용하는 외부 오브젝트의 기능은 자유롭게 확장하거나 변경할 수 있게 만들었다(개방 폐쇄 원칙).
- 결국 한쪽의 기능 변화가 다른 쪽의 변경을 요구하지 않아도 되게 했고(낮은 결합도), 자신의 책임과 관심사에만 순수하게 집중하는(높은 웅집도) 깔끔한 코드를 만들 수 있었다.
- 오브젝트가 생성되고 여타 오브젝트와 관계를 맺는 작업의 제어권을 별도의 오브젝트 팩토 리를 만들어 넘겼다. 또는 오브젝트 팩토리의 기능을 일반화한 IoC 컨테이너로 넘겨서 오브 젝트가 자신이 사용할 대상의 생성이나 선택에 관한 책임으로부터 자유롭게 만들어줬다(제어의 역전/IoC).
- 전통적인 싱글톤 패턴 구현 방식의 단점을 살펴보고, 서버에서 사용되는 서비스 오브젝트 로서의 장점을 살릴 수 있는 싱글톤을 사용하면서도 싱글톤 패턴의 단점을 극복할 수 있도록 설계된 컨테이너를 활용하는 방법에 대해 알아봤다(싱글톤 레지스트리).
- 설계 시점과 코드에는 클래스와 인터페이스 사이의 느슨한 의존관계만 만들어놓고, 런타임 시에 실제 사용할 구체적인 의존 오브젝트를 제3자(DI 컨테이너)의 도움으로 주입받아서 다이내믹한 의존관계를 갖게 해주는 IoC의 특별한 케이스를 알아봤다(의존관계 주입/DI).
- 의존 오브젝트를 주입할 때 생성자를 이용하는 방법과 수정자 메소드를 이용하는 방법을 알 아봤다(생성자 주입과 수정자 주입).
- 마지막으로, XML을 이용해 DI 설정정보를 만드는 방법과 의존 오브젝트가 아닌 일반 값 을 외부에서 설정해서 런타임 시에 주입하는 방법을 알아봤다. (XML 설정)
