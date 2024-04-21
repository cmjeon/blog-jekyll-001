---
# layout: post
title: "7장 스프링 핵심 기술의 응용 - 7.6 스프링 3.1의 DI"
# excerpt: "DAO 에 트랜잭션을 적용해보면서 스프링이 어떻게 성격이 비슷한 여러 기술을 추상화하고, 일관된 방법으로 사용할 수 있도록 지원하는지 알아봅니다."
categories:
  - "토비의 스프링 3.1"
tags:
  - "스프링 핵심 기술의 응용"
---

## 7.6 스프링 3.1의 DI

스프링이 처음 등장한 이후 많은 변화를 겪은 것은 사실이지만 스프링이 근본적으로 지지하는 객체지향 언어인 자바의 특징과 장점을 극대화하는 프로그래밍 스타일과 이를 지원하는 도구로서의 스프링 정체성은 변하지 않았습니다.

#### 자바 언어의 변화와 스프링

DI 가 적용된 코드를 작성할 때 사용되는 자바 언어가 그간 많은 변화가 있어서 스프링의 사용 방식에도 영향을 줬습니다.

- 애노테이션의 메타정보 활용

자바 코드의 메타정보를 이용한 프로그래밍 방식입니다.

자바 코드는 실행되는 것이 아니라 다른 자바 코드에 의해 데이터처럼 취급되기도 합니다.

자바 코드의 일부를 리플렉션 API 등을 이용해 어떻게 만들었는지 살펴보고 그에 때라 구현하는 방식입니다.

원래 리플렉션 API 는 자바 코드나 컴포넌트를 작성하는 데 사용되는 툴을 개발할 때 이용하도록 만들어졌습니다.

그러다가 점점 자바 코드의 메타정보를 데이터로 활용하는 스타일의 프로그래밍 방식에 활용되기 시작했습니다.

이 방식은 애노테이션이 등장하면서 본격화되기 시작했습니다.

애노테이션은 리플렉션 API 를 이용해 애노테이션의 메타정보를 조회하여 설정된 값을 가져와 참고하는 방식으로 사용됩니다.

애노테이션은 프레임워크가 참조하는 메타정보로 활용되기에 유리한 점이 많았기 때문에 활용이 늘어났습니다.

객체지향 프로그래밍을 적용하면 핵심 로직을 담은 오브젝트가 클라이언트에 의해 생성되고, 관계를 맺고 제어되는 구조가 됩니다.

이 때 클라이언트는 일종의 IoC 프레임워크로 동작하고, 팩토리는 IoC 프레임워크가 참조하는 메타정보가 됩니다.

메타정보를 편하게 작성하기 위해서 스프링 초창기부터 XML 이 프레임워크가 사용하는 DI 메타정보로 적극 활용되었습니다.

애노테이션이 등장하면서 XML 같은 외부 파일이 아닌 자바 코드의 일부로 메타정보를 사용할 수 있는 방식이 가능해졌습니다.

애노테이션은 정의하기에 따라 타입, 필트, 메소드, 파라미터, 생성자, 로컬 변수에 적용이 가능합니다.

그리고 애노테이션을 통해 다양한 부가 정보를 얻어낼 수 있습니다.

XML 과 다르게 애노테이션은 자바 코드에 존재하므로 변경할 때마다 컴파일하는 단점이 있습니다.

- 정책과 관례를 이용한 프로그래밍

애노테이션 같은 메타정보를 활용하는 방식은 명시적인 방식에서 관례적인 방식으로 프로그래밍 스타일을 변화시켜 왔습니다.

이런 스타일의 장점은 모든 작업을 표현하는 것에 비해 작성할 내용이 줄어든다는 것입니다.

스프링은 애노테이션으로 메타정보를 작성하고, 관례를 활용해서 간결한 코드에 많은 내용을 담을 수 있도록 하고 있습니다.

이제 최신 DI 스타일로 변경하는 과정을 보여주고 설명합니다.

### 7.6.1 자바코드를 이용한 빈설정

먼저 XML 을 없애는 작업을 합니다.

지금까지 만든 XML 설정은 테스트용 DI 설정입니다.

이제부터는 DI 관련 정보를 스프링 3.1로 변경하는 일과 테스트환경과 운영환경에서 동작할 때 필요로 하는 DI 정보를 분리해내는 일도 포함합니다.

#### 테스트 컨텍스트의 변경

스프링 3.1 은 애노테이션과 자바 코드로 만들어진 DI 설정정보와 XML 을 동시에 사용할 수 있는 방법을 제공해줍니다.

그래서 순차적으로 XML 파일을 애노테이션과 자바 코드로 변경해보겠습니다.

