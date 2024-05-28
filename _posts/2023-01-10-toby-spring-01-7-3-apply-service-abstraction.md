---
# layout: post
title: "7장 스프링 핵심 기술의 응용 - 7.3 서비스 추상화 적용"
# excerpt: "DAO 에 트랜잭션을 적용해보면서 스프링이 어떻게 성격이 비슷한 여러 기술을 추상화하고, 일관된 방법으로 사용할 수 있도록 지원하는지 알아봅니다."
categories:
  - "토비의 스프링 3.1"
tags:
  - "스프링 핵심 기술의 응용"
---

## 7.3 서비스 추상화 적용

JaxbXmlSqlReader 를 개선할 부분입니다.

- 필요에 따라 JAXB 외에 다른 기술로 바꿔서 사용할 수 있게 한다.
- XML 파일을 임의의 파일위치나 원격지 등 좀 더 다양한 소스에서 가져올 수 있게 한다.

### 7.3.1 OXM 서비스추상화

XML 과 자바오브젝트를 매핑해서 상호 변환해주는 기술을 OXM Object-XML Mapping 이라고 합니다.

기능이 같은 여러가지 기술이 존재한다면 서비스 추상화를 고민해볼 수 있습니다.

<div class="notice--primary" markdown="1">
로우레벨의 구체적인 기술과 API 에 종속되지 않고 추상화된 레이어와 API 를 제공해서 구현 기술에 대해 독립적인 코드를 작성할 수 있게 해주는 서비스 추상화가 필요하다.<br>
스프링이 제공하는 OXM 추상 계층의 API 를 이용해서 XML 문서와 오브젝트 사이의 변환을 처리하게 하면, 코드 수정 없이도 OXM 기술을 자유롭게 바꿔서 적용할 수 있다.<br>
<br>
7장_ 스프링 핵심 기술의 응용, 597.<br>
</div>

#### OXM 서비스 인터페이스

스프링의 OXM 추상화 인터페이스에는 자바 오브젝트를 MXL 로 변환하는 Marshaller 와 반대인 Unmarshaller 가 있습니다.

Unmarshaller 인터페이스의 unmarshal() 메소드는 XML 파일에 대한 정보를 담은 Source 타입의 오브젝트를 파라미터로 받으면 자바 오브젝트 트리로 변환하고 루트 오브젝트를 돌려줍니다.

#### JAXB 구현 테스트

Unmarshaller 인터페이스의 JAXB 를 사용하는 구현 클래스의 이름은 Jaxb2Marshaller 입니다.

OxmTest-context.xml 을 만들고 Jaxb2Marshaller 를 빈으로 등록해줍니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ... >

  // highlight-next-line
  <bean id="unmarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
    <property name="contextPath" value="springbook.user.sqlservice.jaxb" />
  </bean>

</beans>
```

unmarshaller 빈은 Unmarshaller 타입입니다.

@Autowired 를 이용해서 unmarshaller 빈을 가져와서 unmarshal() 메소드를 호출해주면 됩니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class OxmTest {

  // highlight-start
  @Autowired
  Unmarshaller unmarshaller;
  // highlight-end
  
  @Test 
  public void unmarshallSqlMap() throws XmlMappingException, IOException  {
    Source xmlSource = new StreamSource(getClass().getResourceAsStream("sqlmap.xml"));
    // highlight-next-line
    Sqlmap sqlmap = (Sqlmap)this.unmarshaller.unmarshal(xmlSource);
    
    List<SqlType> sqlList = sqlmap.getSql();    
    assertThat(sqlList.size(), is(3));
    assertThat(sqlList.get(0).getKey(), is("add"));
    assertThat(sqlList.get(0).getValue(), is("insert"));
    assertThat(sqlList.get(1).getKey(), is("get"));
    assertThat(sqlList.get(1).getValue(), is("select"));
    assertThat(sqlList.get(2).getKey(), is("delete"));
    assertThat(sqlList.get(2).getValue(), is("delete"));
  }
}
```

#### Castor 구현 테스트

Castor 이라는 OXM 기술도 있습니다.

Castor 는 XML 매핑파일을 이용해서 변환할 수 있습니다.

