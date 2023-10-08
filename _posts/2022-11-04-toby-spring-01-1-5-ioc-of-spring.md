---
layout: single
title: "1장 오브젝트와 의존관계 - 1.5 스프링의 IoC"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 1장 오브젝트와 의존관계

## 1.5 스프링의 IoC

스프링의 핵심을 담당하는 건 빈 팩토리 또는 애플리케이션 컨텍스트라고 불리는 것이다.

이 두가지는 앞에서 만든 DaoFactory 를 일반화 한 것이다.

### 1.5.1 오브젝트 팩토리를 이용한 스프링 IoC

스프링에서는 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트를 빈 Bean 이라고 부른다.

동시에 스프링 컨테이너가 생성, 관계 설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트를 가리킨다.

스프링에서 제어의 역전을 담당하는 IoC 오브젝트를 빈 팩토리 bean factory 라고 부른다.

보통은 빈 팩토리를 확장한 IoC 방식으로 만들어진 애플리케이션 컨텍스트 application context 라는 것을 사용한다.

빈 팩토리라고 말할 때는 빈의 생성, 관계에 집중한 것이고, 애플리케이션 컨텍스트라고 말할 때는 구성요소의 제어 작업을 담당하는 IoC 엔진이라는 의마가 좀 더 부각된 것이다.

스프링이 DaoFactory 를 설정을 담당하는 클래스라고 인식할 수 있도록 @Configuration 애노테이션을 추가한다.

그리고 오브젝트를 만들어주는 메소드에 @Bean 애노테이션을 붙여준다.

```java
@Configuration
public class DaoFactory {

  @Bean
  public UserDao userDao() {
    // ...
  }

  @Bean
  public ConnectionMaker connectionMaker() {
    // ...
  }
}
```

UserDaoTest 에서 DaoFactory 를 이용하여 애플리케이션 컨텍스트를 생성한다.

```java
public class UserDaoTest {

	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		UserDao dao = context.getBean("userDao", UserDao.class);

		// ...
	}

}
```

getBean() 메소드의 첫번째 파라미터는 오브젝트를 리턴하는 메소드명이다.

예제처럼 두번째 파라미터에 리턴타입을 제공하면 제네릭 메소드 방식을 이용해서 캐스팅 코드가 불필요하다.

### 1.5.2 애플리케이션 컨텍스트의 동작방식

스프링에서는 애플리케이션 컨텍스트를 IoC 컨테이너, 스프링 컨테이너, 빈 팩토리 등으로 부른다.

애플리케이션 컨텍스트는 애플리케이션 컨텍스트 인터페이스를 구현하였는데, 애플리케이션 컨텍스트 인터페이스는 빈 팩토리 인터페이스를 상속하였다.

따라서 애플리케이션 컨텍스트는 빈 팩토리의 일종이다.

@Configuration 이 붙은 DaoFactory 는 애플리케이션 컨텍스트가 활용하는 IoC 설정정보이다.

아래는 애플리케이션 컨텍스트가 사용되는 방식이다.

[![application-context-work](/assets/images/posts/2022-11-04-toby-spring-01-1-5-ioc-of-spring/application-context-work.png)](/assets/images/posts/2022-11-04-toby-spring-01-1-5-ioc-of-spring/application-context-work.png)

DaoFactory 를 오브젝트 팩토리로 직접 사용하는 것보다 애플리케이션 컨텍스트를 사용하면 몇 가지 장점이 있다.

1. 클라이언트가 구체적인 팩토리 클래스를 알 필요가 없다.
1. 종합 IoC 서비스를 제공해준다.
    - 오브젝트에 대한 후처리, 정보의 조합, 설정방식의 다변화 등 다양한 기능을 제공한다.
1. 빈을 검색하는 다양한 방법을 제공한다.

### 1.5.3 스프링 IoC 용어 정리

- 빈 bean : 스프링이 IoC 방식으로 관리하는 오브젝트
- 빈 팩토리 bean factory : IoC 를 담당하는 핵심 컨테이너
- 애플리케이션 컨텍스트 application context
    - 빈 팩토리를 확장한 IoC 컨테이너
    - 빈 팩토리 + 스프링에서 제공하는 애플리케이션 지원 기능
- 설정정보/설정 메타정보 configuration metadata
    - 빈 팩토리가 IoC 를 적용하기 위해 사용하는 메타정보
    - 애플리케이션의 형상정보, 청사진
- 컨테이너 container
    - IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트, 빈 팩토리의 또 다른 이름
    - 하나의 어플리케이션에 애플리케이션 컨텍스트 오브젝트가 여러 개 만들어질 수 있고, 이를 통틀어서 스프링 컨테이너라고 함
