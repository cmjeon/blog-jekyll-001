---
# layout: post
title: "7장 스프링 핵심 기술의 응용 - 7.2 인터페이스의 분리와 자기참조 빈"
# excerpt: "DAO 에 트랜잭션을 적용해보면서 스프링이 어떻게 성격이 비슷한 여러 기술을 추상화하고, 일관된 방법으로 사용할 수 있도록 지원하는지 알아봅니다."
categories:
  - "토비의 스프링 3.1"
tags:
  - "스프링 핵심 기술의 응용"
---

## 7.2 인터페이스의 분리와 자기참조 빈

### 7.2.1 XML 파일 매핑

SQL 정보는 설정정보에 있기 보다는 SQL 을 저장해두는 전용 포맷을 가진 독립적인 파일을 이용하는 편이 바람직합니다.

SQL 정보를 담는 XML 문서를 만들고, 이 파일에서 SQL 을 읽어뒀다가 DAO 에 제공해주는 SQL 서비스 구현 클래스를 만들어 보겠습니다.

#### JAXB

JAXB(Jav Architecture for XML Binding) 을 이용하여 파일을 읽어오도록 합니다.

JAXB 는 클래스를 만들 수 있는 컴파일러를 제공해주는데, 이 컴파일러에 스키마를 이용하면 클래스를 만들 수 있습니다.

이 클래스로 XML 파일의 정보를 매핑할 오브젝트를 생성하고, 오브젝트에는 매핑정보가 애노테이션으로 담기게 됩니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-1.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-1.png)

#### SQL 맵을 위한 스키마 작성과 컴파일

SQL 정보를 담은 XML 문서를 만듭니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sqlmap ... >
  // highlight-next-line
  <sql key="userAdd">insert into users (...) ...</sql>
  <sql key="userGet">...</sql>
</sqlmap>
```

그리고 XML 문서 구조를 정의하고 있는 스키마 파일을 만듭니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema ...>
  <element name="sqlmap">
    <complexType>
      <sequence>
        // highlight-next-line
        <element name="sql" maxOccurs="unbounded" type="tns:sqlType" />
      </sequence>
    </complexType>
  </element>
  <complexType name="sqlType">
    <simpleContent>
      <extension base="string">
        // highlight-next-line
        <attrivute name="key" use="required" type="string" />
      </extension>
    </simpleContent>
  </complexType>
</schema>
```

스키마 파일을 JAXB 컴파일러에 이용해서 컴파일하면 클래스파일이 생성됩니다.

팩토리 클래스와 2개의 바인딩용 클래스가 생성됩니다.

```shell
# 팩토리 클래스
ObjectFactory.java
# 바인딩용 클래스
SqlType.java
Sqlmap.java
```

이 클래스를 이용해 오브젝트를 생성하면 XML 정보가 오브젝트로 전환됩니다.

#### 언마샬링

생성된 클래스를 이용하기 전에 JAXB API 학습테스트를 합니다.

XML 문서를 읽어서 자바의 오브젝트로 변환하는 것을 JAXB 에서는 언마샬링 unmarshalling 이라고 부릅니다.

반대로 오브젝트를 XML 문서로 변환하는 것을 마샬링 marshalling 이라고 합니다.

오브젝트를 바이트 스트림으로 바꾸는 것을 직렬화 serialization 이라고 부르는 것과 비슷합니다.

```java
public class JaxbTest {

  @Test
  public void readSqlmap() throws JAXBException, IOException {
    
    String contextPath = Sqlmap.class.getPackage().getName(); 
    JAXBContext context = JAXBContext.newInstance(contextPath);
    Unmarshaller unmarshaller = context.createUnmarshaller();
    
    // sqlmap.xml 파일을 언마샬링해서 Sqlmap 의 오브젝트로 돌려준다.
    // highlight-start
    Sqlmap sqlmap = (Sqlmap) unmarshaller.unmarshal(
        getClass().getResourceAsStream("sqlmap.xml"));
    // highlight-end
    
    // highlight-next-line
    List<SqlType> sqlList = sqlmap.getSql();

    // assertions ...
  }
  
}
```

