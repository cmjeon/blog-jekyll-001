---
# layout: single
title: "5장 서비스 추상화 - 5.4 메일 서비스 추상화"
# excerpt: "DAO 에 트랜잭션을 적용해보면서 스프링이 어떻게 성격이 비슷한 여러 기술을 추상화하고, 일관된 방법으로 사용할 수 있도록 지원하는지 알아봅니다."
categories:
  - "토비의 스프링 3.1"
tags:
  - "서비스 추상화"
---

## 5.4 메일 서비스 추상화

### 5.4.1 JavaMail을 이용한 메일 발송 기능

레벨이 업그레이드 되는 사용자에게 안내 메일을 발송하는 기능을 만들어보겠습니다.

먼저 email 필드를 추가하고 테스트를 해 봅니다.

1. User 테이블에 email 필드를 추가
2. User 클래스에 email 프로퍼티 추가
3. 테스트 코드 수정
4. 테스트 수행

#### JavaMail 메일 발송

자바에서 메일을 발송할 때는 표준 기술인 JavaMail 을 사용합니다.

```java
// ...

public class UserService {
	
  // ...
	
  protected void upgradeLevel(User user) {
    user.upgradeLevel();
    userDao.update(user);
    // highlight-next-line
    sendUpgradeEMail(user);
  }
	
  private void sendUpgradeEMail(User user) {
    Properties props = new Properties();
    props.put("mail.smtp.host", "mail.ksug.org");
    Session s = Session.getlnstance(props, null);
    
    MimeMessage message = new MimeMessage(s); 
    
    try {
      message.setFrom(new InternetAddress("useradmin@ksug.org"));
      message.addRecipient(Message.RecipientType.TO, new InternetAddress(user.getEmail())); message.setSubject("Upgrade 안내");
      message.setText("사용자님의 등급이 " + user.getLevel().name() + “로 업그레이드되었습니다");
      
      Transport.send(message);
    } catch (AddressException e) {
      throw new RuntimeException(e); 
    } catch (MessagingException e) {
      throw new RuntimeException(e);
    } catch (UnsupportedEncodingException e) {
      throw new RuntimeException(e);
    }
  }
}
```

### 5.4.2 JavaMail이 포함된 코드의 테스트

sendUpgradeEMail() 메소드에 대한 테스트는 메일 서버가 준비되지 않은 환경에서도 테스트는 동작하여야 합니다.

UserService 에 테스트용 JavaMail 을 연결해서 요청하는 방식으로 변경해 봅니다.

### 5.4.3 테스트를 위한 서비스 추상화

테스트를 위해 JavaMail 과 동일한 인터페이스를 갖는 테스트용 오브젝트를 만들어서 사용하면 될 것 같습니다.

#### JavaMail을 이용한 테스트의 문제점

그런데 JavaMail 의 API 는 이 방법을 적용할 수 없습니다.

JavaMail 의 핵심 API 에는 구현을 바꿀 수 있는게 없습니다.

Session 과 Transport 가 인터페이스가 아닌 구현체이기 때문에 테스트용 JavaMail 을 만들 수 없기 때문입니다.

스프링에서는 JavaMail 을 사용해 만든 코드가 테스트하기 힘들다는 문제를 해결하기 위해 추상화 기능을 제공하고 있습니다.

#### 메일 발송 기능 추상화

메일 발송 기능 추상화로 MailSender 라는 인터페이스를 제공하고 있습니다.

사용을 위해서는 JavaMailSenderImpl 클래스를 이용하면 됩니다.

MailSender 인터페이스에는 SimpleMailMessage 라는 인터페이스를 구현한 클래스를 파라미터로 받는 메소드로만 구성되어 있습니다.

기본적으로는 MailSender 인터페이스의 구현 클래스인 JavaMailSenderImpl 을 이용하여 메일 발송용 코드를 만듭니다.

```java

// ...

private void sendUpgradeEMail(User user) {
  // highlight-next-line
  JavaMailSenderlmpl mailSender = new JavaMailSenderlmpl();
  mailSender.setHost("mail. server. com");

  SimpleMailMessage mailMessage = new SimpleMailMessage();
  mailMessage.setTo(user.getEmail());
  mailMessage.setFrom("useradmin@ksug.org");
  mailMessage.setSubject("Upgrade 안내");
  mailMessage.setText("사용자님의 등급이 " + user.getLevel().name());
  
  mailSender.send(mailMessage);
}
```

