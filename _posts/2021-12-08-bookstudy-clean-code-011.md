---
layout: single
title: 클린코드 스터디 - 011
categories: 
  - bookstudy - cleancode
tags: 
  - cleancode
---

## 11장 시스템

### 도시를 세운다면?

도시의 일상은 큰 그림을 그리는 사람도 있고, 작은 사항에 집중하는 사람이 있기 때문에 유지됨

도시가 잘 돌아가는 다른 이유는 적절한 추상화와 모듈화

이로 인해 큰 그림을 이해하지 못하더라도 개인과 개인이 관리하는 구성요소가 잘 돌아감

깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 뽑기 쉬워짐

이 장에서는 시스템 수준에서도 깨끗한 코드를 유지하는 방법을 살펴보겠음

### 시스템 제작과 시스템 사용을 분리하라

제작과 사용은 분명 아주 다름

도시에 호텔을 건설(제작) 할 때와 호텔이 완공된 후(사용) 는 관심사가 아주 다름

소프트웨어도 시작 단계와 런타임 단계의 관심사는 분리되어야 함

```java
public Service getService() {
	if(service == null) {
		service = new MyServiceImpl(...); // 모든 상황에 적합한 기본값일까?
	}
	return service;
}
```

위의 코드는 초기화 지연(Lazy Initialization) 이나 계산 지연(Lazy Evaluation) 기법임

여기의 getService 메서드는 MyServiceImpl 와 파라미터에 의존하고 있음

그리고 getService 메서드를 테스트하기 위해서는 테스트 전용 객체(Test double, Mock Object?)가 필요해짐

런타임 로직에 객체 생성 로직을 섞어놓은 이유로 인해 else 에 대한 테스트도 필요해짐

이런 초기화 지연 기법을 여러 곳에서 반복적으로 사용하면 모듈성(응집도?) 이 저조해지고 중복이 심각해짐

체계적이고 탄탄한 시스템을 만들고 싶다면 소프트웨어의 모듈성을 높이기 위해 설정 로직과 실행 로직을 분리하여야 함

#### Main 분리

시스템의 생성과 사용을 분리하는 방법 중 하나로 생성과 관련한 코드를 모두 main 이나 main 이 호출하는 모듈(구축자)로 분리하고, 나머지 시스템은 모든 객체가 생성되고 사용 준비가 되었다고 가정하는 것임

main 메서드에서 구축자를 통해 객체를 생성하고, 어플리케이션을 실행함

어플리케이션은 객체가 생성되는 과정은 전혀 모르고, 오직 사용만 함

이 방법의 특징은 어플리케이션에서 사용할 모든 객체는 main 이나 구축자에 의해 생성되여야 함

#### 팩토리

객체가 생성되는 시점을 애플리케이션에서 결정하고 싶을 수 있음

예컨데 주문처리 시스템에서 OrderProcessing(어플리케이션) 이 LineItemFactory(구축자) 인터페이스에 LineItem 생성을 요청하면 LineItemFactory 에서 LineItem 인스턴스를 생성해 Order 에 추가하는 경우임

이 때 Abstract Factory 패턴을 사용함

#### 의존성 주입

제작과 사용을 분리하는 강력한 메커니즘 하나가 의존성 주입(Dependency Injection) 임

의존성 주입은 제어 역전(Inversion of Control) 을 의존성 관리에 적용한 메커니즘

제어 역전에서는 한 객체가 다른 객체에 책임을 떠넘기고 다른 객체는 넘겨받은 책임만 맡게 됨

Single Responsibility Principle 을 만족함

의존성 관리 맥락에서 객체는 의존성관리 책임을 특수한 컨테이너에 전가함

그 특수한 컨테이너는 필요한 객체의 인스턴스를 생성하고 생성자나 setter 메서드로 의존성을 설정함

### 확장

성장할지도 모른다는 기대로 조그만 마을에 6차선을 뚫는 비용을 정당화할 수 있을까?

'처음부터 올바르게' 시스템을 만들 수 있다는 믿음은 미신임