### 7.2.2 XML 파일을 이용하는 SQL 서비스

#### SQL 맵 XML 파일

SQL 문서는 DAO 의 로직의 일부라고 볼 수 있으므로 DAO 와 같은 패키지에 두는 게 좋습니다.

#### XML SQL 서비스

sqlmap.xml 에 있는 SQL 을 가져와 DAO 에 제공해주는 SqlService 구현클래스를 만들어봅니다.

JAXB 로 XML 문서를 언마샬링하면 `<sql>` 태그 하나는 Sql 클래스의 오브젝트에 하나씩 담깁니다.

생성자에서 JAXB 를 이용해 XML 로 된 SQL 문서를 읽어드리고, 변환된 Sql 오브젝트들을 맵으로 옮겨서 저장합니다.

그리고 DAO 의 요청에 따라 SQL 을 찾아서 반환합니다.

```java
public class XmlSqlService implements SqlService {
  
  // SQL 을 저장하는 HashMap
  // highlight-next-line
  private Map<String, String> sqlMap = new HashMap<String, String>();

  public XmlSqlService() {
    
    // highlight-start
    String contextPath = Sqlmap.class.getPackage().getName(); 
    try {
      JAXBContext context = JAXBContext.newInstance(contextPath);
      Unmarshaller unmarshaller = context.createUnmarshaller();
      InputStream is = UserDao.class.getResourceAsStream("sqlmap.xml");
      // highlight-end
      // unmarshaller 로 sqlmap 파일을 언마샬링 한다.
      Sqlmap sqlmap = (Sqlmap)unmarshaller.unmarshal(is);

      for(SqlType sql : sqlmap.getSql()) {
        sqlMap.put(sql.getKey(), sql.getValue());
      }
    } catch (JAXBException e) {
      throw new RuntimeException(e);
    } 
  }

  public String getSql(String key) throws SqlRetrievalFailureException {
    // highlight-next-line
    String sql = sqlMap.get(key);
    if (sql == null)  
      throw new SqlRetrievalFailureException(key + "를 이용해서 SQL을 찾을 수 없습니다");
    else
      return sql;
  }
}
```

이렇게 SQL 문장을 스프링의 빈 설정정보로부터 분리하는데 성공했습니다.

### 7.2.3 빈의 초기화 작업

XmlSqlService 코드를 살펴보면 몇 가지 개선점이 보입니다.

예외가 발생할 수도 있는 복잡한 초기화 작업을 생성자에서 다루는 것은 좋지 않습니다.

이럴 때는 초기 상태의 오브젝트를 만든 후, 별도의 초기화 메소드를 사용하는 것이 바람직합니다.

또한 SQL 을 담은 XML 파일의 위치가 코드에 고정되는 것도 좋지 않습니다.

바뀔 가능성이 있는 내용이기 때문에 외부에서 DI 로 설정해줄 수 있게 만들어야 합니다.

```java
public class XmlSqlService implements SqlService {

  // XML 파일의 위치를 DI 받을 수 있게 변경
  // highlight-start 
  public void setSqlmapFile(String sqlmapFile) {
    this.sqlmapFile = sqlmapFile;
  }
  // highlight-end
  
  // 초기화 메소드를 선언
  public void loadSql() {
    String contextPath = Sqlmap.class.getPackage().getName(); 
    try {
      // ...
      // highlight-next-line
      InputStream is = UserDao.class.getResourceAsStream(this.sqlmapFile);
      // ...
    } catch (JAXBException e) {
      throw new RuntimeException(e);
    } 
  }

}
```

XmlSqlService 오브젝트는 빈이기 때문에 제어권이 스프링에 있습니다.

빈 후처리기는 스프링 컨테이너가 빈을 생성한 뒤 부가적인 작업을 수행할 수 있게 해주는 기능입니다.

AOP 를 위한 프록시 자동생성기가 대표적인 빈 후처리기 입니다.

