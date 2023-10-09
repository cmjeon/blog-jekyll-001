---
layout: single
title: "2장 테스트 - 2.5 학습 테스트로 배우는 스프링"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 2장 테스트

## 2.5 학습 테스트로 배우는 스프링

자신이 만들지 않은 프레임워크나 라이브러리를 사용하기 위해 테스트 코드를 작성하는 경우가 있다.

이런 테스트를 학습 테스트 learning test 라고 한다.

학습 테스트의 목적은 기능에 대한 검증도 있지만 해당 기능을 얼마나 제대로 이해하고 있는지를 검증하는 것이다.

<!--truncate-->

학습 테스트는 기능의 사용법을 빠르고 정확하게 익히는데도 도움이 된다.

### 2.5.1 학습 테스트의 장점

테스트를 작성하면서 학습하면 얻게되는 장점들이다.

- 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다.
    - 자동화된 테스트 코드를 사용하면 다양한 조건에 따라 어떻게 다르게 작동하는지를 빠르게 확인할 수 있다.
- 학습 테스트 코드를 개발 중에 참고할 수 있다.
    - 학습 테스트는 다양한 기능과 조건에 대한 테스트 코드를 개별적으로 만들고 남겨둘 수 있다.
    - 이는 나중에 샘플 코드로 참고할 수 있다.
- 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다.
    - 업데이트 된 제품을 테스트를 통해 미리 검증해볼 수 있다.
- 테스트 작성에 대한 좋은 훈련이 된다.
    - 기능에 대한 학습 테스트는 실제로 프레임워크를 사용하는 애플리케이션 코드의 테스트와 비슷하다.
    - 또한, 애플리케이션 코드에 비해 단순하기 때문에 훈련의 기회로 삼으면 좋다.
- 새로운 기술을 공부하는 과정이 즐거워진다.
    - 테스트 코드를 만들면서 실제로 동작하는 모습을 확인하면 흥미가 생긴다.

스프링 레퍼런스 매뉴얼이나 서적만을 가지고 공부할 때는 잘 이해가 안되는 내용도 학습 테스트를 통해 이해할 수 있다.

스프링 학습 테스트를 만들 때 참고할 수 있는 가장 좋은 소스는 스프링 자신에 대한 테스트 코드이다.

스프링 배포판의 압축을 풀어보면 프레임워크 소스코드와 테스트 코드를 발견할 수 있다.

### 2.5.2 학습 테스트 예제

Junit 에 대한 학습 테스트를 만들어 본다.

테스트 메소드마다 새로운 오브젝트를 만드는 것이 사실인지 확인해보자.

새로운 테스트 클래스를 만들고 세 개의 테스트 메소드를 만든다.

테스트 클래스 자신의 타입으로 스태틱 변수를 하나 선언한다.

테스트 메소드에는 현재 스태틱 변수에 담긴 오브젝트와 자신을 비교한다.

```java
public class JUnitTest {
  static JUnitTest testObject;
  
  @Test public void test1() {
    assertThat(this, is(not(sameInstance(testObject))));
    testObject = this;
  }
    
  @Test public void test2() {
    assertThat(this, is(not(sameInstance(testobject)))); 
    testObject = this;
  }
  
  @Test public void test3() {
    assertThat(this, is(not(sameInstance(testObject))));
    testObject = this;
  }
  
}
```

세 개의 테스트 오브젝트 중 어떤 것도 중복이 되지 않는다는 것을 확인하도록 검증 방법을 바꾼다.

```java
public class JUnitTest {
  static Set<JUnitTest> testobjects = new HashSet<JUnitTest>();
  
  @Test public void test1() {
    assertThat(testObjects, not(hasItem(this)));
    testObjects.add(this);
  }
  
  @Test public void test2() { 
    assertrtThat(testObjects, not(hasItem(this)));
    testObjects.add(this);
  }
  
  @Test public void test3() { 
    assertThat(testObjects, not(hasItem(this))); 
    testObjects.add(this);
  }
  
}
```

