---
# layout: post
title: "7장 스프링 핵심 기술의 응용 - 7.5 DI를 이용해 다양한 구현 방법 적용하기"
# excerpt: "DAO 에 트랜잭션을 적용해보면서 스프링이 어떻게 성격이 비슷한 여러 기술을 추상화하고, 일관된 방법으로 사용할 수 있도록 지원하는지 알아봅니다."
categories:
  - "토비의 스프링 3.1"
tags:
  - "스프링 핵심 기술의 응용"
---

## 7.5 DI를 이용해 다양한 구현 방법 적용하기

운영 중인 시스템에서 사용하는 정보를 실시간으로 변경하는 작업을 만들 때 동시성 문제가 가장 먼저 고려되어야 합니다.

SqlRegistry 가 읽기전용으로 동작할 때는 동시성 문제가 발생할 일이 없습니다.

하지만 수정은 문제가 다릅니다.

자바에서 제공되는 주요 기술을 이용해 간단한 방식으로 어느 정도 안전한 업데이트가 가능한 SQL 레지스트리를 구현해보겠습니다.

### 7.5.1 ConcurrentHashMap을 이용한 수정 가능 SQL 레지스트리

HashMap 으로는 멀티스레드 환경에서 동시에 수정을 시도하거나 수정과 동시에 요청하는 경우 예상하지 못한 결과가 발생할 수 있습니다.

멀티스레드 환경에서 안전하게 HashMap 을 조작하려면 Collections.synchronizedMap() 등을 이용해 외부에서 동기화해줘야 합니다.

하지만 이런 방식은 고성능 서비스에서는 성능에 문제가 발생합니다.

따라서 동기화된 해시 데이터 조작에 최적화된 ConcurrentHashMap 을 사용하는 방법이 일반적으로 권장됩니다.

ConcurrentHashMap 은 데이터 조작 시 전체 데이터에 대해 락을 걸지 않고, 조회는 락을 아예 사용하지 않습니다.

#### 수정 가능 SQL 레지스트리 테스트

ConcurrentHashMap 을 이용해 UpdatableSqlRegistry 를 구현해봅니다.

SQL 등록한 것이 잘 조회되는지와, 이를 수정한 후에 수정된 SQL 이 바르게 적용되는지를 확인하는 단위테스트를 만듭니다.

```java 
// title="ConcurrentHashMapSqlRegistryTest.java"
public class ConcurrentHashMapSqlRegistryTest {
  UpdatableSqlRegistry sqlRegistry;
  
  @Before
  public void setUp() {
    sqlRegistry = new ConcurrentHashMapSqlRegistry();
    // 초기 SQL 정보
    // highlight-start
    sqlRegistry.registerSql("KEY1", "SQL1");
    sqlRegistry.registerSql("KEY2", "SQL2");
    sqlRegistry.registerSql("KEY3", "SQL3");
    // highlight-end
  }
  
  @Test
  public void find() {
    checkFindResult("SQL1", "SQL2", "SQL3");
  }

  private void checkFindResult(String expected1, String expected2, String expected3) {
    assertThat(sqlRegistry.findSql("KEY1"), is(expected1));    
    assertThat(sqlRegistry.findSql("KEY2"), is(expected2));    
    assertThat(sqlRegistry.findSql("KEY3"), is(expected3));    
  }
  
  // 예외상황에 대한 테스트는 항상 의식적으로 넣으려고 노력해야 합니다.
  // highlight-start
  @Test(expected= SqlNotFoundException.class)
  public void unknownKey() {
    sqlRegistry.findSql("SQL9999!@#$");
  }
  // highlight-end
      
  
  @Test
  public void updateSingle() {
    sqlRegistry.updateSql("KEY2", "Modified2");    
    checkFindResult("SQL1", "Modified2", "SQL3");
  }
  
  @Test
  public void updateMulti() {
    Map<String, String> sqlmap = new HashMap<String, String>();
    sqlmap.put("KEY1", "Modified1");
    sqlmap.put("KEY3", "Modified3");
    
    sqlRegistry.updateSql(sqlmap);    
    checkFindResult("Modified1", "SQL2", "Modified3");
  }

  @Test(expected=SqlUpdateFailureException.class)
  public void updateWithNotExistingKey() {
    sqlRegistry.updateSql("SQL9999!@#$", "Modified2");
  }
}
```