context 네임스페이스를 사용해서 `<context:annotation-config/>` 태그를 만들어서 설정파일에 넣어주면 빈 설정 기능에 사용할 수 있는 특별한 애노테이션 기능을 부여해주는 빈 후처리기들이 등록됩니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
            ...
            http://www.springframework.org/schema/context 
            http://www.springframework.org/schema/context/spring-context-3.0.xsd
            ...">
  
  // 코드의 애노테이션을 이용한 빈 설정 및 초기화 작업을 해주는 빈 후처리기 등록
  // highlight-next-line          
  <context:annotation-config />             
            
  // @Transactional 이 붙은 타입과 메소드에 트랜잭션 부가기능을 담은 프록시를 추가하도록 만들어주는 후처리기 등록
  // highlight-next-line
  <tx:annotation-driven /> 
  
</beans>
```

스프링은 @PostConstruct 애노테이션을 빈 오브젝트의 초기화 메소드를 지정하는 데 사용합니다.

@PostConstruct 애노테이션을 초기화 작업을 수행할 메소드에 부여해줍니다.

```java
public class XmlSqlService implements SqlService {

  // 초기화 메소드를 선언
  // highlight-next-line
  @PostConstruct
  public void loadSql() {
    // ... 
  }
  
}
```

스프링은 XmlSqlService 클래스로 등록된 빈의 오브젝트를 생성하고 DI 작업을 마친 뒤 @PostConstruct 가 붙은 메소드를 자동으로 실행해줍니다.

프로퍼티까지 모두 준비된 후에 실행된다는 면에서 @PostConstruct 초기화 메소드는 매우 유용합니다.

이제 sqlmapFile 프로퍼티의 값을 sqlService 빈의 설정에 넣어줍니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ... >
  // ...
  // highlight-start
  <bean id="sqlService" class="springbook.user.sqlservice.XmlSqlService">
    <property name="sqlmapFile" value="sqlmap.xml" />
  </bean>
  // highlight-end
</beans>
```

아래는 스프링 컨테이너의 초기 작업 순서입니다.

오브젝트 생성, 의존성 주입, 애노테이션 작업 순입니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-2.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-2.png)

### 7.2.4 변화를 위한 준비: 인터페이스 분리

현재 XmlSqlService 는 SQL 을 특정 포맷의 XML 에서 SQL 데이터를 가져오고, HashMap 타입 컬렉션에 저장하고 사용하고 있습니다.

만약 가져온 SQL 정보를 다른 방식으로 다루고 싶다면 지금까지 만든 코드를 변경해야 합니다.

SQL 을 가져오는 것과 보관해두고 사용하는 것은 각각 독자적으로 변경가능한 독립적인 부분입니다.

#### 책임에 따른 인터페이스 정의

먼저 분리가능한 관심사를 구분해봅니다.

첫째로 SQL 정보를 외부의 리소스로부터 읽어오는 것 입니다.

두번째는 읽어온 SQL 을 보관해두고 필요할 때 제공해주는 것 입니다.

부가적으로 생각해 볼 수 있는 것은 한 번 가져온 SQL 을 필요에 따라 수정할 수 있게 하는 것입니다.

SqlService 의 구현 클래스가 각각의 책임을 가진 SqlReader 와 SqlRegistry 두 가지 타입의 오브젝트를 사용하도록 만듭니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-3.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-3.png)

구조상 SqlReader 가 읽어온 SQL 정보가 다시 SqlRegistry 에 전달되서 등록되어야 합니다.

SqlReader 가 SQL 정보를 가져와서 SqlRegistry 에 전달하는 방법을 Map 을 이용하는 방식으로 생각할 수도 있습니다.

```java
public class XmlSqlService implements SqlService {

  // ...
  
  Map<String, String> sqls = sqlReader.readSql();
  sqlRegistry.addSqls(sqls);
  
  // ...
  
}
```

SqlReader 와 SqlRegistry 는 SQL 정보를 읽어오는 방법과 이를 저장해두는 방법의 구체적인 구현으로부터 독립적인 인터페이스로 만들어져야 합니다.

