---
layout: single
title: "2장 테스트 - 2.4 스프링 테스트 적용"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 2장 테스트

## 2.4 스프링 테스트 적용

현재까지 만들어진 코드로는 어플리케이션 컨텍스트가 각 테스트가 실행될 때마다 생성된다.

총 3번 만들어진다.

테스트를 가능한 독립적으로 하기 위해 매번 새로운 오브젝트를 사용하는게 원칙이다.

하지만 어플리케이션 컨텍스트처럼 자원이 많이 소모되는 경우 공유하는 오브젝트를 만들기도 한다.

공유하는 오브젝트를 쓸 때도 테스트는 일관성있는 실행 결과를 보장하고 테스트의 실행 순서가 결과에 영향을 미치지 않아야 한다.

어플리케이션 컨텍스트를 @BeforeClass 애노테이션을 붙인 스태틱 메소드에서 생성하도록 해보자.

### 2.4.1 테스트를 위한 애플리케이션 컨텍스트 관리

먼저 @Before 메소드에서 애플리케이션 컨텍스트를 생성하는 코드를 제거한다.

그리고 ApplicationContext 에 @Autowired 를 붙여준다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {
    
    // highlight-start
    @Autowired
    private ApplicationContext context;
    // highlight-end
    
    // ...
    
    @Before
    public void setUp() {
        this.dao = this.context.getBean("userDao", UserDao.class);
        // ...
    }
    
    // ...
    
}
```

@RunWith(SpringJUnit4ClassRunner.class) 을 지정해주면 JUnit 이 테스트가 사용할 애플리케이션 컨텍스트를 만들고 관리하는 작업을 진행해준다.

@ContextConfiguration은 자동으로 만들어줄 애플리케이션 컨텍스트의 설정파일 위치를 지정하는 애노테이션이다.

이렇게 해서 하나의 테스트 클래스 내의 테스트 메소드는 같은 애플리케이션 컨텍스 트를 공유해서 사용할 수 있다.

@Autowired 는 스프링의 DI 에 사용되는 특별한 애노테이션이다.

@Autowired 가 붙은 인스턴스 변수가 있으면 스프링이 변수 타입이 일치하는 빈을 찾는다.

그런데 applicationContext.xml 을 보면 ApplicationContext 라는 변수 타입은 없다.

그래도 스프링 애플리케이션 컨텍스트는 초기화할 때 자기 자신도 빈으로 등록하기 때문에 ApplicationContext 의 DI가 가능해진다.

@Autowired는 같은 타입의 빈이 두 개 이상 있는 경우에는 타입만으로는 어떤 빈을 가져올지 결정할 수 없다.

스프링은 빈 하나를 선택할 수 없는 경우에는 변수의 이름과 같은 이름의 빈을 가져온다.

만약 변수 이름으로도 빈을 찾을 수 없다면 에러를 발행시킨다.

가능한 인터페이스를 사용해서 애플리케이션 코드와 느슨하게 연결해두는 편이 좋다.

### 2.4.2 DI 와 테스트

UserDao 와 DB 커넥션 생성 클래스 사이에 DataSource 라는 인터페이스를 뒀다.

그렇다면 왜 구현클래스를 직접 쓰지 않고 인터페이스를 써야 할까?

심지어 구현클래스가 절대로 바뀌않는다고 확신이 들어도 인터페이스를 써야하는 이유가 있을까?

그 이유를 알아본다.

첫째, 소프트웨어 개발에서 절대로 바뀌지 않는 것은 없다.

클래스 대신 인터페이스를 이용하고, new 생성하기보다 DI 로 주입받게 하는건 아주 단순한 작업이다.

이 작은 개선이 변경이 필요한 상황이 닥쳤을 때 시간과 노력을 크게 줄여준다.

둘째, 인터페이스를 두고 DI 를 적용하면 다른 차원의 기능을 도입할 수 있다.

앞서 만들어보았던 DB 커넥션 개수를 카운팅하는 ConnectionMaker 를 생각해보라.

세번째, 테스트때문이다.

효울적인 테스트를 하기 위해서는 DI 를 적용해야 한다.

DI 는 테스트가 작은 단위의 대상에 대해 독립적으로 만들어지고 실행되게 하는데 중요한 역할을 한다.

DI 를 사용하면 UserDao 가 사용할 DataSource 오브젝트를 테스트 코드내에서 변경할 수 있다.

테스트 DB 에 연결해주는 DataSource 를 테스트 내에서 직접 만들 수도 있다.

테스트만을 위해 별도의 DI 설정을 할 수 있다.

test-applicationContext.xml 을 만들고 테스트코드에 해당 파일을 애플리케이션 컨텍스트로 사용하게 하면 된다.

```java
// highlist-next-line
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserDaoTest {
    
    // ...
    
}
```

테스트 환경에 적합한 구성을 가진 설정파일을 이용해서 테스트를 진행하면 된다.

스프링 컨테이너를 사용하지 않고 테스트를 만들 수도 있다.

@Before 메소드를 통해 직접 UserDao 의 오브젝트를 생성하고 테스트용 DataSource 오브젝트를 만들어 DI 해줄 수 있다.

이렇게 하면 매 테스트 매소드가 돌 때마다 새로운 UserDao 오브젝트가 만들어지지만 가벼운 오브젝트라면 괜찮다.

DI 가 적용된 코드라면 깔끔하게 테스트 코드를 만들 수 있다.

- 침투적 invasive 기술 : 특정 인터페이스나 클래스를 사용하도록 강제하는 기술. 애플리케이션 코드가 해당 기술에 종속된다.
- 비침투적 noninvative 기술 : 애플리케이션 로직을 담은 코드에 영향을 주지 않고 적용이 가능한 기술. 스프링

테스트하기 좋은 코드가 좋은 코드일 가능성이 높다.

DI 를 테스트에 이용하는 방법 중에 스프링 컨테이너 없이 테스트 할 수 있는 방법을 가장 우선적으로 고려하자.

수행속도도 빠르고 또한 코드가 간결하다.

만약 복잡한 의존관계를 갖고 있는 오브젝트를 테스트해야 한다면 스프링의 설정을 이용한 DI 방식의 테스트를 이용한다.

테스트 전용 설정파일을 만들면 된다.

예외적으로 의존관계를 강제로 구성해야할 수도 있다.

이 경우에는 컨텍스트에서 DI 받은 오브젝트에 다시 테스트 코드로 수동 DI 해서 테스트하자.

@DirtiesContext 애노테이션을 붙이면 된다.