UserDaoTest, UserServiceTest 에는 설정정보를 담은 XML 의 위치를 지정하는 코드가 들어가 있습니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
// highlight-next-line
@ContextConfiguration(location="/test-applicationContext.xml")
public class UserDaoTest {
```

@ContextConfiguration 은 스프링 테스트가 테스트용 DI 정보를 어디서 가져와야 하는지 지정할 때 사용하는 애노테이션입니다.

DI 설정정보를 담은 클래스는 평범한 자바 클래스를 만들고 @Configuration 애노테이션을 달아주면 만들 수 있습니다.

```java
@Configuration
public clas TestApplicationContext {
}
```

이제 @ContextConfiguration 의 XML 의 위치를 변경해줍니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
// highlight-next-line
@ContextConfiguration(class=TestApplicationContext.class)
public class UserDaoTest {
```

이제 XML 에 있던 빈 설정정보를 TestApplicationContext 로 이동시켜야 합니다.

순차적으로 옮기기 위해 우선 TestApplicationContext 에서 XML 의 빈 설정정보를 가져와서 사용하게 합니다.

```java
@Configuration
@ImportResource("/test-applicationContext.xml")
public class TestApplicationContext {
```

TestApplicationContext 에 자바 코드와 애노테이션으로 정의된 DI 정보와 @ImportResource 로 가져온 XML 의 DI 정보가 합쳐져서 최종 DI 설정정보로 통합됩니다.

이제 단계적으로 XML 의 내용을 자바코드로 옮기고, 모든 내용을 옮기고 나면 XML 파일과 @ImportResource 를 제거할 것입니다.

#### <context:annotation-config /\> 제거

XML 파일에 <context:annotation-config /\> 가 있습니다.

<context:annotation-config /\> 는 빈 초기화 메소드인 @PostConstruct 를 실행하도록 해주는데 사용됩니다.

XML 에 담긴 DI 정보를 이용하는 스프링 컨테이너를 사용하는 경우 @PostConstruct 같은 애노테이션의 기능이 필요하면 반드시 <context:annotation-config /\> 를 포함시켜야 합니다.

반면에 TestApplicationContext 처럼 @Configuration 이 붙은 설정 클래스를 사용하면 <context:annotation-config /\> 가 필요없습니다.

컨테이너가 직접 @PostConstruct 애노테이션을 처리하는 빈 후처리기를 등록해주기 때문입니다.

#### <bean\> 의 전환

XML 에 있는 DB 연결과 트랜잭션 매니저 빈을 자바코드로 옮겨봅니다.

```xml 
// title="test-applicationContext.xml"
<beans mxlns="..."
  <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost/springbook?characterEncoding=UTF-8" />
    <property name="username" value="spring" />
    <property name="password" value="book" />
  </bean>
  // ...
</beans>
```

```java 
// title="TestApplicationContext.java"
@Configuration
public class TestApplicationContext {

  @Bean
  public DataSource dataSource() {
    // highlight-next-line
    SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
    
    // highlight-next-line
    dataSource.setDriverClass(Driver.class);
    dataSource.setUrl("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8");
    dataSource.setUsername("spring");
    dataSource.setPassword("book");
    return ds;
  }
  // ...
}
```

빈 오브젝트를 저장할 로컬 변수는 빈의 구현 클래스에 맞는 프로퍼티 값 주입이 필요합니다.

따라서 DataSource 가 아닌 SimpleDriverDataSource 타입으로 선언합니다.

XML 에서는 문자열 "com.mysql.jdbc.Driver" 를 보고 알아서 com.mysql.jdbc.Drive.class 로 변환해줍니다.

하지만 자바 코드로 작성할 때는 프로퍼티 타입에 맞는 값을 넣어야 합니다.

이제 트랜잭션 매니저를 옮깁니다.

빈을 다른 빈에 주입할 때는 수정자 메소드에서 주입해줄 빈의 메소드를 직접 호출해서 그 리턴 값을 넣어주면 됩니다.

```xml 
// title="test-applicationContext.xml"
<beans mxlns="..."
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />  
  </bean>
  // ...
</beans>
```

```java 
// title="TestApplicationContext.java"
@Configuration
public class TestApplicationContext {

  @Bean
  // highlight-next-line
  public PlatformTransactionManager transactionManager() {
    DataSourceTransactionManager tm = new DataSourceTransactionManager();
    // highlight-next-line
    tm.setDataSource(dataSource());
    return tm;
  }
  // ...
}
```

인터페이스가 TransactionManager 가 아닌 PlatformTransactionManager 입니다.

수정자메소드에서 dataSource() 를 직접 호출하고 있습니다.

나머지 빈들도 순차적으로 옮깁니다.

testUserService 빈을 옮길 때 주의해야 합니다.

```xml 
// title="test-applicationContext.xml"
<beans mxlns="..."
  <bean id="userService" class="springbook.user.service.UserServiceImpl">
    <property name="userDao" ref="userDao" />
    <property name="mailSender" ref="mailSender" />
  </bean>
  <bean id="testUserService" 
      class="springbook.user.service.UserServiceTest$TestUserService"
      // highlight-next-line 
      parent="userService" />
  // ...
</beans>
```

```java 
// title="TestApplicationContext.java"
@Configuration
public class TestApplicationContext {

  @Bean
  public UserService userService() {
    UserServiceImpl service = new UserServiceImpl();
    service.setUserDao(userDao());
    service.setMailSender(mailSender());
    return service;
  }
  
  @Bean
  public UserService testUserService() {
    TestUserService testService = new TestUserService();
    // highlight-start
    testService.setUserDao(userDao());
    testService.setMailSender(mailSender());
    // highlight-end
    return testService;
  }
  // ...
}
```

XML 파일에서는 parent 정의를 이용해 userService 프로퍼티 정의를 상속할 수 있었습니다.

자바코드로 전환할 때는 모든 프로퍼티를 주입해주어야 합니다.

또한 UserServiceTest 클래스에 있는 TestUserService 에 public 을 붙여주면 됩니다.

XML 과 자바 클래스를 동시에 DI 정보로 사용할 때, 자바 코드로 정의한 빈은 XML 의 `<property>` 를 이용해 참조할 수 있습니다.

반대로 자바코드에서 XML 에 정의한 빈을 참조할 때는 @Autowired 가 붙은 필드를 선언해서 주입받을 수 있습니다.

다른 @Configuration 클래스에서 정의한 빈을 참조할 때도 사용할 수 있습니다.

sqlService, sqlRegistry, unmarshaller 를 자바코드로 이동합니다.

```xml 
// title="test-applicationContext.xml"
<beans mxlns="..."
  <bean id="sqlService" class="springbook.user.sqlservice.OxmSqlService">
    <property name="unmarshaller" ref="unmarshaller" /> 
    <property name="sqlRegistry" ref="sqlRegistry" />
  </bean>
  
  <bean id="sqlRegistry" class="springbook.user.sqlservice.updatable.EmbeddedDbSqlRegistry">
    <property name="dataSource" ref="embeddedDatabase" />
  </bean>

  <bean id="unmarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
    <property name="contextPath" value="springbook.user.sqlservice.jaxb" />
  </bean>
  // ...
</beans>
```

```java 
// title="TestApplicationContext.java"
@Configuration
public class TestApplicationContext {
  @Bean
  public SqlService sqlService() {
    OxmSqlService sqlService = new OxmSqlService();
    sqlService.setUnmarshaller(unmarshaller());
    sqlService.setSqlRegistry(sqlRegistry());
    return sqlService;
  }
  
  // highlight-start
  @Resource
  Database embeddedDatabase;
  // highlight-end
  
  @Bean
  public SqlRegistry sqlRegistry() {
    EmbeddedDbSqlRegistry sqlRegistry = new EmbeddedDbSqlRegistry();
    // highlight-next-line
    sqlRegistry.setDataSource(this.embeddedDatabase);
    return sqlRegistry;
  }
  
  @Bean
  public Unmarshaller unmarshaller() {
    Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
    marshaller.setContextPath("springbook.user.sqlservice.jaxb");
    return marshaller;
  }
  // ...
}
```

sqlRegistry 에서 XML 에 정의된 embeddedDatabase 를 사용하기 위해서는 @Resource 사용해야 합니다.

@Resource 와 @Autowired 의 차이는 @Autowired 는 필드의 타입을 기준으로 빈을 찾고, @Resource 는 필드의 이름을 기준으로 빈을 찾는다는 것입니다.

embeddedDatabase 는 DataSource 타입의 빈을 생성합니다.

하지만 이미 TestApplicationContext 에 dataSource 빈이 존재합니다.

따라서 타입을 기준으로 embeddedDatabase 빈을 주입받게 되면 혼란이 발생할 수 있습니다.

그래서 필드 이름과 일치하는 빈 아이디를 주입받을 때는 @Resource 를 이용합니다.

#### 전용 태그 전환

이제 XML 에는 `<jdbc:embedded-database>` 와 `<tx:annotation-driven />` 두 개의 빈 설정만 있습니다.

`<jdbc:embedded-database>` 전용 태그는 type 에서 지정한 내장형 DB 를 생성하고 `<jdbc:script>` 로 지정한 스크립트로 초기화하고, DataSource 타입의 DB 커넥션 오브젝트를 빈으로 등록해줍니다.

XML 에서 `<jdbc:embedded-database>` 전용 태그로 빈 오브젝트를 생성하는 방식을 자바코드로 변경합니다.

```xml 
// title="test-applicationContext.xml"
<beans mxlns="..."
  <jdbc:embedded-database id="embeddedDatabase" type="HSQL">
    <jdbc:script location="classpath:springbook/user/sqlservice/updatable/sqlRegistrySchema.sql"/>
  </jdbc:embedded-database>
  // ...
</beans>
```

```java 
// title="TestApplicationContext.java"
@Configuration
public class TestApplicationContext {
  @Bean 
  public DataSource embeddedDatabase() {
    return new EmbeddedDatabaseBuilder()
      .setName("embeddedDatabase")
      .setType(HSQL)
      .addScript("classpath:springbook/user/sqlservice/updatable/sqlRegistrySchema.sql")
      .build();
  }
  // ...
}
```

이제 @Resource 로 빈을 호출할 필요가 없으니 @Resource 로 정의한 필드를 제거합니다.

sqlRegistry 에서 this.embeddedDatabase 로 주입받던 빈을 embeddedDatabase() 메소드를 호출하여 받을 수 있도록 변경한다.

이제 `<tx:annotation-driven />` 하나의 전용 태그만 남았습니다.

이 전용 태그는 트랜잭션 AOP 기능을 지원하는 태그입니다.

과거에는 이 전용 태그를 완전히 대체하기가 어려왔다고 합니다.

하지만 이제는 @EnableTransactionManagement 로 간단하게 변환할 수 있습니다.

```xml 
// title="test-applicationContext.xml"
<beans mxlns="..."
  <tx:annotation-driven />
  // ...
</beans>
```

```java 
// title="TestApplicationContext.java"
@Configuration
@ImportResource("/test-applicationContext.xml")
// highlight-next-line
@EnableTransactionManagement
public class TestApplicationContext {
  // ...
}
```

이렇게 XML 에서 모든 빈 설정을 옮겨왔습니다.

TestApplicationContext 에 선언된 @ImportResource 를 제거하고, XML 파일도 제거해줄 수 있습니다.

### 7.6.2 빈 스캐닝과 자동와이어링

#### @Autowired를 이용한 자동와이어링

@Autowired 는 스프링 컨테이너가 생성한 빈을 클래스의 멤버 필드로 주입받기 위해 사용하는 애노테이션 입니다.

@Autowired 는 자동와이어링 기법을 이용해서 족너에 맞는 빈을 찾아 자동으로 수정자 메소드나 필드에 넣어줍니다.

자동와이어링은 컨테이너가 타입이나 이름을 기준으로 주입될 빈을 찾아줍니다.

@Autowired 를 수정자 메소드에 붙이면 파라미터 타입을 보고 주입 가능한 타입의 빈을 찾습니다.

**Info:**<br>
@Autowired 는 먼저 타입을 기준으로 주입될 빈을 찾습니다.<br>
찾는 타입의 빈이 2개 이상인 경우 이름을 기준으로 빈을 선택합니다.<br>
만약 최종 후보를 찾지 못하면 에러를 발생 시킵니다.
{: .notice--info}

@Autowired 를 필드에 붙일 수도 있습니다.

이렇게 하면 수정자 메소드에서 파라미터로 빈을 넣어줄 필요가 없게 됩니다.

수정자 메소드가 실행되면 파라미터로 받은 빈 오브젝트를 필드에 저장하는데, 직접 필드에 빈 오브젝트를 넣도록 만든 것 입니다.

private 으로 선언된 필드라도 빈 오브젝트를 넣을 수 있습니다.

자바에서는 private 필드에는 클래스 외부에서 값을 넣을 수 없습니다.

하지만 스프링은 리플렉션 API 를 이용해 제약조건을 우회해서 값을 넣어줄 수 있습니다.

만약 빈 오브젝트를 그대로 저장하지 않는다면 수정자 메소드가 여전히 필요합니다.

```java
public class UserDaoJdbc implements UserDao {

  //highlight-next-line
  @Autowired
  private SqlService sqlService;
  
  private JdbcTemplate jdbcTemplate;
  
  //highlight-next-line
  @Autowired
  public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
  }
  
  // ...
}
```

스프링과 무관하게 직접 오브젝트를 생성하고 다른 오브젝트를 주입해서 테스트하는 단위 테스트를 만드는 경우에는 수정자 메소드가 필요합니다.

#### ©Component를 이용한 자동 빈 등록

@Component 는 클래스에 부여되고, @Component 가 붙은 클래스는 빈 스캐너에 의해 자동으로 빈으로 등록됩니다.

@Component 는 빈으로 등록될 후보 클래스에 붙여주는 일종의 마커 marker 입니다.

@Component 애노테이션을 단 클래스는 별도의 수정자 메소드 없이도 다른 빈에서 @Autowired 로 사용될 수 있습니다.

이를 위해서 빈 자동등록 기능을 사용하겠다는 정의가 필요합니다.

TestApplicationContext 에 @ComponentScan 을 추가해서 지정합니다.

```java
@Configuration
@EnableTransactionManagement
// highlight-next-line
@ComponentScan(basePackages="springbook.user")
public class TestApplicationContext {
  // ...
}
```

@Component 애노테이션이 달린 클래스를 찾기 위해 모든 클래스를 조회하는 것은 부담이 많이 갑니다.

그래서 @ComponentScan 을 선언할 때 기준이 되는 패키지를 정해서 특정 패키지 아래에서만 찾도록 합니다.

빈의 타입은 @Component 가 붙은 클래스이고, 빈의 아이디는 클래스의 이름의 첫 글자를 소문자로 바꿔서 사용합니다.

자동 빈 등록 기능을 사용하는 클래스에는 @Autowired 를 적용해야합니다.

왜냐하면 의존관계를 담은 프로퍼티를 따로 지정할 방법이 없기 때문입니다.

만약 @Component 가 붙은 클래스의 빈의 아이디를 다른 이름으로 지정해주고 싶다면 애노테이션에 이름을 지정해주면 됩니다.

```java
@Component("userDao")
```

여러 개의 애노테이션에 공통적인 속성을 부여하려면 메타 애노테이션을 이용할 수 있습니다.

메타 애노테이션은 애노테이션의 정의에 부여된 애노테이션입니다.

예컨데, 어떤 애노테이션에 빈 스캔을 통해 자동등록 대상으로 인식되는 기능을 부여하려면 그 애노테이션 정의에 @Component 를 메타 애노테이션으로 붙여주면 됩니다.

스프링에서는 DAO 기능을 제공하는 클래스에는 @Repository 애노테이션을 이용하도록 권장합니다.

@Repository 애노테이션을 살펴보면 @Component 를 메타 애노테이션으로 가지고 있습니다.

UserServiceImpl 클래스에 @Component 를 도입하면 오류가 발생합니다.

이유는 @Component 가 부여된 UserServiceImpl 클래스의 빈 아이디가 userServiceImpl 이 되었기 때문에 @Autowired 로 userService 빈을 받는 곳에서 UserService 타입의 2개 중에 어떤 빈을 선택할 수 없게 되었기 때문입니다.

**Info:**<br>
@Component 가 부여된 UserServiceImpl 클래스는 UserService 타입의 userServiceImpl 아이디를 가진 빈이 됩니다.<br>
빈 컨테이너에는 UserService 타입의 testUserService 아이디를 가진 빈과 userServiceImpl 아이디를 가진 빈 2개가 존재하게 됩니다.<br>
{: .notice--info}
```java
@Autowired
UserService userService
```
위 처럼 자동와이어링을 하는 경우 타입과 이름으로 빈을 찾게 되는데, 동일한 타입의 빈을 2개 찾았지만 이름과 동일한 아이디를 가진 빈을 찾지 못하게 됩니다.
{: .notice--info}

@Autowired 된 필드의 이름을 userServiceImpl 로 변경하거나, @Component("userService") 로 변경하면 빈을 지정할 수 있게 됩니다.

스프링에서는 비즈니스 로직을 담고 있는 서비스 계층의 클래스에는 @Service 애노테이션을 이용하도록 권장합니다.

서비스 계층은 트랜잭션 경계가 되는 곳이라서 @Transaction 이 함께 사용되는 경우가 많습니다.

### 7.6.3 컨텍스트분리와 @Import

1장부터 지금까지 해온 가장 대표적인 작업은, 성격이 다르고 변경 이유와 주기가 다른 코드를 분리해서 깔끔하게 다듬는 것이었습니다.

DI 정보도 성격에 따라 분리해봅니다.

#### 테스트용 컨텍스트 분리

testUserService 와 mailSender 빈은 테스트에서만 사용되는 빈입니다.

지금까지 만들었던 TestApplicationContext 클래스의 이름을 AppContext 로 변경합니다.

그리고 TestAppContext 를 새로 만들고 테스트에서만 사용되는 빈 설정 코드를 옮깁니다.

```java
@Configuration
public class TestAppContext {
  @Bean
  public UserService testUserService() {
    return new TestUserService();
  }
  
  @Bean
  public MailSender mailSender() {
    return new DummyMailSender();
  }
}
```

테스트용 빈을 사용하는 테스트 클래스의 @ContextConfiguration 애노테이션에 2개의 설정 클래스를 사용할 수 있도록 수정합니다.

```java
@ContextConfiguration(classes={TestAppContext.class, AppContext.class})
public class UserDaoTest {
  // ...
}
```

#### @Import

SqlService 의 구현 클래스와 이를 지원하는 보조 빈들은 독립적으로 변경될 가능성이 있기 때문에 다른 애플리케이션 구성과 분리하는 것이 좋습니다.

따라서 SqlService 와 관련된 빈들을 분리해 봅니다.

```java
// highlight-next-line
@Configuration
public class SqlServiceContext {
  @Bean
  public SqlService sqlService() {
    OxmSqlService sqlService = new OxmSqlService();
    sqlService.setUnmarshaller(unmarshaller());
    sqlService.setSqlRegistry(sqlRegistry());
    return sqlService;
  }
  
  @Bean
  public SqlRegistry sqlRegistry() {
    EmbeddedDbSqlRegistry sqlRegistry = new EmbeddedDbSqlRegistry();
    sqlRegistry.setDataSource(embeddedDatabase());
    return sqlRegistry;
  }
  
  @Bean
  public Unmarshaller unmarshaller() {
    Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
    marshaller.setContextPath("springbook.user.sqlservice.jaxb");
    return marshaller;
  }
  
  @Bean 
  public DataSource embeddedDatabase() {
    return new EmbeddedDatabaseBuilder()
      .setName("embeddedDatabase")
      .setType(HSQL)
      .addScript("classpath:springbook/user/sqlservice/updatable/sqlRegistrySchema.sql")
      .build();
  }
}
```

이렇게 하면 설정 클래스가 3개가 되었습니다.

SqlService 클래스는 애플리케이션 구동 시에 항상 필요합니다.

AppContext 에서 @Import 를 통해 설정정보에 포함시킬 수 있습니다.

```java
@Configuration
@EnableTransactionManagement
@ComponentScan(basePackages="springbook.user")
// highlight-next-line
@Import(SqlServiceContext.class)
public class AppContext {
  // ...
}
```

### 7.6.4 프로파일

MailSender 타입의 빈은 TestAppContext 에만 존재하기 때문에 실제 운영시에 에러가 발생합니다.

그렇다고 운영용 MailSender 빈을 만들고 AppContext 에 빈으로 추가하면 테스트에 문제가 발생합니다.

UserServiceTest 를 실행할 때 운영에 정의한 빈과 테스트용 빈이 충돌을 일으키기 때문입니다.

스프링은 빈 정보를 읽는 순서에 따라 뒤의 빈 설정이 앞의 빈 설정을 덮어쓰게 됩니다.

따라서 여기서는 TestAppContext 를 먼저 선언하고 AppContext 를 나중에 선언했기 때문에 운영용 MailSender 빈을 테스트할 때 사용하게 됩니다.

#### @Profile 과 @ActiveProfiles

@Profile 애노테이션을 이용해서 실행환경에 따라 설정 클래스를 선언해줄 수 있습니다.

@Profile("test") 애노테이션을 붙인 설정 클래스는 test 환경에서만 적용됩니다.

@Profile 을 지정하지 않은 설정 클래스는 디폴트 빈 설정정보로 취급되어 항상 적용됩니다.

프로파일을 적용하여 사용하기 위해서는 스프링 컨테이너를 실행할 때 @ActiveProfiles 애노테이션을 붙여서 어떤 프로파일로 구동되어야 하는지 명시할 수 있습니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
// highlight-next-line
@ActiveProfiles("test")
@ContextConfiguration(classes=AppContext.class)
public class UserServiceTest {
```

#### 컨테이너의 빈 등록 정보 확인

스프링 컨테이너는 모두 BeanFactory 라는 인터페이스를 구현하고 있습니다.

BeanFactory 의 구현 클래스 중에 DefaultListableBeanFactory 가 있고, 대부분의 스프링 컨테이너는 이 클래스를 이용해 빈을 등록하고 관리합니다.

DefaultListableBeanFactory 의 getBeanDefinitionNames() 메소드가 있어서 컨테이너에 등록된 빈 정보를 조회할 수 있습니다.

#### 중첩 클래스를 이용한 프로파일 적용

ProductionAppContext 와 TestAppContext 를 AppContext 의 중첩 클래스로 만들 수 있습니다.

```java
@Configuration
@EnableTransactionManagement
@ComponentScan(basePackages="springbook.user")
@EnableSqlService
@Import({
    SqlServiceContext.class, 
    AppContext.TestAppContext.class,
    AppContext.ProductionAppContext.class
})
public class AppContext {
  // ...
  