```xml
<?xml version="1.0"?>
<!DOCTYPE mapping PUBLIC "-//EXOLAB/Castor Mapping DTD Version 1.0//EN" "http://castor.org/mapping.dtd">
<mapping>
    <class name="springbook.user.sqlservice.jaxb.Sqlmap">
        <map-to xml="sqlmap" />
        <field name="sql"
               type="springbook.user.sqlservice.jaxb.SqlType"
               required="true" collection="arraylist">
            <bind-xml name="sql" node="element" />
        </field>
    </class>
    <class name="springbook.user.sqlservice.jaxb.SqlType">
        <map-to xml="sql" />
        <field name="key" type="string" required="true">
            <bind-xml name="key" node="attribute" />
        </field>
        <field name="value" type="string" required="true">
            <bind-xml node="text" />
        </field>
    </class>
</mapping>
```

이번에는 Castor 를 사용하도록 설정을 변경해 줍니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ... >

  // highlight-next-line
  <bean id="unmarshaller" class="org.springframework.oxm.castor.CastorMarshaller">
    <property name="mappingLocation" value="springbook/learningtest/spring/oxm/mapping.xml" />
  </bean>

</beans>
```

이렇게 서비스 추상화는 로우레벨의 기술을 필요에 따라 변경하면서도 일관된 애플리케이션 코드를 유지할 수 있도록 해줍니다.

### 7.3.2 OXM 서비스 추상화 적용

스프링의 OXM 추상화 기능을 이용하는 SqlService 를 만들어 보겠습니다.

OxmSqlService 라고 하고 SqlReader 는 스프링의 OXM Unmarshaller 를 이용하도록 하겠습니다.

#### 멤버 클래스를 참조하는 통합 클래스

OxmSqlService 는 BaseSqlService 와 유사하게 SqlReader 타입의 의존 오브젝트를 사용하지만 스태틱 멤버 클래스로 내장하여 사용하도록 만들어봅니다.

이렇게 의존 오브젝트를 자신만이 사용하도록 독점하는 구조로 만들면 별도의 빈 설정없이 깔끔하게 만들 수 있습니다.

**Note:**<br>
7장_ 스프링 핵심 기술의 응용, 603.<br>
내장된 SqlReader 구현을 외부에서 사용하지 못하도록 제한하고 스스로 최적화된 구조로 만들어 두는 것이다.<br>
밖에서 볼 때는 하나의 오브젝트로 보이지만 내부에서는 의존관계를 가진 두 개의 오브젝트가 깔끔하게 결합돼서 사용된다.<br>
유연성은 조금 손해보더라도 내부적으로 낮은 결합도를 유지한 채로 응집도가 높은 구현을 만들 때 유용하게 쓸 수 있는 방법이다.
{: .notice--primary}

SqlReader 구현을 내장하고 있는 OxmSqlService 의 구조입니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-7.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-7.png)

```java 
// title="OxmSqlService.java"
public class OxmSqlService implements SqlService {

  // highlight-next-line
  private final OxmSqlReader oxmSqlReader = new OxmSqlReader();

  // highlight-next-line
  private class OxmSqlReader implements SqlReader {
    // ...
  }
}
```

OxmSqlReader 는 private 멤버 클래스라서 외부에서 접근하거나 사용할 수 없습니다.

또한 final 로 선언하고 직접 오브젝트를 생성하기 때문에 OxmSqlReader 를 DI 하거나 변경할 수 없습니다.

두 개의 클래스를 강하게 결합시키고 확장이나 변경에 제한을 주는 이유는 OxmSqlService 를 OXM 을 이용하는 서비스 구조로 최적화하기 위해서입니다.

**Note:**<br>
7장_ 스프링 핵심 기술의 응용, 604.<br>
스프링의 OXM 서비스 추상화를 사용하면 언마샬러를 빈으로 등록해야 한다.<br>
편리한 확장과 유연한 변경을 위해서 클래스를 분리하고 빈을 따로 등록해 DI 할 수 있도록 기본 구조를 가져간 것은 좋지만, 자꾸 늘어나는 빈의 개수와 반복되는 비슷한 DI 구조가 불편하게 느껴질 수도 있다.
{: .notice--info}

설정을 단순하게 하는 다른 방법으로 BaseSqlService 를 확장해서 디폴트 설정을 두는 방법도 있습니다.

하지만 디폴트 의존 오브젝트를 만들어주는 방식의 한계는 디폴트로 내부에서 만드는 오브젝트의 프로퍼티를 외부에서 지정해주기 힘들다는 점 입니다.

그래서 디폴트 의존 오브젝트를 만들 때는 하나의 빈 설정만으로 SqlService 와 SqlReader 의 필요한 프로퍼티 설정이 모두 가능하도록 만들 필요가 있습니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-8.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-8.png)

OxmSqlReader 는 OxmSqlService 에 의해서만 만들어지기 때문에 OxmSqlReader 가 DI 로 제공받아야 하는 프로퍼티는 OxmSqlService 의 프로퍼티를 통해 간접적으로 DI 받아야 합니다.

아래는 OxmSqlService 의 프로퍼티를 통해 OxmSqlReader 의 프로퍼티를 설정해주는 코드입니다.

```java
public class OxmSqlService implements SqlService {