SqlReader 에서 내부의 구현방식으로 SQL 정보를 읽어와서는 Map 으로 바꿔주고, SqlRegistry 에서도 Map 으로 읽은 정보를 다시 내부의 구현방식으로 바꿔야하는 과정이 필요하게 됩니다.

둘 사이에서 정보를 전달하기 위해 일시적으로 Map 타입의 형식을 갖도록 만들어지는 것은 바람직하지 않습니다.

SqlReader 에게 SqlRegistry 전략을 제공해주면서 이를 이용해서 SQL 정보를 SqlRegistry 에 저장하라고 요청하는 방식을 생각해 볼 수 있습니다.

```java
public class XmlSqlService implements SqlService {

  // ...
  
  sqlReader.readSql(sqlRegistry);
  
}
```

자바의 오브젝트는 데이터를 가질 수 있습니다.

자신이 가진 데이터를 이용해 어떻게 작업해야 할지도 가장 잘 알고 있습니다.

그렇다면 오브젝트 스스로 자신의 데이터로 충실히 작업하게 만들면 됩니다.

SqlReader 는 내부의 정보를 형식을 바꿔서 반환하는 대신에 협력관계에 있는 의존 오브젝트인 SqlRegistry 가 SQL 정보의 등록을 수행할 때 정보를 제공하면 됩니다.

아래는 SqlReader 가 SqlRegistry 와 의존관계를 가지고 작업을 진행하도록 만들었을 때의 구조입니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-4.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-4.png)

SqlService 는 SqlReader 가 사용할 SqlRegistry 오브젝트를 제공해주는 역할을 합니다.

SqlReader 입장에서는 SqlRegistry 의 오브젝트를 런타임 시에 제공받으니 일종의 수동 DI 입니다.

SqlRegistry 는 SqlService 에 등록된 SQL 을 검색해서 돌려주는 기능을 하기에 SqlService 가 의존하는 오브젝트이기도 합니다.

#### SqlRegistry 인터페이스

```java
public interface SqlRegistry {
  
  // SQL 을 등록하는 기능
  void registerSql(String key, String sql); 

  // SQL 을 검색하는 기능
  String findSql(String key) throws SqlNotFoundException; 
  
}
```

SQL 을 검색하는 기능은 실패할 때는 예외를 던지게 하였습니다.

코드에 버그가 있거나 설정에 문제가 있어 발생한 예외라면 복구할 가능성이 적다고 판단되어 런타임 예외로 만들었습니다.

런타임 예외이지만 명시적으로 메소드가 던지는 예외를 선언해두는 편이 좋습니다.

#### SqlReader 인터페이스

```java
public interface SqlReader {

  // SQL 정보를 읽는 기능
  void read(SqlRegistry sqlRegistry); 
  
}
```

SQL 정보를 읽는 기능은 SQL 정보를 읽고, SqlRegistry 오브젝트를 파라미터로 DI 받아서 SQL 을 등록하는데 사용합니다.

### 7.2.5 자기참조 빈으로 시작하기

#### 다중 인터페이스 구현과 간접 참조

SqlService 구현 클래스는 SqlReader 와 SqlRegistry 두 개의 프로퍼티를 DI 받을 수 있는 구조이어야 합니다.

SqlService 를 포함한 3개의 인터페이스를 구현한 클래스간의 구조는 아래와 같습니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-5.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-5.png)

인터페이스에만 의존하도 만들어야 스프링의 DI 를 적용할 수 있습니다.

DI 를 적용하지 않더라도 자신이 사용하는 오브젝트의 클래스가 어떤 것인지 알지 못하게 만드는 것이 좋습니다.

그것이 구현 클래스를 바꾸고 의존 오브젝트를 변경해서 자유롭게 확장할 기회를 제공해주는 것입니다.

3 개의 인터페이스를 하나의 클래스가 모두 구현하는 경우를 생각해 봅니다.

클래스의 구현 내용을 상속하는 extends 와 다르게 인터페이스는 한 개 이상을 상속하는 것이 가능합니다.

인터페이스도 상속이라고 표현할 수 있습니다.

인터페이스 구현은 타입을 상속하는 것이기 때문입니다.