  @Configuration
  @Profile("production")
  // highlight-next-line
  public static class ProductionAppContext {
    // ...
  }
  
  @Configuration
  @Profile("test")
  // highlight-next-line
  public static class TestAppContext {
    // ...
  }
}
```

### 7.6.5 프로퍼티 소스

AppContext 에 정의된 dataSource 빈은 프로파일에 속하지 않은 빈 설정정보이므로 테스트와 운영 시점 모두 동일한 DB 연결정보를 가진 빈이 만들어집니다.

하지만 DB 연결정보는 환경에 따라 다르게 설정될 수 있어야 합니다.

#### @PropertySource

프로퍼티 파일의 확장자는 보통 properties 이고, 내부에 키=값 형태로 프로퍼티를 정의합니다.

```plain 
// title="database.properties"
db.driverClass=com.mysql.jdbc.Driver
db.url=jdbc:mysql://localhost/springbook?characterEncoding=UTF-8
db.username=spring
db.password=book
```

AppContext 의 dataSource() 메소드가 database.properties 파일의 내용을 가져와 DB 연결정보 프로퍼티에 넣어주도록 만들어본다.

컨테이너가 프로퍼티 값을 가져오는 대상을 프로퍼티 소스 property source 라고 합니다.

프로퍼티 소스 등록에는 @PropertySource 애노테이션을 이용합니다.

```java 
// title="AppContext.java"
@Configuration
@EnableTransactionManagement
@ComponentScan(basePackages="springbook.user")
@EnableSqlService
// highlight-next-line
@PropertySource("/database.properties")
public class AppContext {
```

@PropertySource 로 등록한 리소스로부터 가져오는 프로퍼티 값은 컨테이너가 관리하는 Environment 타입의 환경 오브젝트에 저장됩니다.

환경 오브젝트는 빈처럼 @Autowired 를 통해 필드로 주입받을 수 있습니다.

Environment 오브젝트의 getProperty() 메소드는 프로퍼티 이름을 파라미터로 받아 스트링 타입의 프로퍼티 값을 돌려줍니다.

```java 
// title="AppContext.java"
// ...
public class AppContext {
  
