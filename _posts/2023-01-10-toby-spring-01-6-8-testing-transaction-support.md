---
# layout: single
title: "6장 AOP - 6.8 트랜잭션 지원 테스트"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

## 6.8 트랜잭션 지원 테스트

### 6.8.1 선언적 트랜잭션과 트랜잭션 전파 속성

트랜잭션 전파 속성은 유용한 개념이다.

REQUIRED 전파 속성은 진행 중인 트랜잭션이 있으면 참여하고 없으면 새로운 트랜잭션을 시작해주는 속성이다.

이를 통해 비즈니스 로직상 코드가 중복되는 일을 피할 수 있고, 애플리케이션을 작은 기능 단위로 쪼개서 개발할 수도 있다.

트랜잭션 전파라는 기법을 사용했기 때문에 작은 메소드가 독자적인 트랜잭션 단위가 될 수도 있고, 다른 트랜잭션의 일부로 참여할 수도 있는 것이다.

AOP 를 이용해 코드 외부에서 트랜잭션의 기능을 부여해주고 속성을 지정할 수 있게 하는 방법을 선언적 트랜잭션 declarative transaction 이라고 한다.

반대로 TransactionTemplate 나 개별 데이터 기술의 트랜잭션 API 를 사용해 직접 코드 안에서 사용하는 방법을 프로그램에 의한 트랜잭션 programmatic transaction 이라고 한다.

특별한 경우가 아니리ㅏ면 선언적 방식의 트랜잭션을 사용하는 것이 바람직하다.

### 6.8.2 트랜잭션 동기화와 테스트

트랜잭션의 자유로운 전파의 기술적 배경에는 AOP 와 스프링의 트랜잭션 추상화가 있다.

#### 트랜잭션 매니저와 트랜잭션 동기화

트랜잭션 추상화 기술의 핵심은 트랜잭션 매니저와 트랜잭션 동기화이다.

트랜잭션 매니저를 통해 구체적인 트랜잭션 기술의 종류에 상관없이 일관된 트랜잭션 제어가 가능했다.

트랜잭션 동기화 기술을 통해  시작된 트랜잭션 정보를 저장소에 보관했다가 DAO 에서 공유할 수가 있었다.

특히 트랜잭션 동기화 기술은 트랜잭션 전파를 위해 중요한 역할을 한다.

트랜잭션 동기화 기술 덕분에 진행중인 트랜잭션이 있는지 확인하고, 참여할 수 있게 만드는게 가능하다.

트랜잭션 매니저를 통해 트랜잭션을 제어하는 것이기 때문에 트랜잭션 매니저를 이용해 트랜잭션에 참여하는 것도 가능하다.

#### 트랜잭션 매니저를 이용한 테스트용 트랜잭션 제어

트랜잭션 매니저를 이용해 각자의 트랜잭션으로 동작하던 메소드를 하나로 통합할 수 있다. 

트랜잭션을 시작하기 위해 트랜잭션 정의를 담은 오브젝트를 만들고 이를 트랜잭션에 제공하면서 새로운 트랜잭션을 요청한다.

```java
@Test
public void transactionSync() {
    DefaultTransactionDefinition txDefinition = new DefaultTransactionDefinition();
    TransactionStatus txStatus = transactionManager.getTransaction(txDefinition);

    userDao.deleteAll();
    userDao.add(user1);
    userDao.add(user2);
    
    transactionManager.commit(txStatus);
}
```

세 개의 메소드의 트랜잭션 속성이 모두 REQUIRED 이므로 이미 시작된 트랜잭션이 있으면 참여하게 된다.

#### 트랜잭션 동기화 검증



#### 롤백 테스트



### 6.8.3 테스트를 위한 트랜잭션 애노테이션



#### @Transactional



#### @Rollback



#### @TransactionConfiguration



#### NotTransactional과 Propagation.NEVER



#### 효과적인 DB 테스트



## 6.9 정리
