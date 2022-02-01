---
layout: single
title: 클린코드 스터디 - 010
categories: 
  - bookstudy - cleancode
tags: 
  - cleancode
---

## 10장 클래스

### 클래스 체계

변수목록 : 정적 공개 상수 → 정적 비공개 변수

함수 : 공개 함수 → 비공개 함수

#### 캡슐화

- 캡슐화를 풀어주는 결정은 항상 최후의 수단으로 사용한다.
- 테스트 코드가 함수를 호출하거나 변수를 사용해야 한다면, protected로 선언하여 패키지 내에 한해 전체공개를 할 수 있다.
- 그래도 최대한 비공개 상태를 유지하는 것이 좋다.

### 클래스는 작아야 한다

- 함수의 크기는 물리적인 행의 수로 크기를 측정
- 클래스는 행의 수가 아닌, 책임의 수로 크기를 측정
- 클래스는 최대한 적은 책임을 가져야한다.

#### 단일 책임 원칙(Single Responsibility Principle)

- 클래스나 모듈은 변경해야 할 이유가 하나뿐이어야 하는 원칙이다.
- 책임을 단일화 하려고 애쓰다 보면, 코드를 추상화하기도 쉬워진다.
- SRP를 따르다 보면 클래스가 많아지게 되는데, 이는 큰 그림을 이해하기 어려워진다고들 하지만, 규모가 어느 수준 이상이 되어 복잡성을 다루기 위해서는 체계적인 정리가 필수다.
  
    → 도구 상자에 작은 칸막이가 많은 것 VS 도구 상자 한 칸막이에 다 때려박는 것

#### 응집도

- 클래스는 인스턴스 변수의 수가 작아야 한다.
- 인스턴스 변수를 사용하는 메서드 : 응집도 비례
- 모든 메서드가 인스턴스 변수를 사용하는 것이 응집도가 가장 높다.
- 하지만, 일부의 메서드만 인스턴스 변수에 접근한다고 하는 상황이생기면, 해당 메서드와 변수를 모아서 작은 클래스로 쪼개는 것이 낫다.

#### 변경하기 쉬운 클래스

- 깨끗한 시스템은 클래스를 체계적으로 정리하여, 변경이 일어날 때 수반하는 위험을 낮춘다.
- 어떠한 변경이든 클래스에 손대면 다른 곳을 망가뜨리는 문제가 발생할 수 있다.
- 새 기능을 수정하거나 기존기능을 변경할 때, 건드릴 코드가 최소인 시스템 구조가 바람직하다.
- 새 기능을 추가할 때, 시스템을 확장할 뿐, 기존코드를 변경하지 않아야 한다. (Adapter Pattern)

```java
public class Sql {
  public Sql(String table, Columns[] columns);
  public String create();
  public String insert(Object[] fields);
  public String selectAll();
  public String findByKey(String keyColumn, String keyValue);
  ...
}
```

위의 클래스 Sql에서 추가적으로 다른 기능을 지원한다고 했을 때, 반드시 Sql문에 손을 대야한다.

```java
abstract public class Sql {
  public Sql(String table, Columns[] columns);
  abstract public String generate();
}

public class CreateSql extends Sql {
  public CreateSql(String table, Column[] columns);
  @Override 
  public String generate();
}

public class SelectSql extends Sql {
  public selectSql(String table, Columns[] columns);
  @Override
  public String generate();
}

...
```

위처럼 Sql을 추상클래스로 생성하여, 모든 클래스를 Sql클래스에서 파생하는 클래스로 만들었다.
- 각 클래스는 단순해지고 가독성이 좋아졌다.
- update문을 추가한다고 했을 때, 기존 클래스의 변경 없이, 새로운 클래스를 생성하면 된다.

#### 변경으로부터 격리

- 요구사항은 항상 변한다. → 코드도 변한다
- 상세한 구현(코드)에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험할 수 있다.
- 인터페이스와 추상클래스를 사용하여 구현이 미치는 영향을 격리해야 한다.
- 시스템의 결합도를 낮추면 유연성과 재사용성도 높아진다.
- 결합도가 낮다는 소리는 각 시스템 요소들이 잘 분리되어 있고, 변경으로부터 잘 격리되어 있다는 뜻
- 결합도를 최소로 줄이면 DIP(의존성 주입 원칙)을 자연스럽게 따를 수 있다.
- 즉, 상세한 구현이 아니라 추상화에 의존해야한다.

## 참고
- 