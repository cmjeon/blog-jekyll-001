---
# layout: single
title: "6장 AOP - 6.2 고립된 단위 테스트"
categories:
  - "토비의 스프링 3.1"
tags:
  - "AOP"
---

## 6.2 고립된 단위 테스트

가장 편하고 좋은 테스트 방법은 가능한 한 작은 단위로 쪼개서 테스트하는 것이다.

### 6.2.1 복잡한 의존관계 속의 테스트

UserService 의 구현 클래스가 동작하려면 세 가지 타입의 의존 오브젝트가 필요하다.

원래 UserServiceTest 가 테스트하고자 하는 대상인 UserService 는 사용자 정보를 관리하는 비즈니스 로직의 구현코드이다.

UserService 구현 클래스의 코드가 바르게 작성되어 있다면 성공하고, 아니면 실패하면 된다.

하지만 UserService 는 UserDao, TransactionManager, MailSender 라는 세 가지 의존관계를 가지고 있어서 테스트가 진행되는 동안 이 오브젝트들이 같이 실행된다.

이런 경우는 테스트 준비가 힘들고, 동일한 결과가 나오지 않는 등 부작용이 발생할 수 있어서 테스트를 작성하고 실행하는 것을 기피하는 원인이 된다.

### 6.2.2 테스트 대상 오브젝트 고립시키기

테스트의 대상이 환경이나, 외부 서버, 다른 클래스의 코드에 종속되고 영향을 받지 않도록 고립시킬 필요가 있다.

테스트를 의존 대상으로부터 분리해서 고립시키는 방법은 테스트를 위한 대역 Mock 을 사용하는 것이다.

#### 테스트를 위한 UserServiceImpl 고립

고립된 테스트가 가능하도록 UserService 를 재구성해보자.

UserServiceImpl 을 MockUserDao 와 MockMailSender 에 의존하게 한다.

트랜잭션 관련코드가 분리되어 UserServiceImpl 에서는 더이상 PlatformTransactionManager 를 의존하지 않는다.

MockUserDao 는 코드가 정상적으로 수행되도록 도와주고, 부가적인 검증 기능까지 가지도록 만든다.

UserServiceImpl 과의 사이에서 주고받은 정보를 저장해뒀다가, 테스트의 검증에 사용할 수 있게 만들 필요가 있다.

#### 고립된 단위 테스트 활용

upgradeLevels() 테스트를 살펴보면 다섯 단계의 작업으로 구성된 것을 알 수 있다.

```java
@Test
public void upgradeLevels() throws Exception {
    // 1.
    userDao.deleteAll();
    for(User user : users) userDao.add(user);
    
    // 2.
    MockMailSender mockMailSender = new MockMailSender();
    userServiceImpl.setMailSender(mockMailSender);
    
    // 3.
    userService.upgradeLevels();
    
    // 4.
    checkLevelUpgraded(user.get(0), false);
    checkLevelUpgraded(user.get(1), true);
    checkLevelUpgraded(user.get(2), false);
    checkLevelUpgraded(user.get(3), true);
    checkLevelUpgraded(user.get(4), false);
    
    // 5.
    List<String> request = mockMailSender.getRequest();
    assertThat(request.size(), is(2));
    assertThat(request.get(0), is(users.get(1).getMail()));
    assertThat(request.get(1), is(users.get(3).getMail()));
}
```

1. UserDao 를 통해 가져올 테스트 정보를 사전에 설정한다.
2. MailSender 목 오브젝트를 DI 해준다.
3. userService 의 메소드를 실행한다.
4. UserDao 를 이용해 데이터를 조회해서 결과를 확인한다.
5. 목 오브젝트를 통해 메일 발송이 되었는지를 확인한다.

#### UserDao 목 오브젝트

UserDao 와 DB 에 의존하고 있는 테스트 방식을 목 오브젝트를 만들어 적용해본다.

UserDao 의 메소드를 사용하는 경우는 getAll(), update(user) 두 가지이다.