동시성에 대한 부분을 테스트하면 좋겠지만 간단하지 않기 때문에 일단 수정 기능을 검증하는 테스트만 만듭니다.

코드와 테스트를 만들어 검증하는 간격을 가능한 짧게하고, 예외상황을 포함한 기능을 세세하게 검증하도록 테스트를 만드는 것이 중요합니다.

#### 수정 가능 SQL 레지스트리 구현

ConcurrentHashMapSqlRegistry 를 만들어봅니다.

```java 
// title="ConcurrentHashMapSqlRegistry.java"
public class ConcurrentHashMapSqlRegistry implements UpdatableSqlRegistry {

  // highlight-next-line
  private Map<String, String> sqlMap = new ConcurrentHashMap<String, String>();

  public String findSql(String key) throws SqlNotFoundException {
    String sql = sqlMap.get(key);
    if (sql == null)  throw new SqlNotFoundException(key + "를 이용해서 SQL을 찾을 수 없습니다");
    else return sql;
  }

  public void registerSql(String key, String sql) { sqlMap.put(key, sql);  }

  public void updateSql(String key, String sql) throws SqlUpdateFailureException {
    if (sqlMap.get(key) == null) {
      throw new SqlUpdateFailureException(key + "에 해당하는 SQL을 찾을 수 없습니다");
    }
    
    sqlMap.put(key, sql);
  }

  public void updateSql(Map<String, String> sqlmap) throws SqlUpdateFailureException {
    for(Map.Entry<String, String> entry : sqlmap.entrySet()) {
      updateSql(entry.getKey(), entry.getValue());
    }
  }
  
}
```

OxmSqlService 는 sqlRegistry 를 지정하지 않으면 HashMapSqlRegistry 를 기본으로 사용하도록 되어 있습니다.

OxmSqlService 가 새로 만든 ConcurrentHashMapSqlRegistry 빈을 사용하도록 설정을 변경합니다.

```xml 
// title="test-applicationContext.xml"
<beans xmlns="...">

  <bean id="sqlService" class="springbook.user.sqlservice.OxmSqlService">
    <property name="unmarshaller" ref="unmarshaller" />
    // highlight-next-line
    <property name="sqlRegistry" ref="sqlRegistry" />
  </bean>
  
  // highlight-start
  <bean id="sqlRegistry" class="springbook.user.sqlservice.updatable.ConcurrentHashMapSqlRegistry">
  </bean>
  // highlight-end
  
</beans>
```

ConcurrentHashMapSqlRegistry 와 OxmSqlService 가 서로 협력하여 SqlService 기능을 제공하는데 이상이 없는지 확인할 수 있습니다.

### 7.5.2 내장형 데이터베이스를 이용한 SQL 레지스트리 만들기

이번에는 ConcurrentHashMapSqlRegistry 대신 내장형 DB embedded DB 를 이용해 SQL 을 저장하고 수정하도록 해보겠습니다.

내장형 DB 는 애플리케이션에 내장되어 애플리케이션과 함께 시작되고 종료되는 DB 를 말합니다.

데이터가 메모리에 저장되기 때문에 성능이 뛰어나고, Map 같은 컬렉션, 오브젝트를 이용한 방법에 비해 안정적으로 등록, 수정, 검색이 가능합니다.

또한 락킹, 격리수준, 트랜잭션을 적용할 수도 있습니다.

#### 스프링의 내장형 DB 지원 기능

자바에서 많이 사용되는 내장형 DB 는 Derby, HSQL, H2 를 꼽을 수 있습니다.

내장형 DB 는 애플리케이션 내에서 DB 를 생성하고 테이블을 만드는 초기화 작업이 필요합니다.

스프링에서는 내장형 DB 를 초기화하는 내장형 DB 빌더를 제공합니다.

#### 내장형 DB 빌더 학습 테스트