구현 클래스가 구현은 다르지만 같은 타입을 상속받았기 때문에 다형성을 활용할 수 있습니다.

따라서 하나의 클래스가 여러 개의 인터페이스를 상속한다면 여러 종류의 타입으로 존재할 수 있습니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-6.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-6.png)

XmlSqlService 가 세 개의 인터페이스를 구현하게 해도 상관없습니다.

이제부터는 책임이 분리되지 않았던 XmlSqlService 클래스가 책임을 정의한 인터페이스를 구현하도록 만들어 보겠습니다.

#### 인터페이스를 이용한 분리

먼저 XmlSqlService 는 SqlService 인터페이스만의 구현 클래스라고 생각하면 다른 두 개의 인터페이스 타입 오브젝트에 의존하는 구조여야 합니다.

```java
public class XmlSqlService implements SqlService {

  private SqlReader sqlReader;
  private SqlRegistry sqlRegistry;

  public void setSqlReader(SqlReader sqlReader) {
    this.sqlReader = sqlReader;
  }

  public void setSqlRegistry(SqlRegistry sqlRegistry) {
    this.sqlRegistry = sqlRegistry;
  }

  // ...

}
```

다음으로 XmlSqlService 가 SqlRegistry 를 구현하도록 합니다.

```java
public class XmlSqlService implements SqlService, SqlRegistry {

  // ...

  // highlight-next-line
  private Map<String, String> sqlMap = new HashMap<String, String>();

  public String findSql(String key) throws SqlNotFoundException {
    String sql = sqlMap.get(key);
    if (sql == null)
      throw new SqlRetrievalFailureException(key + "를 이용해서 SQL을 찾을 수 없습니다");
    else
      return sql;
  }

  public void registerSql(String key, String sql) {
    sqlMap.put(key, sql);
  }

  // ...

}
```

마지막 인터페이스인 SqlReader 를 구현합니다.

```java
public class XmlSqlService implements SqlService, SqlRegistry, SqlReader {

  // ...

  private String sqlmapFile;

  public void setSqlmapFile(String sqlmapFile) {
    this.sqlmapFile = sqlmapFile;
  }

  // highlight-next-line
  public void read(SqlRegistry sqlRegistry) {
    String contextPath = Sqlmap.class.getPackage().getName(); 
    try {
      JAXBContext context = JAXBContext.newInstance(contextPath);
      Unmarshaller unmarshaller = context.createUnmarshaller();
      InputStream is = UserDao.class.getResourceAsStream(sqlmapFile);
      Sqlmap sqlmap = (Sqlmap)unmarshaller.unmarshal(is);
      for(SqlType sql : sqlmap.getSql()) {
        // highlight-next-line
        sqlRegistry.registerSql(sql.getKey(), sql.getValue());
      }
    } catch (JAXBException e) {
      throw new RuntimeException(e);
    }     
  }

}
```

SqlReader 의 구현 코드에서 SqlRegistry 구현 코드의 내부정보에 직접 접근해서는 안됩니다.

파라미터로 전달받은 SqlRegistry 의 registerSql() 메소드를 사용해야만 합니다.

@PostConstruct 가 달린 초기화 메소드와 getFinder() 메소드가 sqlReader 와 sqlRegistry 오브젝트를 사용하도록 변경합니다.

```java
public class XmlSqlService implements SqlService, SqlRegistry, SqlReader {

  @PostConstruct
  public void loadSql() {
    // highlight-next-line
    this.sqlReader.read(this.sqlRegistry);
  }

  public String getSql(String key) throws SqlRetrievalFailureException {
    // highlight-start
    try {
      return this.sqlRegistry.findSql(key);
    } 
    catch(SqlNotFoundException e) {
      throw new SqlRetrievalFailureException(e);
    }
    // highlight-end
  }
  
}
```

loadSql() 메소드에서는 sqlReader 가 sqlRegistry 를 파라미터로 받아서 SQL 을 읽고 저장하도록 합니다.

getSql() 메소드에서는 SqlRegistry 타입의 오브젝트에게 요청해서 findSql() 메소드를 실행해 SQL 을 검색하게 하고, 발생하는 예외를 SqlService 인터페이스에서 정의한 예외로 전환해줍니다.