  @Autowired
  Environment env;
  
  @Bean
  public DataSource dataSource() {
    SimpleDriverDataSource ds = new SimpleDriverDataSource();
    
    // highlight-start
    try {
      ds.setDriverClass((Class<? extends java.sql.Driver>)Class.forName(env.getProperty("db.driverClass")));
    } catch(ClassNotFoundException e) {
      throw new RuntimeException(e);
    }
    ds.setUrl(env.getProperty("db.url"));
    ds.setUsername(env.getProperty("db.username"));
    ds.setPassword(env.getProperty("db.password"));
    // highlight-end
    
    return ds;
  }
}
```

#### PropertySourcesPlaceholderConfigurer

Environment 오브젝트 대신 프로퍼티 값을 직접 DI 받는 것도 가능합니다.

@Value 애노테이션을 사용하면 됩니다.

```java 
// title="AppContext.java"
// ...
public class AppContext {
  
  @Value("${db.driverClass}")
  Class<? extends Driver> driverClass;
  
  @Value("${db.url}") 
  String url;
  
  @Value("${db.username}") 
  String username;
  
  @Value("${db.password}") 
  String password;
  
  // highlight-start
  @Bean
  public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
    return new PropertySourcesPlaceholderConfigurer();
  }
  // highlight-end
  