이제 mailSender 를 DI 해 봅니다.

sendUpgradeEMail() 메소드에는 JavaMailSenderImpl 클래스가 구현한 MailSender 인터페이스만 남깁니다.

구체적인 메일 전송 구현을 담은 클래스의 정보는 코드에서 모두 제거할 수 있습니다.

```java
// ...

import org.springframework.mail.MailSender;
import org.springframework.mail.SimpleMailMessage;

public class UserService {

  // ...
	
  // highlight-next-line
  private MailSender mailSender;
	
  // highlight-start
  public void setMailSender(MailSender mailSender) {
    this.mailSender = mailSender;
  }
  // highlight-end

  // ...
	
  private void sendUpgradeEMail(User user) {
    SimpleMailMessage mailMessage = new SimpleMailMessage();
    mailMessage.setTo(user.getEmail());
    mailMessage.setFrom("useradmin@ksug.org");
    mailMessage.setSubject("Upgrade 안내");
    mailMessage.setText("사용자님의 등급이 " + user.getLevel().name());
		
    // highlight-next-line
    this.mailSender.send(mailMessage);
  }
	
}
```

#### 테스트용 메일 발송 오브젝트

테스트를 위해 MailSender 인터페이스의 구현 클래스인 DummyMailSender 를 생성합니다.

그리고 테스트 설정파일에서 DummyMailSender 을 DI 하도록 설정합니다.

이렇게 하면 UserServiceTest 의 upgradeAllOrNothing() 테스트가 통과됩니다.

#### 테스트와 서비스 추상화

이처럼 서비스 추상화를 이용하면 트랜잭션 같은 기능에 대한 추상 인터페이스를 만들 수도 있습니다.

JavaMail 처럼 테스트가 어려운 API 를 사용할 때도 유용하게 쓸 수 있습니다.

JavaMail 이 아닌 다른 API 를 사용해도 그에 맞는 MailSender 구현 클래스를 만들어서 DI 해주면 됩니다.

어떤 경우에도 UserService 와 같은 애플리케이션 계층의 코드는 메일 발송을 요청한다는 기능에 맞게 작성되면 됩니다.

주로 외부의 리소스와 연동하는 부분의 작업이 서비스 추상화의 대상이 될 수 있습니다.

### 5.4.4 테스트 대역

#### 의존 오브젝트의 변경을 통한 테스트 방법

테스트 대상이 되는 오브젝트가 또 다른 오브젝트에 의존하는 일은 매우 흔합니다.

트랜잭션과 메일의 사례에서도 확인하였지만 스프링 DI 를 통해 의존하는 오브젝트를 테스트용으로 변경하는 것이 가능합니다.

#### 테스트 대역의 종류와 특징

테스트용으로 사용되는 특별한 오브젝트들을 테스트 대역 test double 이라고 합니다.

대표적인 테스트 대역은 테스트 스텁 test stub 입니다.

테스트 스텁이 결과를 반환해야 하는 경우도 있는데, 이런 경우에는 목 오브젝트 mock object 를 사용합니다.

#### Mock 오브젝트를 이용한 테스트

UserServiceTest 에 목 오브젝트를 적용해 봅니다.

우선 MockMailSender 클래스를 만듭니다.

```java
static class MockMailSender implements MailSender {

  private List<String> requests = new ArrayList<String>();
  
  public List<String> getRequests() { 
    return requests;
  }
  
  public void send(SimpleMailMessage mailMessage) throws MailException {
    requests.add(mailMessage.getTo()[0]);
  }

  public void send(SimpleMailMessage[] mailMessage) throws MailException {
  }
  
}
```

MockMailSender 클래스는 UserService 가 send() 메소드를 통해 자신을 호출하면 관련 정보를 목록에 저장해둡니다.

그리고 테스트에서 읽어갈 수 있도록 getRequests() 메소드를 만들어서 목록을 제공합니다.