  private final OxmSqlReader oxmSqlReader = new OxmSqlReader();

  // ...
  
  // highlight-start
  public void setUnmarshaller(Unmarshaller unmarshaller) {
    this.oxmSqlReader.setUnmarshaller(unmarshaller);
  }
  // highlight-end
  
  // highlight-start
  public void setSqlmapFile(String sqlmapFile) {
    this.oxmSqlReader.setSqlmapFile(sqlmapFile);
  }
  // highlight-end
  
  // ...
  
  private class OxmSqlReader implements SqlReader {
  
    private Unmarshaller unmarshaller;
    private final static String DEFAULT_SQLMAP_FILE = "sqlmap.xml";
    private String sqlmapFile = DEFAULT_SQLMAP_FILE;

    // ...

    // highlight-start
    public void setUnmarshaller(Unmarshaller unmarshaller) {
      this.unmarshaller = unmarshaller;
    }
    // highlight-end
    
    // highlight-start
    public void setSqlmapFile(String sqlmapFile) {
      this.sqlmapFile = sqlmapFile;
    }
    // highlight-end
    
    // ...
    
  }
}
```

이 방법은 UserDaoJdbc 안에서 JdbcTemplate 을 직접 만들어 사용할 때와 유사합니다.

OxmSqlService 가 UserDaoJdbc 와 다른 점은 UserDaoJdbc 에서는 setDataSource() 메소드에서 JdbcTemplate 오브젝트의 생성과 DataSource 전달을 하였습니다.

OxmSqlService 의 경우에는 두 개의 프로퍼티가 필요하기 때문에 미리 오브젝트를 만들어주고 각 수정자 메소드에서 DI 받은 값을 oxmSqlReader 에 전달해주어야 합니다.

완성된 OxmSqlService 입니다.

```java
public class OxmSqlService implements SqlService {
  
  // highlight-start
  private final OxmSqlReader oxmSqlReader = new OxmSqlReader();
  private SqlRegistry sqlRegistry = new HashMapSqlRegistry();
  // highlight-end
  
  public void setSqlRegistry(SqlRegistry sqlRegistry) {
    this.sqlRegistry = sqlRegistry;
  }
  
  public void setUnmarshaller(Unmarshaller unmarshaller) {
    this.oxmSqlReader.setUnmarshaller(unmarshaller);
  }
  
  // highlight-start
  @PostConstruct
  public void loadSql() {
    this.oxmSqlReader.read(this.sqlRegistry);
  }
  // highlight-end
  
  // highlight-next-line
  private class OxmSqlReader implements SqlReader {
    
    private Unmarshaller unmarshaller;
    private final static String DEFAULT_SQLMAP_FILE = "sqlmap.xml";
    private String sqlmapFile = DEFAULT_SQLMAP_FILE;

    public void setUnmarshaller(Unmarshaller unmarshaller) {
      this.unmarshaller = unmarshaller;
    }

    public void setSqlmap(Resource sqlmap) {
      this.sqlmap = sqlmap;
    }