스프링의 내장형 DB 지원 기능이 동작하는 방법을 보기 위해 학습테스트를 만듭니다.

테이블을 생성하는 schema.sql 과 데이터를 등록하는 data.sql 을 각각 만듭니다.

```text 
-- title="schema.sql"
CREATE TABLE SQLMAP (
  KEY_ VARCHAR(100) PRIMARY KEY,
  SQL_ VARCHAR(100) NOT NULL
);
```

```text 
-- title="data.sql"
INSERT INTO SQLMAP(KEY_, SQL_) values('KEY1', 'SQL1');
INSERT INTO SQLMAP(KEY_, SQL_) values('KEY2', 'SQL2');
```

이제 테스트를 만들어서 EmbeddedDataBuilder 가 어떻게 동작하는지 확인합니다.

```java 
// title="EmbeddedDbTest.java"
public class EmbeddedDbTest {

  EmbeddedDatabase db;
  SimpleJdbcTemplate template;
  
  @Before
  public void setUp() {
    // highlight-start
    db = new EmbeddedDatabaseBuilder()
      .setType(HSQL)      
      .addScript("classpath:/springbook/learningtest/spring/embeddeddb/schema.sql") 
      .addScript("classpath:/springbook/learningtest/spring/embeddeddb/data.sql")
      .build();
    // highlight-end
    
    template = new SimpleJdbcTemplate(db); 
  }
  
  // highlight-start
  @After
  public void tearDown() {
    db.shutdown();
  }
  // highlight-end
  
  @Test
  public void initData() {
    assertThat(template.queryForInt("select count(*) from sqlmap"), is(2));
    
    List<Map<String,Object>> list = template.queryForList("select * from sqlmap order by key_");
    assertThat((String)list.get(0).get("key_"), is("KEY1"));
    assertThat((String)list.get(0).get("sql_"), is("SQL1"));
    assertThat((String)list.get(1).get("key_"), is("KEY2"));
    assertThat((String)list.get(1).get("sql_"), is("SQL2"));
  }
  
  @Test
  public void insert() {
    template.update("insert into sqlmap(key_, sql_) values(?, ?)", "KEY3", "SQL3");
    
    assertThat(template.queryForInt("select count(*) from sqlmap"), is(3));
  }
  
}
```

#### 내장형 DB를 이용한 SqlRegistry 만들기

학습테스트에서 살펴본 것 처럼 스프링에서 내장형 DB 를 사용하려면 EmbeddedDatabaseBuilder 를 사용하면 됩니다.

EmbeddedDatabaseBuilder 를 사용하기 위해서는 초기화 코드가 필요합니다.

초기화 코드가 필요할 때는 팩토리 빈으로 만들어주면 좋습니다.

EmbeddedDatabaseBuilder 를 활용해서 EmbeddedDatabase 타입의 오브젝트를 생성해주는 팩토리 빈을 만들어야 합니다.

스프링에는 팩토리 빈을 만드는 작업을 대신해주는 전용 태그인 `<jdbc:embedded-database>`가 있습니다.

```xml 
// title="HSQL 내장형 DB 설정 예"
<jdbc:embedded-database id="embeddedDatabase" type="HSQL">
  <jdbc:script location="classpath:schema.sql"/>
</jdbc:embedded-database>
```

EmbeddedDatabase 타입의 embeddedDatabase 아이디를 가진 빈이 dataSource 로 등록됩니다.

```java 
// title="EmbeddedDbSqlRegistry.java"
public class EmbeddedDbSqlRegistry implements UpdatableSqlRegistry {

  SimpleJdbcTemplate jdbc;
  
  // highlight-start
  public void setDataSource(DataSource dataSource) {
    jdbc = new SimpleJdbcTemplate(dataSource);
  }
  // highlight-end
  
  // ...
}
```

내장형 DB 를 사용하기 위해서 DataSource 타입의 오브젝트를 주입받은 수정자 부분을 봅니다.

정의한 빈의 타입은 EmbeddedDatabase 타입인데, DataSource 타입으로 DI 받고 있습니다.