#### 자기참조 빈 설정

여기까지 XmlSqlService 클래스 안의 코드들을 세 가지 인터페이스를 구현하는 방법으로 분리했습니다.

같은 클래스 안의 내용이지만 다른 인터페이스를 통해 SQL 을 읽을 때는 SqlReader 인터페이스를 통하고, SQL 을 검색할 때는 SqlRegistry 인터페이스를 통해 접근하도록 하였습니다.

이제 빈 설정으로 DI 가 일어나도록 합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ... >
  // ...
  // highlight-start
  <bean id="sqlService" class="springbook.user.sqlservice.XmlSqlService">
    <property name="sqlReader" ref="sqlService" />
    <property name="sqlRegistry" ref="sqlService" />
    <property name="sqlmapFile" value="sqlmap.xml" />
  </bean>
  // highlight-end
</beans>
```

스프링은 프로퍼티의 ref 항목에 자기 자신을 넣는 것을 허용합니다.

이를 통해 sqlService 를 구현한 메소드와 초기화 메소드는 외부에서 DI 된 오브젝트라고 생각하게 됩니다.

사실 자기 자신을 참조하는 빈은 흔히 쓰이는 방법은 아닙니다.

책임이 다르다면 클래스를 구분하고 각기 다른 오브젝트로 만들어지는 것이 자연스럽습니다.

<div class="notice--info" markdown="1">
자기참조 빈을 만들어보는 것은 책임과 관심사가 복잡하게 얽혀 있어서 확장이 힘들고 변경에 취약한 클래스를 유연한 구조로 만들려고 할 때 처음 시도해볼 수 있는 방법입니다.<br>
이를 통해 기존의 복잡하게 얽혀 있던 코드를 책임을 가진 단위로 구분해낼 수 있습니다.<br>
</div>

### 7.2.6 디폴트의존관계

#### 확장 가능한 기반 클래스

이제 XmlSqlService 의 세가지 기능을 분리하고 DI 로 조합하여 사용하게 만들어 봅니다.

SqlService 인터페이스를 구현하는 기본 클래스를 BaseSqlService 로 합니다.

```java
public class BaseSqlService implements SqlService {

  // highlight-start
  private SqlReader sqlReader;
  private SqlRegistry sqlRegistry;
  // highlight-end
    
  public void setSqlReader(SqlReader sqlReader) {
    this.sqlReader = sqlReader;
  }

  public void setSqlRegistry(SqlRegistry sqlRegistry) {
    this.sqlRegistry = sqlRegistry;
  }

  @PostConstruct
  public void loadSql() {
    this.sqlReader.read(this.sqlRegistry);
  }

  public String getSql(String key) throws SqlRetrievalFailureException {
    try {
      return this.sqlRegistry.findSql(key);
    } 
    catch(SqlNotFoundException e) {
      throw new SqlRetrievalFailureException(e);
    }
  }
}
```

BaseSqlService 를 SqlService 빈으로 등록하고 SqlReader 와 SqlRegistry 를 구현한 클래스도 빈으로 등록하여 DI 해줍니다.

HashMap 을 이용한 SqlRegistry 코드도 독립 클래스 HashMapSqlRegistry.java 로 분리합니다.

```java
public class HashMapSqlRegistry implements SqlRegistry {
  // ...
}
```

JAXB 를 이용한 SqlReader 코드로 독립 클래스 JaxbXmlSqlReader.java 로 분리합니다.

```java
public class JaxbXmlSqlReader implements SqlReader {
  // ...
}
```

빈 설정도 변경해 줍니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ... >
  // ...
  // highlight-start
  <bean id="sqlService" class="springbook.user.sqlservice.BaseSqlService">
    <property name="sqlReader" ref="sqlReader" />
    <property name="sqlRegistry" ref="sqlRegistry" />
  </bean>
  
  <bean id="sqlReader" class="springbook.user.sqlservice.JaxbXmlSqlReader">
    <property name="sqlmapFile" value="sqlmap.xml" />
  </bean>
  
  <bean id="sqlRegistry" class="springbook.user.sqlservice.HashMapSqlRegistry">
  </bean>
  // highlight-end
</beans>
```