    public void read(SqlRegistry sqlRegistry) {
      try {
        Source source = new StreamSource(
            UserDao.class.getResourceAsStream(this.sqlmapFile));
            
        Sqlmap sqlmap = (Sqlmap)this.unmarshaller.unmarshal(source);
        
        for(SqlType sql : sqlmap.getSql()) {
          sqlRegistry.registerSql(sql.getKey(), sql.getValue());
        }
      } catch (IOException e) {
        throw new IllegalArgumentException(this.sqlmapFile + "을 가져올 수 없습니다", e);
      }
    }
  }
}
```

SqlService 인터페이스를 구현한 부분은 BaseSqlService 와 동일합니다.

SqlReader 를 DI 받을 수 없다는 것 외에는 SqlReader 인터페이스를 이용한다는 것은 동일하기 때문입니다.

SqlReader 구현 오브젝트를 내부에서 만드는 방법을 사용하여 OXM 을 적용했지만 빈 설정은 단순하게 유지할 수 있습니다.

OXM 언마샬러 빈은 별도로 필요합니다.

SqlService 와 OXM 언마샬러를 사용하는 SqlReader 그리고 SqlRegistry 는 하나의 빈만 등록하는 것으로 충분합니다.

SqlRegistry 역시 필요에 따라 다른 구현으로 교체할 수도 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ... >
            
  <!-- sql service -->
  <bean id="sqlService" class="springbook.user.sqlservice.OxmSqlService">
    <property name="unmarshaller" ref="unmarshaller" /> 
  </bean>
    
  <bean id="unmarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
    <property name="contextPath" value="springbook.user.sqlservice.jaxb" />
  </bean>
  
  // ...

</beans>
```

추상화된 OXM 적용이라는 큰 기술적 변화가 있었지만 설정은 여전히 이해하기 좋습니다.

#### 위임을 이용한 BaseSqlService의 재사용

OxmSqlService 의 SqlReader 를 스태틱 멤버 클래스로 고정했기 때문에 설정이 간결해지고, 의도되지 않은 방식으로 확장될 위험이 없어졌습니다.

하지만 OxmSqlService 와 BaseSqlService 의 loadSql(), getSql() 메소드가 중복된다는 점이 꺼림칙합니다.

이럴 때는 loadSql(), getSql() 의 구현 로직을 BaseSqlService 에 두고 OxmSqlService 는 설정이나 구성을 변경해주기 위한 어댑터 개념으로 BaseSqlService 앞에 두는 설계가 가능합니다.

OxmSqlService 의 외형적인 틀은 유지한 채로 SqlService 의 기능 구현은 BaseSqlService 로 위임하는 것입니다.

이런 구조는 보통 프록시를 만들 때 사용합니다.

위임을 위해서는 두 개의 빈을 등록하고 클라이언트 요청을 직접 받는 빈이 주요한 내용은 뒤의 빈에게 전달해주는 구조로 만들어야 합니다.

하지만 OxmSqlService 와 BaseSqlService 를 위임구조로 만들기 위해 두 개의 빈을 등록하는 것은 불편합니다.

그래서 이런 경우는 DI 를 포기하고 OxmSqlService 와 BaseSqlService 를 한 클래스로 묶는 방법을 고려해 봅니다.

[![7장 스프링 핵심 기술의 응용](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-9.png)](/assets/images/posts/2023-01-10-toby-spring-01-7-1-detach-sql-and-dao/07-9.png)

OxmSqlService 는 OXM 기술에 특화된 SqlReader 를 내장하고 있습니다.

실제 SqlReader 와 SqlService 를 이용해 SqlService 의 기능을 구현하는 일은 내부에서 BaseSqlService 를 만들어서 위임합니다.

```java
public class OxmSqlService implements SqlService {
  
  // highlight-next-line
  private final BaseSqlService baseSqlService = new BaseSqlService();
  
  // ...
  
  @PostConstruct
  public void loadSql() {
    // highlight-start
    this.baseSqlService.setSqlReader(this.oxmSqlReader);
    this.baseSqlService.setSqlRegistry(this.sqlRegistry);
    
    this.baseSqlService.loadSql();
    // highlight-end
  }

  public String getSql(String key) throws SqlRetrievalFailureException {
    // highlight-next-line
    return this.baseSqlService.getSql(key);
  }
  
}
```

위임구조를 이용하면 OxmSqlService 에 있던 중복 코드를 깔끔하게 제거할 수 있습니다.

SqlReader 와 SqlRegistry 를 활용해 SqlService 를 제공하는 코드는 BaseSqlService 에만 존재합니다.

### 7.3.3 리소스추상화

자바에는 다양한 위치에 존재하는 리소스에 대해 단일화된 접근 인터페이스를 제공해주는 클래스가 없습니다.

그나마 URL 을 이용해 웹상의 리소스에 접근 할 때 사용할 수 있는 java.net.URL 클래스가 있습니다.