물론 EmbeddedDatabase 인터페이스는 DataSource 를 상속한 인터페이스입니다.

**Info:**<br>
클라이언트는 자신이 필요로 하는 기능을 가진 인터페이스를 통해 의존 오브젝트를 DI 하는 것이 가장 바람직합니다.<br>
따라서 DB 종료기능을 가진 EmbeddedDatabase 대신 DataSource 을 사용한 것입니다.
{: .notice--info}

#### UpdatableSqlRegistry 테스트 코드의 재사용

ConcurrentHashMapSqlRegistry 와 EmbeddedDbSqlRegistry 둘 다 UpdatableSqlRegistry 인터페이스를 구현하고 있습니다.

인터페이스가 같은 구현클래스라고 하더라도 구현 방식에 따라 검증 내용이나 테스트 방법이 달라질 수 있고, 의존 오브젝트의 구성에 따라 목이나 스텁을 이용하기도 합니다.

하지만 DAO 는 DB 까지 연동하는 테스트를 하는 편이 효과적이고 DataSource 를 테스트 대역을 쓰기는 어렵습니다.

따라서 ConcurrentHashMapSqlRegistry 와 EmbeddedDbSqlRegistry 둘 다 테스트 방법이 동일할 것으로 보입니다.

기존에 만들었던 ConcurrentHashMapSqlRegistryTest 의 테스트 코드를 EmbeddedDbSqlRegistryTest 에서 공유하는 방법을 찾아 봅니다.

```java 
// title="ConcurrentHashMapSqlRegistryTest.java"
public class ConcurrentHashMapSqlRegistryTest {
  
  UpdatableSqlRegistry sqlRegistry;
  
  @Before
  public void setUp() {
    // highlight-next-line
    sqlRegistry = new ConcurrentHashMapSqlRegistry();
    // ...
  }
  
  // ...
  
}
```

ConcurrentHashMapSqlRegistryTest 를 보면 UpdatableSqlRegistry 구현클래스를 생성하는 부분만 분리하면 나머지 테스트 코드 공유가 가능합니다.

따라서 상속을 통해 테스트 코드를 공유하도록 만듭니다.

```java 
// title="AbstractUpdatableSqlRegistryTest.java"
public abstract class AbstractUpdatableSqlRegistryTest {

  UpdatableSqlRegistry sqlRegistry;
  
  @Before
  public void setUp() {
    // highlight-next-line
    sqlRegistry = createUpdatableSqlRegistry();
  }
  
  abstract protected UpdatableSqlRegistry createUpdatableSqlRegistry();
  
  // 기존의 테스트 코드들
  // ...
  
}
```

추상클래스를 만들고, 상속할 수 있도록 합니다.

UpdatableSqlRegistry 구현클래스는 AbstractUpdatableSqlRegistryTest 를 상속받은 클래스에서 생성하도록 합니다.

```java 
// title="ConcurrentHashMapSqlRegistryTest.java"
public class ConcurrentHashMapSqlRegistryTest extends AbstractUpdatableSqlRegistryTest {

  protected UpdatableSqlRegistry createUpdatableSqlRegistry() {
    // highlight-next-line
    return new ConcurrentHashMapSqlRegistry();
  }
  
}
```

```java 
// title="EmbeddedDbSqlRegistryTest.java"
public class EmbeddedDbSqlRegistryTest extends AbstractUpdatableSqlRegistryTest {
  EmbeddedDatabase db;
  
  @Override
  protected UpdatableSqlRegistry createUpdatableSqlRegistry() {
    // highlight-start
    db = new EmbeddedDatabaseBuilder()
      .setType(HSQL)
      .addScript("classpath:springbook/user/sqlservice/updatable/sqlRegistrySchema.sql")
      .build();
    
    EmbeddedDbSqlRegistry embeddedDbSqlRegistry = new EmbeddedDbSqlRegistry();
    embeddedDbSqlRegistry.setDataSource(db);
    
    return embeddedDbSqlRegistry;
    // highlight-end
  }
  
  @After
  public void tearDown() {
    db.shutdown();
  }
}
```