스프링 테스트 컨텍스트 프레임워크에 대한 학습 테스트도 만들어보자.

JUnit 과 반대로 테스트용 애플리케이션 컨텍스트는 한 개만 만들어진다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@Contextconfiguration
public class JUnitTest {

  ©Autowired
  ApplicationContext context;

  static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();
  static ApplicationContext contextObject = null;
  
  @Test 
  public void test1() {
    assertThat(testObjects, not(hasItem(this))); 
    testObjects.add(this);
    
    assertThat(contextObject == null !! contextObject == this.context, is(true));
    contextObject = this.context;
  }
  
  ©Test
  public void test2() { 
    assertThat(testObjects, not(hasItem(this))); 
    testObjects.add(this);
    
    assertTrue(contextObject == null !! contextObject == this.context);
    contextObject = this.context;
  }
  
  ©Test
  public void test3() { 
    assertThat(testObjects, not(hasItem(this)));
    testObjects.add(this);
    
    assertThat(contextObject, either(is(nullValue())), or(is(this.context)));
    contextObject = this.context;
  }
  
}
```

### 2.5.3 버그 테스트

버스 테스트란 코드에 오류가 있을 때 그 오류를 가장 잘 드러내줄 수 있는 테스트를 말한다.

버그 테스트의 필요성과 장점을 생각해보자

- 테스트의 완성도를 높여준다.
    - 기존 테스트에서 미처 검증하지 못했던 부분이기 때문에 테스트를 보완할 수 있다.
- 버그의 내용을 명확하게 분석하게 해준다.
    - 버그를 테스트로 만들어서 실패하게 만드는 과정에서 버그를 좀 더 효과적으로 분석할 수 있다.
- 기술적인 문제를 해결하는데 도움이 된다.
    - 동일한 문제가 발생하는 가장 단순한 코드와 그에 대한 버그 테스트를 만들어서 도움을 구할 수 있다.

테스트 방법

- 동등분할(Equivalence partitioning)
    - 같은 결과를 내는 값의 범위를 구분해서 각 대표 값으로 테스트 하는 방법
- 경계값분석(boundary value analysis)
    - 동등분할 범위의 경계에서 주로 많이 발생한다는 특징을 이용해서 경계의 근처에 있는 값을 이용해 테스트하는 방법

## 2.6 정리

- 테스트는 자동화돼야 하고, 빠르게 실행할 수 있어야 한다.
- main() 테스트 대신 JUnit 프레임워크를 이용한 테스트 작성이 편리하다.
- 테스트 결과는 일관성이 있어야 한다. 코드의 변경 없이 환경이나 테스트 실행 순서에 따라 서 결과가 달라지면 안 된다.
- 테스트는 포괄적으로 작성해야 한다. 충분한 검증을 하지 않는 테스트는 없는 것보다 나쁠 수 있다.
- 코드 작성과 테스트 수행의 간격이 짧을수록 효과적이다.
- 테스트하기 쉬운 코드가 좋은 코드다.
- 테스트를 먼저 만들고 테스트를 성공시키는 코드를 만들어가는 테스트 주도 개발 방법도 유
  용하다.
- 테스트 코드도 애플리케이션 코드와 마찬가지로 적절한 리팩토링이 필요하다.
- @Before, @After를 사용해서 테스트 메소드들의 공통 준비 작업과 정리 작업을 처리할 수 있다.
- 스프링 테스트 컨텍스트 프레임워크를 이용하면 테스트 성능을 향상시킬 수 있다.
- 동일한 설정파일을 사용하는 테스트는 하나의 애플리케이션 컨텍스트를 공유한다.
- @Autowired를 사용하면 컨텍스트의 빈을 테스트 오브젝트에 DI 할 수 있다.
- 기술의 사용 방법을 익히고 이해를 돕기 위해 학습 테스트를 작성하자.
- 오류가 발견될 경우 그에 대한 버그 테스트를 만들어두면 유용하다.