  @Bean
  public DataSource dataSource() {
    SimpleDriverDataSource ds = new SimpleDriverDataSource();
    
    ds.setDriverClass(this.driverClass);
    ds.setUrl(this.url);
    ds.setUsername(this.username);
    ds.setPassword(this.password);
    
    return ds;
  }
  
  // ...
}
```

@Value 에 넣는 `${db.driverClass}` 는 XML 에서 `<property>` 의 value 에 사용하는 값 치환방식과 유사하기 때문에 치환자라고 불립니다.

@Value 가 붙은 필드의 값을 주입해주는 방식으로 동작합니다.

@Value 와 치환자로 프로퍼티 값을 필드에 주입하려면 PropertySourcesPlaceholderConfigurer 를 빈으로 정의해줘야 합니다.

이 빈 설정 메소드는 반드시 스태틱 메소드로 선언되어야 합니다.

@Value 를 이용하면 driverClass 처럼 문자열을 그대로 사용하지 않고 타입 변환이 필요한 프로퍼티를 스프링이 알아서 처리해줍니다.

### 7.6.6 빈 설정의 재사용과 @Enable*

SqlServiceContext 는 SQL 서비스와 관련된 빈 설정정보를 담고 있습니다.

SQL 서비스를 라이브러리 모듈로 뽑아내서 독립적으로 관리하면서 여러 프로젝트에서 사용하게 할 수도 있습니다.

#### 빈 설정자

OxmSqlService 의 내부 클래스인 OxmSqlReader 에는 매핑 내역을 담은 파일의 위치를 지정하는 부분이 있습니다.

애플리케이션에 따라 SQL 매핑파일 이름이나 위치한 패키지를 변경할 수 있게 하기 위해서는 SQL 매핑 리소스를 빈 클래스 외부에서 설정할 수 있어야 합니다.

SqlMapConfig 인터페이스를 정의하고 SQL 매핑파일의 리소스를 돌려주는 getSqlMapResource() 메소드를 추가합니다.

```java 
// title="SqlMapConfig.java"
public interface SqlMapConfig {
  Resource getSqlMapResouce();
}
```

SqlMapConfig 구현 클래스를 만듭니다.

```java 
// title="UserSqlMapConfig.java"
public class UserSqlMapConfig implements SqlMapConfig {
  @Override
  public Resource getSqlMapResource() {
    return new ClassPathResource("sqlmap.xml", UserDao.class);
  }
}
```

그리고 SqlServiceContext 는 SqlMapConfig 인터페이스에 의존하게 만들고, SqlMapConfig 구현 클래스는 빈으로 정의해 런타임 시 주입되게 만드는 것입니다.

```java 
// title="SqlServiceContext.java"
@Configuration
public class SqlServiceContext {
  @Autowired
  SqlMapConfig sqlMapConfig;
  // ...
}
```

이렇게 하여 SQL 서비스 기능을 사용하려는 애플리케이션은 SqlMapConfig 의 구현 클래스를 만들어 원하는 리소스로부터 SQL 매핑정보를 가져오게 할 수 있습니다.

설정정보를 담은 코드도 리팩토링하면 반복적으로 사용되는 부분은 수정 없이 재사용될 수 있고, 적용환경에 따라 바뀌는 부분은 인터페이스로 분리하고 DI 를 통해 외부에서 주입할 수 있습니다.

@Configuration 애노테이션이 달린 AppContext 같은 클래스도 스프링에서 하나의 빈으로 취급됩니다.

그래서 빈의 자동와이어링에 쓰는 @Autowired 를 이용할 수 있습니다.

@Configuration 은 @Component 를 메타 애노테이션으로 갖고 있기 때문에 @Configuration 이 붙은 클래스도 빈 스캐너에 의해 자동등록 시킬 수 있습니다.

하나의 빈이 꼭 한 가지 타입일 필요는 없습니다.

빈을 DI 받아서 사용하는 쪽은 빈이 특정 인터페이스를 구현하고 있는지만 중요합니다.

AppContext 가 직접 SqlMapConfig 인터페이스를 구현할 수도 있습니다.

AppContext 가 SqlMapConfig 의 구현클래스가 되어 빈으로 만들어봅니다.

```java 
// title="AppContext.java"
public class AppContext implements SqlMapConfig {
  
