---
layout: single
title: "3장 템플릿 - 3.5 템플릿과 콜백"
categories:
  - "토비의 스프링 3.1"
tags:
  - "템플릿"
---

[https://www.yes24.com/Product/Goods/7516911](https://www.yes24.com/Product/Goods/7516911)

# 3장 템플릿

## 3.5 템플릿과 콜백

전략 패턴은 바뀌지 않는 작업 흐름이 존재하고 그 중 일부분만 바꿔서 사용해야 하는 경우 적합한 패턴이다.

앞서 작성된 UserDao, StatementStrategy, JdbcContext 는 전략 패턴의 기본 구조에 익명 내부 클래스를 활용한 방식이다.

이 방식을 스프잉에서는 `템플릿/콜백 패턴` 이라고 부른다.

컨텍스트를 템플릿이라 부르고, 익명 내부 클래스로 만들어지는 오브젝트를 콜백이라고 부른다.

>템플릿(template)은 어떤 목적을 위해 미리 만들어둔 모양이 있는 틀을 말한다.
>
>콜백(callback)은 실행되는 것을 목적으로 다른 오브젝트의 메소드에 전달되는 오브젝트를 말한다.
>
>자바에서는 메소드가 담긴 오브젝트를 전달해야하기 때문에 Functional Object 라고도 한다.

### 3.5.1 템플릿/콜백의 동작원리

템플릿/콜백 패턴에서 콜백은 보통 단일 메소드 인터페이스를 사용한다.

하나의 템플릿이 여러 가지 전략을 사용해야 하면 여러 가지 콜백 오브젝트를 사용한다.

아래는 템플릿/콜백 패턴의 작업 흐름이다.

[![application-context-work](/assets/images/posts/2022-11-29-toby-spring-01-3-4-context-and-di/template-callback-workflow.png)](/assets/images/posts/2022-11-29-toby-spring-01-3-4-context-and-di/template-callback-workflow.png)

클라이언트가 템플릿 메소드를 호출하면서 콜백 오브젝트를 전달하는 것은 메소드 레벨에서 일어나는 DI 이다.

일반적인 DI 는 템플릿에 인스턴스 변수를 만들고 사용할 의존 오브젝트를 수정자 메소드로 받아서 사용한다.

하지만 템플릿/콜백 방식에서는 매번 메소드 단위로 사용할 오브젝트를 새로 전달받는다.

그리고 클라이언트와 콜백이 강하게 결합되어 있다는 특징이 있다.

템플릿/콜백 방식은 전략 패턴과 DI 의 장점을 익명 내부 클래스 사용 전략과 결합한 활용법이다.

템플릿/콜백 패턴을 하나의 디자인 패턴으로 기억해두면 편리하다.

### 3.5.2 편리한 콜백의 재활용

템플릿/콜백 패턴의 아쉬운 점은 익명 내부 클래스 때문에 상대적으로 코드를 작성하고 읽기가 어렵다는 것이다.

#### 콜백의 분리와 재활용

복잡한 익명 내부 클래스에서 바뀌지 않는 부분은 별도의 메소드로 만들어보자.

```java
public void deleteAll() throws SQLException {
  executeSql("delete from users");
}

private void executeSql(final String query) throws SQLException {
  this.jdbcContext.workWithStatementStrategy(
    // highlight-start
    new StatementStrategy() {
      public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        return c.prepareStatement(query);
      }
    }
    // highlight-end
  )
}
```

:::info
변하는 것과 변하지 않는 것을 분리하고 변하지 않는 것은 유연하게 재활용할 수 있게 만든다는 간단한 원리를 계속 적용했을 때 이렇게 단순하면서도 안전하게 작성 가능한 JDBC 활용 코드가 완성된다.
:::

#### 콜백과 템플릿의 결합

executeSql() 메소드를 분리하고 보니 UserDao 만 사용하기에 아깝다.

재사용 가능한 콜백이라면 공유할 수 있도록 템플릿 클래스 안으로 옮겨도 된다.

```java
public class JdbcContext {
  
  // ...
  
  // highlight-start
  public void executeSql(final String query) throws SQLException {
    new StatementStrategy() {
      public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        return c.prepareStatement(query);
      }
    }
  }
  // highlight-end
  
}
```

서로 긴밀하게 연관되어 동작한다면 한 군데 모여 있는게 차라리 유리하다.

### 3.5.3 템플릿/콜백의 응용

스프링은 DI 나 전략패턴, 템플릿/콜백 패턴 등을 활용할 수 있도록 지지한다.

스프링에 내장된 것을 원리도 알지 못한 채로 기계적으로 사용하는 경우와 적용된 패턴을 이해하고 사용하는 경우는 큰 차이가 있다.

스프링은 기본적으로 OCP 를 지키고, 전략 패턴과 DI 를 바탕에 깔고 있으니 원한다면 언제든지 확장해서 사용이 가능하다.

따라서 스프링의 기본인 DI, 전략 패턴, 템플릿/콜백 패턴에 익숙해지도록 학습해야 한다.

고정된 작업 흐름을 갖고 있으면서 여기저기서 자주 반복되는 코드가 있다면 중복되는 코드를 분리할 방법을 생각하는 습관을 기르자.

가장 전형적인 템플릿/콜백 패턴의 후보는 try/catch/finally 블록을 사용하는 코드이다.

#### 테스트와 try/catch/finally

파일안의 숫자를 더하는 테스트를 만든다.

```java
public class CalcSumTest {
  @Test
  public void sumOfNumbers() throws IOException {
    Calculator calculator = new Calculator();
    int sum = calculator.calcSum(getClass().getResource("numbers.txt").getPath());
    assertThat(sum, is(10));
  }
}
```

Calculator 클래스를 만들고 calcSum() 메소드로 파일을 열고 라인을 읽어서 더하는 코드를 작성한다.

```java
public class Calculator() {
  public Integer calcSum(String filepath) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(filepath));
    Integer sum = 0;
    String line = null;
    while((line = br.readLine()) != null) {
      sum += Integer.valueOf(line);
    }
    
    br.close();
    return sum;
  }
}
```

수행중 예외가 발생하면 문제가 발생하기 때문에 try/catch/finally 를 적용해준다.

```java
public class Calculator() {
  public Integer calcSum(String filepath) throws IOException {
    BufferedReader br = null;
    
    try {
      br = new BufferedReader(new FileReader(filepath));
      Integer sum = 0;
      String line = null;
      while((line = br.readLine()) != null) {
        sum += Integer.valueOf(line);
      }
    } catch(IOException e) {
      System.out.println(e.getMessage());
      throw e;
    } finally {
      if (br != null) {
        try { br.close(); }
        catch(IOException e) { System.out.println(e.getMessage()); }
      }
    }
  }
}
```

#### 중복의 제거와 템플릿/콜백 설계

비슷한 코드가 한두번까지는 그냥 넘어가도 세번이상 반복되면 코드를 개선할 시점이라고 생각해야 한다.

> 템플릿/콜백을 적용할 때는 템플릿과 콜백의 경계와 서로가 각각 전달하는 내용이 무엇인지 파악하는게 중요하다.

위의 예시에서는 템플릿이 BufferedReader 를 만들어 콜백에게 전달하고, 콜백이 각 라인을 읽어서 알아서 처리한 후 결과를 템플릿에 돌려주면 된다.

콜백 인터페이스를 만든다.

```java
public interface BufferedReaderCallback {
  Integer doSomethingWithReader(BufferedReader br) throws IOException;
}
```

템플릿부분도 만든다.

```java
public Integer fileReadTemplate(String filepath, BufferedReaderCallback callback) throws IOException {
  BufferedReader br = null;
  
  try {
    br = new BufferedReader(new FileReader(filepath));
    // highlight-start
    int ret = callback.doSomethingWithReader(br);
    // highlight-end
    return ret;
  } catch(IOEceptoin e) {
    System.out.println(e.getMessage());
    throw e;
  } finally {
    if (br != null) {
      try { br.close(); }
      catch(IOException e) { System.out.println(e.getMessage()); }
    }
  }
}    
```

calcSum() 메소드를 만들어서 위에 만든 템플릿에 callback 을 전달하면서 사용한다.

```java
public Integer calcSum(string filepath) throws IOException {
  // highlight-start
  BufferedReaderCallback sumCallback = new BufferedReaderCallback() {
    public Integer doSomethingWithReader(BufferedReader br) throws IOException {
      Integer sum = 0;
      String line = null;
      while(line = br.readLine()) != null) {
        sum += Integer.valueOf(line);
      }
      return sum;
    }
  };
  // highlight-end
  return fileReadTemplate(filepath, sumCallback);
}
```

숫자의 곱을 구하는 메소드도 만들어본다.

우선 테스트를 만든다.

```java
public class CalcSumTest {
  Calculator calculator;
  String numFilepath;
  
  @Before
  public void setUp() {
    this.calculator = new Calculator();
    this.numFilepath = getClass().getResource("numbers.txt").getPath();
  }
  
  @Test
  public void sumOfNumbers() throws IOException {
    assertThat(calculator.calcSum(this.numFilepath), is(10));
  }
  
  @Test
  public void multiplyOfNumbers() throws IOException {
    assertThat(calculator.calcMultiply(this.numFilepath), is(24));
  }
}
```

곱을 계산하는 callback 을 가진 calcMultiply() 메소드를 만든다.

```java
public Integer calcMultiply(string filepath) throws IOException {
  // highlight-start
  BufferedReaderCallback multiplyCallback = new BufferedReaderCallback() {
    public Integer doSomethingWithReader(BufferedReader br) throws IOException {
      Integer multiply = 1;
      String line = null;
      while(line = br.readLine()) != null) {
        multiply *= Integer.valueOf(line);
      }
      return multiply;
    }
  };
  // highlight-end
  return fileReadTemplate(filepath, multiplyCallback);
}
```

#### 템플릿/콜백의 재설계

calcSum() 메소드와 calcMultiply() 메소드 역시 비슷한 부분이 많다.

같은 부분은 변수를 초기화 하고 BufferedReader 를 이용해 파일을 라인으로 읽고, 읽은 내용을 변수에 계산하고, 결과를 리턴하는 것이다.

다른 부분은 라인을 더하거나 곱하는 부분이다.

템플릿과 콜백을 찾아낼 때는 변하는 코드의 경계를 찾고 그 경계를 사이에 두고 주고받는 일정한 정보가 있는지 확인하면 된다.

같은 부분은 템플릿으로 만들고, 다른 부분은 콜백으로 만든다.

먼저 callback 인터페이스를 만든다.

```java
public interface LineCallback {
  Integer doSomethingWithLine(String line, Integer value);
}
```

BufferedReader 로 파일을 읽는 부분을 템플릿으로 만든다.

```java
public Integer lineReadTemplate(String filepath, LineCallback callback, int intVal) throws IOException {
  BufferedReader br = null;
  try {
    br = new BufferedReader(new FileReader(filepaht));
    Integer res = initVal;
    String line = null;
    while(line = br.readLine() != null) {
      // highlight-start 
      res = callback.doSomethingWithLine(line, res);
      // highlight-end
    }
  } catch(IOException e) {
  } finally { ... }
}
```

calcSum(), calcMultiply() 메소드에 템플릿을 적용한다.

```java
public Integer calcSum(string filepath) throws IOException {
  LineCallback sumCallback = new LineCallback() {
    public Integer doSomethingWithLine(String line, Integer value) {
      // highlight-next-line
      return value + Integer.valueOf(line);
    }
  };
  // highlight-next-line
  return lineReadTemplate(filepath, sumCallback, 0);
}

public Integer calcMultiply(string filepath) throws IOException {
  LineCallback multiplyCallback = new LineCallback() {
    public Integer doSomethingWithLine(String line, Integer value) {
      // highlight-next-line
      return value * Integer.valueOf(line);
    }
  };
  // highlight-next-line
  return lineReadTemplate(filepath, multiplyCallback, 0);
}
```

코드의 특성이 바뀌는 경계를 잘 살피고 그것을 인터페이스를 사용해 분리한다는, 가장 기본적인 객체지향 원칙에 충실하면 어렵지 않게 템플릿/콜백 패턴을 만들어 활용할 수 있다.

#### 제네릭스를 이용한 콜백 인터페이스

만약 파일을 라인 단위로 처리해서 만드는 결과의 타입을 다양하게 가져가고 싶다면, 제네릭스를 이용하면 된다.

>제네릭스는 자바 언어에 타입 파라미터라는 개념을 도입한 것이다.
>
>제네릭스를 이용하면 다양한 오브젝트 타입을 지원하는 인터페이스나 메소드를 정의할 수 있다.

각 라인의 값을 문자열로 리턴해주도록 변경하기 위해 인터페이스를 수정한다.

```java
public interface LineCallback<T> {
  T doSomethingWithLine(String line, T value);
}
```

템플릿에도 타입 파라미터를 추가한다.

```java
// highlight-next-line
public <T> lineReadTemplate(String filepath, LineCallback<T> callback, T initVal) throws IOException {
  BufferedReader br = null;
  try {
    br = new BufferedReader(new FileReader(filepaht));
    // highlight-next-line
    T res = initVal;
    String line = null;
    while(line = br.readLine() != null) {
      res = callback.doSomethingWithLine(line, res);
    }
  } catch(IOException e) {
  } finally { ... }
}
```

이제 LineCallback 과 lineReadTemplate() 템플릿은 파일의 라인을 처리해서 T 타입의 결과를 만들어내는 범용 템플릿/콜백이 되었다.

이제 문자열을 받아 연결하는 concatenate() 메소드를 만들어본다.

```java
public String concatenate(String filepath) throws IOException {
  LineCallback<String> concatenateCallback = new LineCallback<String>() {
    public String doSomethingWithLine(String line, String value) {
      return value + line;
    }
  };
  return lineReadTemplate(filepath, concatenateCallback, "");
}
```

기존의 calcSum(), calcMultiply() 도 Integer 타입 파라미터를 가진 인터페이스로 정의해주면 된다.

```java
LineCallback<Integer> sumCallback = new LineCallback<Integer>() { ... };
```