#### XML 설정을 통한 내장형 DB의 생성과 적용

내장형 DB 를 등록하는 방법은 jdbc 스키마의 전용 태그를 사용하는 것이 편합니다.

```xml 
// title="test-applicationContext.xml"
<beans xmlns="...">
  // ...
  
  <bean id="sqlRegistry" class="springbook.user.sqlservice.updatable.EmbeddedDbSqlRegistry">
    // highlight-next-line
    <property name="dataSource" ref="embeddedDatabase" />
  </bean>
  
  // highlight-next-line
  <jdbc:embedded-database id="embeddedDatabase" type="HSQL">
    <jdbc:script location="classpath:springbook/user/sqlservice/updatable/sqlRegistrySchema.sql"/>
  </jdbc:embedded-database>
  
</beans>
```

`<jdbc:embedded-database>` 태그로 만들어지는 EmbeddedDatabase 타입 빈은 스프링 컨테이너가 종료될 때 자동으로 shutdown() 메소드가 호출되도록 설정되기 때문에 별도의 종료코드가 필요하지 않습니다.

### 7.5.3 트랜잭션 적용

여러 개의 SQL 을 수정하는 작업은 반드시 트랜잭션 안에서 일어나야 합니다.

HashMap 과 같은 컬렉션은 트랜잭션 개념을 적용하기 어렵기 때문에 내장형 DB 를 도입하였습니다.

스프링에서 트랜잭션 적용을 위해 AOP 를 이용하는 것이 편리합니다.

하지만 제한된 오브젝트 내에서의 간단한 트랜잭션의 경우 트랜잭션 추상화 API 를 사용하는 편이 좋습니다.

#### 다중 SQL 수정에 대한 트랜잭션 테스트

트랜잭션이 적용되면 성공하고 아니면 실패하는 테스트를 만듭니다.

```java 
// title="EmbeddedDbSqlRegistryTest.java"
public class EmbeddedDbSqlRegistryTest extends AbstractUpdatableSqlRegistryTest {
  EmbeddedDatabase db;
  
  // ...

  @Test
  public void transactionalUpdate() {
    checkFind("SQL1", "SQL2", "SQL3");
    
    Map<String, String> sqlmap = new HashMap<String, String>();
    sqlmap.put("KEY1", "Modified1");
    // highlight-next-line
    sqlmap.put("KEY9999!@#$", "Modified9999");
    
    try {
       // highlight-next-line
      sqlRegistry.updateSql(sqlmap);
      fail();
    }
    catch(SqlUpdateFailureException e) {}
    
    checkFind("SQL1", "SQL2", "SQL3");
  }

}
```

'KEY9999!@#$' 라는 존재하지 않는 키를 지정했기 때문에 테스트는 실패합니다.

실패한 경우 'KEY1' 에 적용된 'Modified1' 이 롤백되어야 하지만 롤백되지 않았기 때문입니다.

#### 코드를 이용한 트랜잭션 적용

코드에 TransactionTemplate 을 사용하여 트랜잭션을 적용하도록 합니다.

```java 
// title="EmbeddedDbSqlRegistry.java"
public class EmbeddedDbSqlRegistry implements UpdatableSqlRegistry {

  SimpleJdbcTemplate jdbc;
  // highlight-next-line
  TransactionTemplate transactionTemplate;
  
  public void setDataSource(DataSource dataSource) {
    jdbc = new SimpleJdbcTemplate(dataSource);
    // highlight-start
    transactionTemplate = new TransactionTemplate(
        new DataSourceTransactionManager(dataSource));
    // highlight-end
  }
  
  // ...

  public void updateSql(final Map<String, String> sqlmap) throws SqlUpdateFailureException {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      protected void doInTransactionWithoutResult(TransactionStatus status) {
        // highlight-start
        for(Map.Entry<String, String> entry : sqlmap.entrySet()) {
          updateSql(entry.getKey(), entry.getValue());
        }
        // highlight-end
      }
    });
  }
  
}
```

트랜잭션으로 동작할 코드를 콜백 형태로 전달하여 수행합니다.