```java
public class UserServiceTest {

  @Test
  // highlight-next-line 
  @DirtiesContext
  public void upgradeLevels() {
    userDao.deleteAll();
    for(User user : users) userDao.add(user);
		
    // highlight-start
    MockMailSender mockMailSender = new MockMailSender();  
    userService.setMailSender(mockMailSender);
    // highlight-end  
		
    userService.upgradeLevels();
		
    checkLevelUpgraded(users.get(0), false);
    checkLevelUpgraded(users.get(1), true);
    checkLevelUpgraded(users.get(2), false);
    checkLevelUpgraded(users.get(3), true);
    checkLevelUpgraded(users.get(4), false);
		
    List<String> request = mockMailSender.getRequests();  
    assertThat(request.size(), is(2));  
    assertThat(request.get(0), is(users.get(1).getEmail()));  
    assertThat(request.get(1), is(users.get(3).getEmail()));  
  }
	
}
```

@DirtiesContext 애노테이션은 컨텍스의 DI 설정을 변경하는 테스트라는 것을 알려줍니다.

목 오브젝트를 이용한 테스트는 상당히 막강합니다.

목 오브젝트는 테스트 대상 오브젝트의 내부에서 일어나는 일이나 다른 오브젝트 사이에서 주고받는 정보까지 검증하는 일을 손쉽게 해줍니다.

테스트가 수행될 수 있도록 의존 오브젝트에 간접적으로 입력 값을 제공해주는 `스텁 오브젝트`와 간접적인 출력 값까지 확인이 가능한 `목 오브젝트`가 있습니다.

이 두 가지는 테스트 대역의 가장 대표적인 방법이며 효과적인 테스트 코드를 작성하는 데 빠질 수 없는 중요한 도구입니다.

## 5.5 정리

- 비즈니스 로직을 담은 코드는 데이터 액세스 로직을 담은 코드와 깔끔하게 분리되는 것이 바람직하다. 비즈니스 로직 코드 또한 내부적으로 책임과 역할에 따라서 깔끔하게 메소드로 정리돼야 한다.
- 이를 위해서는 DAO 의 기술 변화에 서비스 계층의 코드가 영향을 받지 않도록 인터페이스와 이를 잘 활용해서 결합도를 낮춰줘야 한다.
- DAO 를 사용하는 비즈니스 로직에는 단위 작업을 보장해주는 트랜잭션이 필요하다.
- 트랜잭션의 시작과 종료를 지정하는 일을 트랜잭션 경계설정이라고 한다. 트랜잭션 경계설정은 주로 비즈니스 로직 안에서 일어나는 경우가 많다.
- 시작된 트랜잭션 정보를 담은 오브젝트를 파라미터로 DAO 에 전달하는 방법은 매우 비효율적이기 때문에 스프링이 제공하는 트랜잭션 동기화 기법을 활용하는 것이 편리하다.
- 자바에서 사용되는 트랜잭션 API 의 종류와 방법은 다양하다. 환경과 서버에 따라서 트랜잭션 방법이 변경되면 경계설정 코드도 함께 변경돼야 한다.
- 트랜잭션 방법에 따라 비즈니스 로직을 담은 코드가 함께 변경되면 단일 책임 원칙에 위배되며, DAO 가 사용하는 특정 기술에 대해 강한 결합을 만들어낸다.
- 트랜잭션 경계설정 코드가 비즈니스 로직 코드에 영향을 주지 않게 하려면 스프링이 제공하는 트랜잭션 서비스 추상화를 이용하면 된다.
- 서비스 추상화는 로우레벨의 트랜잭션 기술과 API 의 변화에 상관없이 일관된 API 를 가진 추상화 계층을 도입한다.
- 서비스 추상화는 테스트하기 어려운 JavaMail 같은 기술에도 적용할 수 있다. 테스트를 편리하게 작성하도록 도와주는 것만으로도 서비스 추상화는 가치가 있다.
- 테스트 대상이 사용하는 의존 오브젝트를 대체할 수 있도록 만든 오브젝트를 테스트 대역이라고 한다.
- 테스트 대역은 테스트 대상 오브젝트가 원활하게 동작할 수 있도록 도우면서 테스트를 위해 간접적인 정보를 제공해주기도 한다.
- 테스트 대역 중에서 테스트 대상으로부터 전달받은 정보를 검증할 수 있도록 설계된 것을 목 오브젝트라고 한다.