  @Override
  public Resource getSqlMapResouce() {
    return new ClassPathResource("sqlmap.xml", UserDao.class);
  }
  
  // ...
  
}
```

SqlServiceContext 는 @Autowired 로 SqlMapConfig 타입의 빈을 주입받습니다.

그러면 AppContext 로 만들어진 빈 오브젝트는 SqlServiceContext 에 주입되어 사용됩니다.

#### ©Enable* 애노테이션

SQL 서비스가 필요한 애플리케이션은 메인 설정 클래스에서 @Import 로 SqlServiceContext 빈 설정을 추가하고 SqlMapConfig 를 구현해 SQL 매핑파일의 위치를 지정해주기만 하면 됩니다.

스프링에서는 모듈화된 빈 설정을 가져올 때 사용하는 @Import 를 다른 애노테이션으로 대체할 수도 있습니다.

SqlService 를 사용하겠다는 의미로 @EnableService 라는 애노테이션을 만들고 @Import 를 메타 애노테이션으로 등록합니다.

```java 
// title="EnableSqlService.java"
@Import(value=SqlServiceContext.class)
public @interface EnableSqlService {
}
```

@Enable 로 시작하는 애노테이션은 @Import 를 메타 애노테이션으로 갖고 있습니다.

@EnableSqlService 로 애노테이션을 만들면 SQL 서비스를 사용하겠다는 의미가 잘 드러나게 됩니다.

또한, 애노테이션을 정의할 때 엘리먼트를 넣어서 옵션을 지정하게 할 수도 있습니다.

지금까지 XML 에 담겨 있던 빈 설정정보를 애노테이션과 자바 코드로 바꾸고, 자동등록과 와이어링 기능을 적용하고, 실행환경에 따라 달라지는 빈 설정정보를 분리해서 프로파일을 적용했습니다.

## 7.7 정리

- SQL 처럼 변경될 수 있는 텍스트로 된 정보는 외부 리소스에 담아두고 가져오게 만들면 편리하다.
- 성격이 다른 코드가 한데 섞여 있는 클래스라면 먼저 인터페이스를 정의해서 코드를 각 인터페이스별로 분리하는 게 좋다. 다른 인터페이스에 속한 기능은 인터페이스를 통해 접근하게 만들고, 간단히 자기참조 빈으로 의존관계를 만들어 검증한다. 검증을 마쳤으면 아예 클래스를 분리해도 좋다.
- 자주 사용되는 의존 오브젝트는 디폴트로 미리 정의해두면 편리하다.
- XML 과 오브젝트 매핑은 스프링의 OXM 추상화 기능을 활용한다.
- 특정 의존 오브젝트를 고정시켜 기능을 특화하려면 멤버 클래스로 만드는 것이 편리하다. 기존에 만들어진 기능과 중복되는 부분은 위임을 통해 중복을 제거하는 게 좋다.
- 외부의 파일이나 리소스를 사용하는 코드에서는 스프링의 리소스 추상화와 리소스 로더를 사용한다.
- DI 를 의식하면서 코드를 작성하면 객체지향 설계에 도움이 된다.
- DI 에는 인터페이스를 사용한다. 인터페이스를 사용하면 인터페이스 분리 원칙을 잘 지키는데도 도움이 된다.
- 클라이언트에 따라서 인터페이스를 분리할 때, 새로운 인터페이스를 만드는 방법과 인터페이스를 상속하는 방법 두 가지를 사용할 수 있다.
- 애플리케이션에 내장하는 DB를 사용할 때는 스프링의 내장형 DB 추상화 기능과 전용 태그를 사용하면 편리하다.