#### 디폴트 의존관계를 갖는 빈 만들기

확장을 고려해 기능을 분리하고, 인터페이스와 전략 패턴을 도입하면 늘어난 클래스와 인터페이스 구현, 그리고 의존관계 설정에 대한 부담을 감수해야 합니다.

특정 의존 오브젝트가 대부분의 환경에서 디폴트라면 디폴트 의존관계를 갖는 빈을 만들어 볼 수도 있습니다.

<div class="notice--info" markdown="1">
디폴트 의존관계란 외부에서 DI 받지 않는 경우 기본적으로 자동 적용되는 의존관계를 말합니다.<br>
</div>

DefaultSqlService 를 만들고 사용할 디폴트 의존 오브젝트를 스스로 DI 하는 방식을 생각해볼 수 있습니다.

```java
public class DefaultSqlService extends BaseSqlService {
  public DefaultSqlService() {
    setSqlReader(new JaxbXmlSqlReader());
    setSqlRegistry(new HashMapSqlRegistry());
  }
}
```

하지만 의존 오브젝트인 JaxbXmlSqlReader 에 필요한 프로퍼티인 sqlmapFile 을 주입할 수 없어서 테스트는 실패합니다.

DefaultSqlService 가 sqlmapFile 을 받아서 내부적으로 JaxbXmlSqlReader 만드는 방법도 있습니다.

하지만 사용여부가 불확실한 JaxbXmlSqlReader 때문에 DefaultSqlService 가 sqlmapFile 을 가지고 있는 것은 어색합니다.

<div class="notice--primary" markdown="1">
외부 클래스의 프로퍼티로 정의해서 전달받는 방법 자체는 나쁘지 않지만 DefaultSqlService 에 적용하기에는 적절치 않다.<br>
디폴트라는 건 다른 명시적인 설정이 없는 경우에 기본적으로 사용하겠다는 의미다.<br>
DefaultSqlService 는 JaxbXmlSqlReader 를 디폴트 오브젝트로 갖고 있을 뿐, 이를 사용하지 않을 수도 있다.<br>
따라서 반드시 필요하지않은 sqlmapFile 을 프로퍼티로 등록해두는 것은 바람직하지 못하다.<br>
<br>
7장_ 스프링 핵심 기술의 응용, 595.<br>
</div>

JaxbXmlSqlReader 가 sqlmapFile 의 기본값을 가진 오브젝트를 만들도록 합니다..

```java
public class JaxbXmlSqlReader implements SqlReader {

  // highlight-start
  private final String DEFAULT_SQLMAP_FILE = "sqlmap.xml";
  private String sqlmapFile = DEFAULT_SQLMAP_FILE;
  // highlight-end

  public void setSqlmapFile(String sqlmapFile) { 
    this.sqlmapFile = sqlmapFile;
  }

  // ...
  
}
```

이렇게 하면 sqlmapFile 프로퍼티를 지정하면 지정된 파일이 사용되고, 아니면 디폴트로 선언된 파일이 사용됩니다.

DI 를 사용한다고 해서 항상 모든 프로퍼티 값을 설정에 넣고 빈으로 지정할 필요는 없습니다.

만들어진 DefaultSqlService 는 SqlService 를 바로 구현한 것이 아니라 BaseSqlService 를 상속했습니다.

DefaultSqlService 는 BaseSqlService 의 sqlReader 와 sqlRegistry 프로퍼티를 그대로 가지고 있고, 또한 변경할 수 있습니다.

디폴트 의존 오브젝트를 사용하면 설정으로 다른 구현 오브젝트를 사용하더라도 일단 디폴트 의존 오브젝트를 만들어버린다는 단점이 있습니다.

이럴때는 @PostConstruct 초기화 메소드를 이용해서 프로퍼티가 설정되었는지 확인하고 없는 경우에만 디폴트 오브젝트를 만드른 방법을 사용하면 됩니다.