getAll() 은 미리 준비된 사용자 목록을 제공해주면 된다.

update() 는 '전체 사용자 중에서 업그레이드 대상자는 레벨을 변경해준다' 는 부분을 검증할 수 있는 기능이기 때문에 변경된 내용의 보관이 필요하다.

따라서 getAll() 은 스텁으로서, update() 에 대해서는 목 오브젝트로서 동작하는 타입의 테스트 대역이 필요하다.

이 클래스의 이름을 MockUserDao 라고 하고, UserServiceTest 내부에 만들자

```java
static class MockUserDao implements UserDao {
    
    private List<User> users;
    private List<User> updated = new ArrayList();
    
    private MockUserDao(List<User> users) {
        this.users = users;
    }
    
    public List<User> getUpdated() {
        return this.updated;
    }
    
    public List<User> getAll() {
        return this.users;
    }
    
    public void update(User user) {
        update.add(user);
    }
    
    public void add(User user) { throw new UnsupportedOperationException(); }
    public void deleteAll() { throw new UnsupportedOperationException(); }
    public void get(String id) { throw new UnsupportedOperationException(); }
    public void getCount() { throw new UnsupportedOperationException(); }
    
}
```

테스트에서 사용하지 않을 메소드도 구현해줘야 한다면 UnsupportedOperationException 을 던지도록 만드는 편이 좋다.

MockUserDao 에는 두 개의 List<User> 를 정의해둔다. 

users 는 getAll() 메소드가 호출되면 DB 에서 가져온 것처럼 돌려주는 용도이고, updated 는 update() 메소드를 실행하면서 넘겨준 업그레이드 대상 User 오브젝트를 저장해두었다가 검증할 때 돌려주기 위한 용도이다.

upgradeLevels() 테스트가 MockUserDao 를 사용하도록 수정한다.

```java
@Test
public void upgradeLevels() throws Exception {
    UserServiceImpl userServiceImpl = new UserServiceImpl();
    
    MockUserDao mockUserDao = new MockUserDao(this.users);
    userServiceImpl.setUserDao(mockUserDao);

    MockMailSender mockMailSender = new MockMailSender();
    userServiceImpl.setMailSender(mockMailSender);

    userService.upgradeLevels();

    List<User> updated = mockUserDao.getUpdated();
    assertThat(updated.size(), is(2));
    checkUserAndLevel(updated.get(0), "joytouch", Level.SILVER);
    checkUserAndLevel(updated.get(1), "madnite1", Level.GOLD);
    
    List<String> request = mockMailSender.getRequest();
    assertThat(request.size(), is(2));
    assertThat(request.get(0), is(users.get(1).getMail()));
    assertThat(request.get(1), is(users.get(3).getMail()));
}
```

고립된 테스트로 만들기 전의 테스트의 대상은 스프링 컨테이너에서 @Autowired 를 통해 가져온 UserService 타입의 빈이었다.

이제는 완전히 고립돼서 테스트만을 위해 독립적으로 동작하는 테스트 대상을 사용할 것이기 때문에 스프링 컨테이너에서 빈을 가져올 필요가 없다.

따라서 먼저 테스트하고 싶은 로직을 담은 클래스인 UserServiceImpl 의 오브젝트를 직접 생성하고, 스프링의 테스트 컨텍스트를 이용하기 위해 도입한 @RunWith 등을 제거한다.

MockUserDao, MockMailSender 오브젝트를 DI 해준다.

#### 테스트 수행 성능의 향상

upgrageLevels() 의 테스트 수행시간이 이전보다 빨라진 것을 알 수 있다.

고립된 테스트를 하면 테스트가 다른 의존 대상에 영향을 받을 경우를 대비해 복잡하게 준비할 필요가 없을 뿐만 아니라, 테스트 수행 성능도 크게 향상된다.

고립된 테스트를 만들려면 목 오브젝트 작성과 같은 약간이 수고가 더 필요할지 모르겠지만, 그 보상은 충분히 기대할 만하다.

### 6.2.3 단위 테스트와 통합 테스트