현재의 코드는 리소스의 위치와 종류에 따라서 다른 클래스와 메소드를 사용해야 합니다.

이런 경우는 목적은 동일하지만 사용법이 각기 다른 여러 가지 기술이 존재하기 때문에 서비스 추상화를 고민해 볼 수 있습니다.

#### 리소스

스프링은 자바의 리소스 접근 API 를 추강화해서 Resource 라는 추상화 인터페이스를 정의했습니다.

스프링의 거의 모든 API 는 외부의 리소스 정보가 필요할 때 항상 이 Resource 추상화를 이용합니다.

Resource 는 스프링에서 빈이 아니라 값으로 취급되기 때문에 추상화 적용 방법이 고민됩니다.

만약 Resource 가 빈으로 등록된다면 리소스 타입에 따라 그에 맞는 Resource 인터페이스의 구현 클래스를 지정해주면 됩니다.

하지만 Resource 는 값으로 취급되기 때문에 가능한 방법은 `<property>` 의 value 애트리뷰트에 넣는 방법밖에 없습니다.

#### 리소스 로더

스프링에는 URL 클래스와 유사하게 접두어를 이용해 Resource 오브젝트를 선언하는 방법이 있습니다.

문자열안에 리소스의 종류와 위치를 표현하게 합니다.

그리고 문자열로 정의된 리소스를 실제 Resource 타입 오브젝트로 변환해주는 ResourceLoader 를 제공합니다.

```java
public interface ResourceLoader {

  Resource getResource(String location);
  
}
```

ResourceLoader 의 대표적인 예는 스프링의 애플리케이션 컨텍스트입니다.

애플리케이션 컨텍스트가 구현해야 하는 인터페이스인 ApplicationContext 는 ResourceLoader 인터페이스를 상속하고 있습니다.

애플리케이션 컨텍스트가 사용할 스프링 설정정보가 담긴 XML 파일도 리소스 로더를 이용해 Resource 형태로 읽어옵니다.

빈의 프로퍼티 값을 변환할 때도 리소스 로더가 자주 사용됩니다.

빈으로 등록 가능한 클래스에 파일을 지정해주는 프로퍼티가 존재한다면 거의 모두 Resource 타입입니다.

Resource 타입은 빈으로 등록하지 않고 `<property>` 태그의 value 를 사용해 문자열로 값을 넣습니다.

문자열인 리소스 정보를 Resource 오브젝트로 변환해서 프로퍼티에 주입할 때, 애플리케이션 컨텍스트 자신이 리소스 로더로서 변환과 로딩 기능을 담당합니다.

만약 myFile 이라는 이름의 프로퍼티가 Resource 타입이라고 하면 다음과 같은 문자열로 리소스를 표현할 수 있습니다.

```xml
<property name="myFile" value="classpath:com/epril/myproject/myfile.txt" />
<property name="myFile" value="file:/data/myfile.txt" />
<property name="myFile" value="http://www.myserver.com/test.dat" />
```

myFile 프로퍼티 입장에서는 추상화된 Resource 타입의 오브젝트를 전달받기 때문에 리소스의 실제 위치나 종류에 관계없이 동일한 방버으로 리소스의 내용을 읽어올 수 있습니다.

#### Resource를 이용해 XML 파일 가져오기

OxmSqlService 에 Resource 를 적용해서 SQL 매핑정보가 담긴 파일을 가져올 수 있게 합니다.

문자열로 되어 있던 sqlmapFile 프로퍼티를 Resource 타입으로 바꿉니다.

Resource 타입은 getInputStream() 메소드를 이용하면 실제 소스가 어떤 것이든 관계없이 스트림으로 가져올 수 있습니다.

Resource 를 사용할 때는 Resource 오브젝트가 실제 리소스가 아니라는 점을 알아야 합니다.

**Note:**<br>
7장_ 스프링 핵심 기술의 응용, 615.<br>
Resource 를 사용할 때는 Resource 오브젝트가 실제 리소스가 아니라는 점을 주의해야 한다.<br>
Resource 는 단지 리소스에 접근할 수 있는 추상화된 핸들러일 뿐이다.<br>
따라서 Resource 타입의 오브젝트가 만들어졌다고 해도 실제로 리소스가 존재하지 않을 수 있다.
{: .notice--primary}