우리는 오늘 주어진 스토리에 맞춰 시스템을 구현하고, 내일은 내일의 스토리에 맞춰 확장하면 됨

이것이 반복적으로 점진적이라고 하는 애자일 방식의 핵심임

Test-driven Development, 리팩터링으로 얻어지는 Clean Code 가 이런 방식의 시스템 확장을 지원함

소프트웨어는 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있음

아래는 관심사를 적절하게 분리하지 못한 EJB 아키텍처임

```java
public abstract class Bank implements javax.ejb.EntityBean {
	// 비즈니스 로직
	...
	// 컨테이너 로직
	...
}
```

EJB 코드는 비즈니스 로직이 컨테이너(EntityBean) 와 강하게 결합되어 있음

Bank 클래스는 비즈니스 로직과 EJB 컨테이너의 로직을 억지로 구현해야 함

#### 횡단(cross-cutting) 관심사

EJB 아키텍처는 일부 영역에서 관심사를 거의 완벽하게 분리하였음

EJB 는 영속성(Persistence), 보안, 트랜젝션을 배치 기술자에서 정의함

이러한 EJB 아키텍처의 처리방식은 Aspect-Oriented Programming 에 대한 힌트를 제공하였음

AOP 에서 '관점'이라는 모듈 구성 개념은 '특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야 한다' 고 명시함

영속성을 예로 들면, 프로그래머는 영속적으로 저장할 객체와 속성을 선언한 후 영속성 책임을 AOP 프레임워크에 위임함

그러면 AOP 프레임워크는 대상 객체의 코드에 영향을 미치지 않는 상태로 동작 방식을 변경함

아래의 자바 프록시, 순수 자바 AOP 프레임워크, AspectJ 은 자바에서 사용하는 관점 혹은 관점과 유사한 메커니즘임

### 자바 프록시

자바 프록시는 단순한 상황에 적합(하다고) 함

프록시를 활용하여 영속성을 지원하는 예제임

```java
// Bank.java
import java.util.*

public interface Bank {
	Collection<Account> getAccounts();
	void setAccounts(Collection<Account> accounts);
}
```

```java
// BankImpl.java
import java.util.*

public class BankImpl implements Bank {
	private List<Account> accounts;
	public Collection<Account> getAccount() {
		return accounts;
	}
	public void setAccounts(Collection<Account> accounts) {
		this.accounts = new ArrayList<Account>();
		for(Account account: accounts) {
			this.accounts.add(account);
		}
	}
}
```

```java
// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*

public class BankProxyHandler implements InvocationHandler {
	private Bank bank;
	public BankProxyHandler(Bank bank) {
		this.bank = bank;
	}
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		String methodName = method.getName();
		if(methodName.equals("getAccounts")) {
			bank.setAccounts(getAccountsFromDatabase());
			return bank.getAccounts();
		} else if(methodName.equals("setAccounts")) {
			bank.setAccounts((Collection<Account>) args[0]);
			setAccountsToDatabase(bank.getAccounts());
			return null;
		} else {
			...
	}
	protected Collection<Account> getAccountsFromDatabase() { ... }
	protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}	
```

```java
// some other...
Bank bank = (Bank) Proxy.newProxyInstance(
	Bank.class.getClassLoader(),
	new Class[] { Bank.class },
	new BankProxyHandler(new BankImpl())
);

```

위 코드는  `bank.getAccounts()`  을 호출하면 내부적으로 invoke 메서드가 호출되어 getAccountFromDatabse 을 호출하여 bank 의 setAccounts 해주는(것으로 보이는) 예제임

확실히 영속성이 비즈니스 로직과 분리되었지만 실제로 짜여진 코드는 많이 복잡하고 깨끗한 코드를 작성하기 어려움

### 순수 자바 AOP 프레임워크

순수 자바 관점을 구현하는 자바 프레임워크는 내부적으로 프록시를 사용함

스프링은 도메인에 초점을 맞춘 POJO(Plain Old Java Object) 로 구현함

