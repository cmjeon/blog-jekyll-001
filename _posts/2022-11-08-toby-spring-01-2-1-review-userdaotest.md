---
layout: single
title: "2장 테스트 - 2.1 UserDaoTest 다시 보기"
categories:
  - "토비의 스프링 3.1"
tags:
  - "spring"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 2장 테스트

스프링이 개발자에게 제공하는 가장 중요한 가치는 객체지향과 테스트라고 말할 수 있다.

애플리케이션은 계속 변하고 복잡해져 간다.

그 변화에 대응할  수 있는 방법이 확장과 변화를 고려한 객체지향적 설계와 만들어진 코드에 확신을 줄 수 있는 테스트이다.

<!--more-->

## 2.1 UserDaoTest 다시 보기

### 2.1.1 테스트의 유용성

만든 코드는 어떤 방법으로든 테스트해야 한다.

코드의 구조와 설계, 적용한 기술이 변경되더라도 처음의 기능을 잘 수행한다는 걸 테스트할 수 없다면 자신감있게 코드를 수정할 수 없다.

테스트란 내가 예상하고 의도했던 대로 코드가 정확히 동작하는지를 확인해고 확신할 수 있게 해주는 작업이다.

### 2.1.2 UserDaoTest의 특징

```java
public class UserDaoTest {
  public static void main(String[] args) throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext("applicationcontext.xml");

    UserDao dao = context.getBean("userDao", UserDao.class);

    User user = new User();
    user.setId("user");
    user.setName("백기선”);
    user.setPassword("married");
    dao.add(user);

    System.out.println(user.getId() + " 등록 성공”);

    User user2 = dao.get(user.getId());
    System.out.println(user2.getName());
    System.out.println(user2.getPassword());

    System.out.println(user2.getId() + " 조회 성공");
  }
}
```

UserDaoTest 내용을 정리해보면 아래와 같다.

- 자바에서 가장 손쉽게 실행 가능한 main() 메소드를 이용한다.
- 테스트할 대상인 UserDao 의 오브젝트를 가져와 메소드를 호출한다.
- 테스트에 사용할 입력 값(User 오브젝트)을 직접 코드에서 만들어 넣어준다.
- 테스트의 결과를 콘솔에 출력해준다.
- 각 단계의 작업이 에러 없이 끝나면 콘솔에 성공 메시지로 출력해준다.

보통 웹 프로그램에서 DAO 를 테스트하기 위해서는 대충이라도 모든 계층의 기능을 만들고 웹 화면을 통해 호출을 해봐야 확인이 가능하다.

이런 테스트 방식은 테스트를 위해 많은 부차적인 기능을 만들어야 하고, 그리고 에러가 발생했을 때 문제를 찾기가 어렵다는 문제가 있다.

오류에 대해 빠르고 정확하게 대응하기 힘들다는 문제가 있다.

테스트 대상이 명확하면 대상에 집중해서 테스트하는 것이 바람직하고, 가능하면 작은 단위로 쪼개서 집중해서 수행해야 한다.

관심사의 분리라는 원리가 여기서도 적용된다.

작은 단위의 코드에 대해 테스트를 수행하는 것을 단위 테스트 Unit test 라고 한다.

단위의 크기가 정해진 것은 없지만 단위를 넘어서는 다른 코드를 신경쓰지 않고 테스트가 동작해야만 한다.

외부의 리소스에 의존하는 테스트는 단위테스트가 아니라고 보기도 한다.

단위 테스트를 하는 이유는 개발자가 설계하고 만든 코드가 원래 의도한대로 동작하는지 확인하기 위해서다.

오류는 코드를 만들자마자 테스트해서 발견하는 편이 큰 단위의 테스트에서 발견하는 것보다 좋다.

UserDaoTest 는 main() 메소드를 실행하는 간단한 방법으로 테스트를 진행한다.

테스트는 이렇게 자동으로 수행되도록 하는 것이 중요하다.

자동으로 수행되는 테스트는 자주 반복할 수 있다.

테스트할 클래스에 main() 메소드를 포함시키는 것보다 별도로 테스트용 클래스를 만들어서 하는 편이 낫다.

테스트는 지속적인 개선과 점진적인 개발을 위해 반드시 필요하다.

테스트가 없으면 코드를 수정하고 설계를 개선해가더라도 결과를 확신할 수가 없다.

기능을 추가할 때도 테스트 코드는 유용하게 쓰일 수 있다.

새로운 기능을 추가하고 그에 대한 테스트를 함께 추가하는 식으로 점진적인 개발이 가능하다.

### 2.1.3 UserDaoTest 의 문제점

UserDaoTest 는 테스트 입력값을 자동으로 준비해준다는 장점이 있다.

하지만 사람의 눈으로 과정을 확인해야하는 단점이 있다.

또한 main() 메소드로 동작하는 방식보다는 좀 더 편리하고 체계적으로 테스트를 실행할 방법이 필요하다.
