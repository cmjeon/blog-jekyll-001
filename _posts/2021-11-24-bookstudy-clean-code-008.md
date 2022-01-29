---
layout: single
title: 클린코드 스터디 - 008
categories: 
  - bookstudy - cleancode
tags: 
  - cleancode
---

## 8장 경계

### 외부 코드 사용하기

java.util.Map 의 예시를 살펴보자

Map 의 기능들은 확실히 유용하지만 위험도 크다

예컨데 map의 사용자라면 누구나 map 내용을 지울 권한이 있고, map 은 객체 유형을 제한하지 않음

한 번 Sensor 객체를 담는 Sensors 를 map 으로 만들어 보자

```java
Map sensors = new HashMap();
Sensor sensor = (Sensor) sensors.get(sensorId);
```

가독성이 떨어지므로, 좀 더 가독성을 높일 방법을 찾아보자

```java
Map<String, Sensor> sensors = new HashMap<Sensor>();
Sensor sensor = sensors.get(sensorId);
```

이 코드에서 불필요한 Map 의 다른 기능들이 사용가능하다

Sensors 를 만들어서 사용자로부터 제네릭이나 다른 기능들을 감춰보자

```java
public class Sensors {
  private Map sensors = new HansMap();
  public Sensor getById(String id) {
    return (Sensor) sensors.get(id);
  }
}
…
Sensors sensors = new Sensors();
Sensor sensor = sensors.getById(sensorId);
```

### 경계 살피고 익히기

외부 코드를 사용하면 적은 시간에 더 많은 기능을 출시하기 쉬워짐

우리가 라이브러리를 익히는 기존의 방법

하루, 이틀 문서를 읽으며 사용법을 결정
우리쪽 코드를 작성해 라이브러리가 예상대로 동작하는지 확인
디버깅

저자가 제안하는 새로운 방법

테스트 케이스를 작성
통제된 환경에서 외부 코드를 이해하는지 확인

외부 라이브러리를 익히는 새로운 방법을 `학습 테스트`라고 부름

### log4j 익히기

log4j 라이브러리(패키지)를 이용해보면서 `학습 테스트`를 실습해보자

화면에 hello 라는 로그를 출력하는 테스트케이스를 만들어 본다

```java
@Test
public void testLogCreate() {
  Logger logger = Logger.getLogger(“MyLogger”);
  logger.info("'Hello");
}
```

Appender … 라는 오류가 발생함

그래서 ConsoleAppender 를 생성하고 다시 돌려봄

```java
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger(“MyLogger”);
  ConsoleAppender appender = new ConsoleAppender();
  logger.addAppender(appender);
  logger.info("hello");
}
```

출력 스트림이 없다는 사실을 발견함

new ConsoleAppender 에 PatternLayout 과 ConsoleAppender 인수를 추가함

```java
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger(“MyLogger”);
  logger.removeAllAppenders();
  logger.addAppender(new ConsoleAppender(
    new PatternLayout(“%p %t %m%n”),
    ConsoleAppender.SYSTEM_OUT));
  logger.info("hello");
}
```

콘솔에 “hello’ 가 출력됨

ConsoleAppender.SYSTEM_OUT 인수를 제거해도 문제없음 하지만 PatternLayout 을 제거했더니 오류가 발생

여기서 log4j 의 버그를 발견할 수 있음

최종적으로 단위 테스트 케이스는 아래와 같음

```java
public class Log4jTest {
  private Logger logger;
  
  @Before
  public void initialize() {
    logger = Logger.getLogger(“logger”);
    logger.removeAllAppenders();
    Logger.getRootLogger().removeAllAppenders();
  }

  @Test
  public void basicLogger() {
    BasicConfigurator.configure();
    logger.info("BasicLogger");
  }

  @Test
  public  void addAppenderWithStream() {
    logger.addAppender(new ConsoleAppender(
      new PattenrLayout(“%p %t %m%n”),
      ConsoleAppender.SYSTEM_OUT));
      logger.info("addAppenderWithSteam");
  }

  @Test
  public void addAppenderWithoutStream() {
    logger.addAppender(new ConsoleAppender(
      new PatternLayout(“%p %t %m%n”)));
      logger.info("addAppenderWithoutStream");
  }
}
```

### 학습 테스트는 공짜 이상이다

학습 테스트는 이해도를 높여주는 실험임

투자하는 노력대비 얻는 성과가 큼

외부 라이브러리의 새 버전이 나오면 학습 테스트를 돌려서 차이를 확인할 수 있음

이러한 학습 테스트가 있으면 새 버전의 외부 라이브러리로 이전이 쉬워짐

필요 이상 낡은 버전의 외부 라이브러리를 유지하려고 노력할 필요가 없음

### 아직 존재하지 않는 코드를 사용하기

경계와 관련한 다른 사례는 송신기 모듈에 대한 기능

저자가 원한 송신기 모듈의 기능

> 지정한 주파수를 이용해 이 스트림에서 들어오는 자료를 아날로그 신호로 전송하라

송신기 모듈에 대한 구체적인 API 가 나오지 않은 상황

Communication Controller 가 송신기 모듈을 사용해아함

저자가 처리한 방법

Transmitter 인터페이스를 만들고 transmit 메소드를 추가함
transmit 메소드는 주파수와 스트림을 인수로 받음
Communication Controller 는 Transmitter 의 tranmit 메소드를 사용함
처음에는 Transmitter 인터페이스를 구현한 FakeTransmitter 를 사용
실제 Transmitter API 가 만들어지면 Transmitter Adapter 를 구현하여 사용

이런 설계는 테스트에도 용이함

FakeTransmitter 를 사용하여 테스트하다 실제 API 가 나오면 경계 테스트 케이스를 생성해 테스트 진행

### 깨끗한 경계

경계에서는 흥미로운 일이 벌어짐

소프트웨어 설계가 우수하면 변경에 많은 투자와 재작업이 필요하지 않음

경계에 위치하는 코드는 깔끔히 분리하고 기대치를 정의하는 테스트 케이스를 작성함

통제가 가능한 우리 코드에 의존하도록 설계하라

새로운 클래스로 외부 라이브러리를 감싸기
Adapter Pattern 을 사용하여 외부 라이브러리가 제공하는 인터페이스를 우리가 원하는 인터페이스로 변환

어느 방법이든 가독성이 높아지고, 경계 인터페이스를 사용하는 일관성도 높아지고, 외부 라이브러리가 변했을 때 변경할 코드도 줄어듬

## 참고
- 