```xml
<beans>
  <bean 
    id="appDataSource" 
    ... 
  />
  <bean 
    id="bankDataAccessObject"
    p:dataSource-ref="appDataSource"
  />
  <bean 
    id="bank"
    p:dataAccessObject-ref="bankDataAccessObject"
  />
</beans>
```

`bank.getAccounts()`  을 호출하면 실제로는 Bank 클래스를 Decorator 패턴으로 감싸고 있는 appDataSource 와 통신하는 셈임

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

이로 인해 어플리케이션은 스프링(프레임워크) 관련 코드가 거의 없음

이는 사실상 스프링 독립적이고, EJB2 시스템이 지녔던 강한 결합(tight-coupling) 문제가 해소됨

EJB3 에서는 xml 설정 파일과 어노테이션을 사용해 선언적으로 횡단관심사를 지원함

아래는 EJB3 으로 Bank 클래스를 다시 작성하였음

```java
package com.example.banking.model;
import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
	@Id @GeneratedValue(Strategy=GenerationType.AUTO)
	...
	@Embeddable
	...
	@Embedded
	...
	@OneToMany(cascade=CascadeType.ALL, fetch=FetchType.EAGER, mappedBy="Bank")
	...
}
```

### AspectJ 관점

관심사를 관점으로 분리하는 가장 강력한 도구는 AspectJ 언어임

AspectJ 는 언어 차원에서 관점을 모듈화 구성으로 지원하는 자바 언어 확장임

AspectJ 는 관심사 분리에 탁월한데, 새 언어 문법과 사용법을 익혀야 하는 단점이 있음

그래서 최근에 나온 AspectJ 어노테이션 폼은 순수 자바 코드와 어노테이션을 사용하여 관점을 정의함

이는 새로운 언어라는 부담을 완화하였음

### 테스트 주도 시스템 아키텍처 구축

관점에 의한 관심사를 분리하는 방법은 그 위력이 막강함

코드 수준에서 아키텍처 관심사를 분리할 수 있다면 진정한 테스트 주도 아키텍처 구축이 가능함

따라서 그때그때 새로운 기술을 채택해 아키텍처를 발전시킬 수 있음

초창기 EJB 아키텍처는 기술을 너무 많이 넣는라 관심사를 제대로 분리하지 못하였음

소프트웨어 설계에는 과유불급이 딱 맞는 말임

### 의사 결정을 최적화하라

모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해짐

'가능한 마지막 순간까지 결정을 미루어라'는 말은 게으르거나 무책임한 것이 아님

최대한 정보를 모아서 최선의 결정을 하기 위함임

성급한 결정은 불충분한 지식으로 내린 결정임

### 명백한 가치가 있을 때 표준을 현명하게 사용하라

건축물이 만들어지는 속도가 놀라운 이유는 건축에는 여러 세기 동안 발전된 최적화된 부품과 방법과 표준이 있기 때문임

EJB2 는 표준이라는 이유만으로 여러 프로젝트(심지어 간단한)에서 채택되었음

저자는 이런 표준에 대한 집착때문에 정작 고객 가치가 뒷전으로 밀리는 경우를 많이 보았음

### 시스템은 도메인 특화 언어가 필요하다

소프트웨어 분야에서도 최근 들어 Domain-Specific Language 가 조명받고 있음

DSL 은 간단한 스크립트 언어나 표준 언어로 구현한 API 임

좋은 DSL 은 개념과 코드 사이의 간극을 좁혀줌

DSL 을 적절히 사용한다면 높은 추상화 수준으로 코드를 표현할 수 있음

### 결론

깨끗한 시스템, 아키텍처는 도메인과 기민함을 유지하면서 발전할 수 있음

모든 추상화 단계에서 의도는 명확하게 표현되어야 함

그러기 위해서는 POJO 를 작성하고 각 구현 관심사를 분리해야 함

시스템을 설계할 때는 실제로 돌아가는 가장 단순한 수단을 사용해야 함

## 참고
- 