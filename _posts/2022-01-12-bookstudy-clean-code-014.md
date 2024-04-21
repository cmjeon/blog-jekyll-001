---
layout: single
title: 클린코드 스터디 - 014
categories: 
  - bookstudy - cleancode
tags: 
  - cleancode
---

## 14장 점진적인 개선

Args 를 만들었고 사용법을 확인해보자

Args 생성자에 형식 문자열(”l,p#,d*”)과 인수 문자열(args)을 넘겨 인스턴스를 생성한다.

생성한 인스턴스에 인수값을 질의해서 가져올 수 있다.

```java
public static void main(String[] args) {
  try {
    Args arg = new Args("l,p#,d*", args);
    boolean logging = arg.getBoolean('l');
    int port = arg.getInteger('p');
    String directory = arg.getString('d');
    executeApplication(logging, port, directory);
  } catch (ArgsException e) {
    System.out.print("Argument error: %s\n", e.errorMessage());
  }
}
```

매개변수 두 개로 Args 클래스의 클래스의 인스턴스를 만든다. 

첫번째 매개변수는 형식 또는 스키마이다. 

“l,p#,d*” 은 명령행 인수 세 개를 정의한다. 

- 첫 번째 -l은 부울 인수다.
- 두 번째 -p는 정수 인수다.
- 세 번째 -d는 문자열 인수다.

두번째 매개변수는 main으로 넘어온 명령행 인수 배열 자체다.

ArgsException이 발생하지 않는다면 명령행 인수의 구문을 성공적으로 분석했다는 말이다. 

인수 값을 가져오기 위해 getBoolean, getInteger, getString 등의 메서드를 사용한다.

### Args 구현

#### Args.java

```java
package com.objectmentor.utilities.args;

import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*; 
import java.util.*;

public class Args {
  private Map<Character, ArgumentMarshaler> marshalers;
  private Set<Character> argsFound;
  private ListIterator<String> currentArgument;
  
  public Args(String schema, String[] args) throws ArgsException { 
    marshalers = new HashMap<Character, ArgumentMarshaler>(); 
    argsFound = new HashSet<Character>();
    
    parseSchema(schema);
    parseArgumentStrings(Arrays.asList(args)); 
  }
  
  private void parseSchema(String schema) throws ArgsException { 
    for (String element : schema.split(","))
      if (element.length() > 0) 
        parseSchemaElement(element.trim());
  }
  
  private void parseSchemaElement(String element) throws ArgsException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1); validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
      marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*")) 
      marshalers.put(elementId, new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
      marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##")) 
      marshalers.put(elementId, new DoubleArgumentMarshaler());
    else if (elementTail.equals("[*]"))
      marshalers.put(elementId, new StringArrayArgumentMarshaler());
    else
      throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
  }
  
  private void validateSchemaElementId(char elementId) throws ArgsException { 
    if (!Character.isLetter(elementId))
      throw new ArgsException(INVALID_ARGUMENT_NAME, elementId, null); 
  }
  
  private void parseArgumentStrings(List<String> argsList) throws ArgsException {
    for (currentArgument = argsList.listIterator(); currentArgument.hasNext();) {
      String argString = currentArgument.next(); 
      if (argString.startsWith("-")) {
        parseArgumentCharacters(argString.substring(1)); 
      } else {
        currentArgument.previous();
        break; 
      }
    } 
  }
  
  private void parseArgumentCharacters(String argChars) throws ArgsException { 
    for (int i = 0; i < argChars.length(); i++)
      parseArgumentCharacter(argChars.charAt(i)); 
  }
  
  private void parseArgumentCharacter(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null) {
      throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null); 
    } else {
      argsFound.add(argChar); 
      try {
        m.set(currentArgument); 
      } catch (ArgsException e) {
        e.setErrorArgumentId(argChar);
        throw e; 
      }
    } 
  }
  
  public boolean has(char arg) { 
    return argsFound.contains(arg);
  }
  
  public int nextArgument() {
    return currentArgument.nextIndex();
  }
  
  public boolean getBoolean(char arg) {
    return BooleanArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public String getString(char arg) {
    return StringArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public int getInt(char arg) {
    return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public double getDouble(char arg) {
    return DoubleArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public String[] getStringArray(char arg) {
    return StringArrayArgumentMarshaler.getValue(marshalers.get(arg));
  } 
}
```

#### ArgumentMarshaler.java

```java
import java.util.Iterator;
 
public interface ArgumentMarshaler {
  void set(Iterator<String> currentArgument) throws ArgsException;
}
```

#### BooleanArgumentMarshaler.java

```java
public class BooleanArgumentMarshaler implements ArgumentMarshaler { 
  private boolean booleanValue = false;
  
  public void set(Iterator<String> currentArgument) throws ArgsException { 
    booleanValue = true;
  }
  
  public static boolean getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof BooleanArgumentMarshaler)
      return ((BooleanArgumentMarshaler) am).booleanValue; 
    else
      return false; 
  }
}
```

#### StringArgumentMarshaler.java

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class StringArgumentMarshaler implements ArgumentMarshaler { 
  private String stringValue = "";
  
  public void set(Iterator<String> currentArgument) throws ArgsException { 
    try {
      stringValue = currentArgument.next(); 
    } catch (NoSuchElementException e) {
      throw new ArgsException(MISSING_STRING); 
    }
  }
  
  public static String getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof StringArgumentMarshaler)
      return ((StringArgumentMarshaler) am).stringValue; 
    else
      return ""; 
  }
}
```

#### IntegerArgumentMarshaler

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class IntegerArgumentMarshaler implements ArgumentMarshaler { 
  private int intValue = 0;
  
  public void set(Iterator<String> currentArgument) throws ArgsException { 
    String parameter = null;
    try {
      parameter = currentArgument.next();
      intValue = Integer.parseInt(parameter);
    } catch (NoSuchElementException e) {
      throw new ArgsException(MISSING_INTEGER);
    } catch (NumberFormatException e) {
      throw new ArgsException(INVALID_INTEGER, parameter); 
    }
  }
  
  public static int getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof IntegerArgumentMarshaler)
      return ((IntegerArgumentMarshaler) am).intValue; 
    else
    return 0; 
  }
}
```

#### DoubleArgumentMarshaler.java

```java

```

#### StringArrayArgumentMarshaler.java

```java

```

#### ArgsException.java

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class ArgException extends Exception {
	private char errorArgumentId = '\0';
	private String errorParameter = null;
	private ErrorCode errorCode = OK;
	public ArgsException() {
	}
	public ArgsException(String message) {
		super(message);
	}
	public ArgsException(ErrorCode errorCode) {
		this.errorCode = errorCode;
	}
	public ArgsException(ErrorCode errorCode, String errorParameter) {
		this.errorCode = errorCode;
		this.errorParameter = errorParameter;
	}
	public ArgsException(ErrorCode errorCode, char errorArgumentId, String errorParamter) {
		this.errorCode = errorCode;
		this.errorParameter = errorParameter;
		this.errorArgumentId = errorArgumentId;
	}
	public char getErrorArgumentId() {
		return errorArgumentId;	
	}
	public void setErrorArgumentId(char errorArgumentId) {
		this.errorArgumentId = errorArgumentId
	}
	public String getErrorParameter() {
		return errorParameter;
	}
	public void setErrorParameter(String errorParameter) {
		this.errorParameter = errorParameter;	
	}
	public ErrorCode getErrorCode() {
		return errorCode;
	}
	public void setErrorCode(ErrorCode errorCode) {
		this.errorCode = errorCode;
	}
	public ErrorCode getErrorCode() {
		return errorCode;	
	}
	public void setErrorCode(ErrorCode errorCode) {
		this.errorCode = errorCode;
	}
	public String errorMessage() {
		switch(errorCode) {
			case OK:
				return "TILT: Should not get here.";
			case UNEXPECTED_ARGUMENT:
				return String.format("Argument -%c unexpected.", errorArgumentId);
			case MISSING_STRING:
				return String.format("Could not find string parameter for -%c.", errorArgumentId);
			case INVALID_INTEGER:
				return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
			case MISSING_INTEGER:
				return String.format("Could not find integer parameter for -%c.", errorArgumentId);
			case INVALID_DOUBLE:
				return String.format("Argument -%c expects a double but was '%s'.", errorArgumentId, errorParameter);
			case MISSING_DOUBLE:
				return String.format("Could not find double parameter for -%c.", errorArgumentId);
			case INVALID_ARGUMENT_NAME:
				return String.format("'%c' is not a valid argument name.", errorArgumentId);
			case INVALID_ARGUMENT_FORMAT:
				return String.format("'%s' is not a valid argument format.", errorParameter);
		}
		return "";
	}
	public enum ErrorCode {
		OK, 
		INVALID_ARGUMENT_FORMAT, 
		UNEXPECTED_ARGUMENT,
		INVALID_ARGUMENT_NAME,
		MISSING_STRING,
		MISSING_INTEGER,
		INVALID_INTEGER,
		MISSING_DOUBLE,
		INVALID_DOUBLE
	}
}
```

#### 어떻게 짰느냐고?

저자는 처음부터 저렇게 구현하지 않았다. 

프로그래밍은 과학보다 공예(craft)에 가깝다. ****

깨끗한 코드를 짜려면 먼저 지저분한 코들르 짠 뒤에 정리해야 한다는 의미이다.

### Args: 1차 초안

```java
import java.text.ParseException; 
import java.util.*;

public class Args {
  private String schema;
  private String[] args;
  private boolean valid = true;
  private Set<Character> unexpectedArguments = new TreeSet<Character>(); 
  private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
  private Map<Character, String> stringArgs = new HashMap<Character, String>(); 
  private Map<Character, Integer> intArgs = new HashMap<Character, Integer>(); 
  private Set<Character> argsFound = new HashSet<Character>();
  private int currentArgument;
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;
  
  private enum ErrorCode {
    OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT}
    
  public Args(String schema, String[] args) throws ParseException { 
    this.schema = schema;
    this.args = args;
    valid = parse();
  }
  
  private boolean parse() throws ParseException { 
    if (schema.length() == 0 && args.length == 0)
      return true; 
    parseSchema(); 
    try {
      parseArguments();
    } catch (ArgsException e) {
    }
    return valid;
  }
  
  private boolean parseSchema() throws ParseException { 
    for (String element : schema.split(",")) {
      if (element.length() > 0) {
        String trimmedElement = element.trim(); 
        parseSchemaElement(trimmedElement);
      } 
    }
    return true; 
  }
  
  private void parseSchemaElement(String element) throws ParseException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1); 
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail)) 
      parseBooleanSchemaElement(elementId);
    else if (isStringSchemaElement(elementTail)) 
      parseStringSchemaElement(elementId);
    else if (isIntegerSchemaElement(elementTail)) 
      parseIntegerSchemaElement(elementId);
    else
      throw new ParseException(String.format("Argument: %c has invalid format: %s.", 
        elementId, elementTail), 0);
    } 
  }
    
  private void validateSchemaElementId(char elementId) throws ParseException { 
    if (!Character.isLetter(elementId)) {
      throw new ParseException("Bad character:" + elementId + "in Args format: " + schema, 0);
    }
  }
  
  private void parseBooleanSchemaElement(char elementId) { 
    booleanArgs.put(elementId, false);
  }
  
  private void parseIntegerSchemaElement(char elementId) { 
    intArgs.put(elementId, 0);
  }
  
  private void parseStringSchemaElement(char elementId) { 
    stringArgs.put(elementId, "");
  }
  
  private boolean isStringSchemaElement(String elementTail) { 
    return elementTail.equals("*");
  }
  
  private boolean isBooleanSchemaElement(String elementTail) { 
    return elementTail.length() == 0;
  }
  
  private boolean isIntegerSchemaElement(String elementTail) { 
    return elementTail.equals("#");
  }
  
  private boolean parseArguments() throws ArgsException {
    for (currentArgument = 0; currentArgument < args.length; currentArgument++) {
      String arg = args[currentArgument];
      parseArgument(arg); 
    }
    return true; 
  }
  
  private void parseArgument(String arg) throws ArgsException { 
    if (arg.startsWith("-"))
      parseElements(arg); 
  }
  
  private void parseElements(String arg) throws ArgsException { 
    for (int i = 1; i < arg.length(); i++)
      parseElement(arg.charAt(i)); 
  }
  
  private void parseElement(char argChar) throws ArgsException { 
    if (setArgument(argChar))
      argsFound.add(argChar); 
    else 
      unexpectedArguments.add(argChar); 
      errorCode = ErrorCode.UNEXPECTED_ARGUMENT; 
      valid = false;
  }
  
  private boolean setArgument(char argChar) throws ArgsException { 
    if (isBooleanArg(argChar))
      setBooleanArg(argChar, true); 
    else if (isStringArg(argChar))
      setStringArg(argChar); 
    else if (isIntArg(argChar))
      setIntArg(argChar); 
    else
      return false;
    
    return true; 
  }
  
  private boolean isIntArg(char argChar) {
    return intArgs.containsKey(argChar);
  }
  
  private void setIntArg(char argChar) throws ArgsException { 
    currentArgument++;
    String parameter = null;
    try {
      parameter = args[currentArgument];
      intArgs.put(argChar, new Integer(parameter)); 
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgumentId = argChar;
      errorCode = ErrorCode.MISSING_INTEGER;
      throw new ArgsException();
    } catch (NumberFormatException e) {
      valid = false;
      errorArgumentId = argChar; 
      errorParameter = parameter;
      errorCode = ErrorCode.INVALID_INTEGER; 
      throw new ArgsException();
    } 
  }
  
  private void setStringArg(char argChar) throws ArgsException { 
    currentArgument++;
    try {
      stringArgs.put(argChar, args[currentArgument]); 
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgumentId = argChar;
      errorCode = ErrorCode.MISSING_STRING; 
      throw new ArgsException();
    } 
  }
  
  private boolean isStringArg(char argChar) { 
    return stringArgs.containsKey(argChar);
  }
  
  private void setBooleanArg(char argChar, boolean value) { 
    booleanArgs.put(argChar, value);
  }
  
  private boolean isBooleanArg(char argChar) { 
    return booleanArgs.containsKey(argChar);
  }
  
  public int cardinality() { 
    return argsFound.size();
  }
  
  public String usage() { 
    if (schema.length() > 0)
      return "-[" + schema + "]"; 
    else
      return ""; 
  }
  
  public String errorMessage() throws Exception { 
    switch (errorCode) {
      case OK:
        throw new Exception("TILT: Should not get here.");
      case UNEXPECTED_ARGUMENT:
        return unexpectedArgumentMessage();
      case MISSING_STRING:
        return String.format("Could not find string parameter for -%c.", errorArgumentId);
      case INVALID_INTEGER:
        return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return String.format("Could not find integer parameter for -%c.", errorArgumentId);
    }
    return ""; 
  }
  
  private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s) -"); 
    for (char c : unexpectedArguments) {
      message.append(c); 
    }
    message.append(" unexpected.");
    
    return message.toString(); 
  }
  
  private boolean falseIfNull(Boolean b) { 
    return b != null && b;
  }
  
  private int zeroIfNull(Integer i) { 
    return i == null ? 0 : i;
  }
  
  private String blankIfNull(String s) { 
    return s == null ? "" : s;
  }
  
  public String getString(char arg) { 
    return blankIfNull(stringArgs.get(arg));
  }
  
  public int getInt(char arg) {
    return zeroIfNull(intArgs.get(arg));
  }
  
  public boolean getBoolean(char arg) { 
    return falseIfNull(booleanArgs.get(arg));
  }
  
  public boolean has(char arg) { 
    return argsFound.contains(arg);
  }
  
  public boolean isValid() { 
    return valid;
  }
  
  private class ArgsException extends Exception {
  } 
}
```

코드는 돌아가지만 엉망이다. 

첫 버전에서는 boolean 인수만 지원하다가 String/Integer 인수 유형을 추가하면서 부터 재앙이 시작되어 리팩터링을 시작했다.

#### 점진적으로 개선하다

프로그램을 망치는 가장 좋은 방법 중 하나는 개선이라는 이름 아래 구조를 크게 뒤집는 행위다. 

어떤 프로그램은 그저 그런 '개선'에서 결코 회복하지 못한다. 

'개선' 전과 똑같이 프로그램을 돌리기가 아주 어렵기  때문이다.

이럴 때, TDD 기법을 사용한다. TDD는 언제 어느 때라도 시스템이 돌아가야 한다는 원칙을 따름.

변경을 가한 후에도 시스템이 변경 전과 똑같이 돌아가야 하기 때문이다.

가장 먼저 기존 코드 끝에 ArgumentMarshaler 클래스의 골격을 추가했다. 우선 Boolean 부터..

##### ArgumentMarshaler 추가

```java
//1차초안 Args.java 끝에 추가함.
private class ArgumentMarshaler { 
  private boolean booleanValue = false;

  public void setBoolean(boolean value) { 
    booleanValue = value;
  }
  
  public boolean getBoolean() {
		return booleanValue;
	} 

	public void setString(String s) { 
	    stringValue = s;
  }
  
  public String getString() {
		return stringValue;
	} 

  public void setInteger(int i) { 
	    integerValue = i;
  }
  
  public int getInteger() {
		return integerValue;
	}

}

//아직 구현부는 없음.
private class BooleanArgumentMarshaler extends ArgumentMarshaler { }
private class StringArgumentMarshaler extends ArgumentMarshaler { }
private class IntegerArgumentMarshaler extends ArgumentMarshaler { }
```

```java
private Map<Character, ArgumentMarshaler> booleanArgs = new HashMap<Character, ArgumentMarshaler>();
private Map<Character, ArgumentMarshaler> stringArgs = new HashMap<Character, ArgumentMarshaler>();
private Map<Character, ArgumentMarshaler> integerArgs = new HashMap<Character, ArgumentMarshaler>();
```

```java
...
//스키마 Element를 Map에 추가.
private void parseBooleanSchemaElement(char elementId) {
  booleanArgs.put(elementId, new BooleanArgumentMarshaler());
}
  
...
//argChar를 키값으로 하는 value (ArgumentMarshaler타입) 을 가져와 setBoolean으로 값 세팅
private void setBooleanArg(char argChar, boolean value) {
  booleanArgs.get(argChar).setBoolean(value);
}
  
...
//setBooleanArg으로 set된 데이터 가져옴.
public boolean getBoolean(char arg) {
  Args.ArgumentMarshaler am = booleanArgs.get(arg);
  return am != null && am.getBoolean();
}
```

위처럼 String과 Integer 도 추가한다. 

##### 파생클래스 구현

```java
 private abstract class ArgumentMarshaler { 
		//5. 제거
-   protected boolean booleanValue = false;
		private String stringValue;
		private int integerValue;

	  //3. 제거
-   public void setBoolean(boolean value) {
-     booleanValue = value;
-	  }
  
-	  public boolean getBoolean() { return booleanValue; }

	  public void setString(String s) { 
      stringValue = s;
	  }
  
	  public String getString() {
			return stringValue;
		} 

	  public void setInteger(int i) { 
      integerValue = i;
	  }
  
	  public int getInteger() {
			return integerValue;
		}
 
		//1. 추가
	  public abstract void set(String s);
		public abstract Object get();
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
		//6.변수 추가
		private boolean booleanValue = false;

		//2.구현
		// 참고로 set의 인수는 BooleanArgumentMarshaler에선 사용하지 않는다.
		// StringArgumentMarshaler와 IntegerArgumentMarshaler에서 필요하기 때문에 정의함.
		public void set(String s) {
				booleanValue = true; 
		}

		public Object get() {
				return booleanValue;
		}
}

//BooBooleanArgumentMarshaler와 같은 식으로 구현.
private class StringArgumentMarshaler extends ArgumentMarshaler { }
private class IntegerArgumentMarshaler extends ArgumentMarshaler { }
```

```java
private void parseBooleanSchemaElement(char elementId) {
  booleanArgs.put(elementId, new BooleanArgumentMarshaler());
}
  
...
private void setBooleanArg(char argChar, boolean value) {
- booleanArgs.get(argChar).setBoolean(value);
  booleanArgs.get(argChar).set("true")
}
  
...
public boolean getBoolean(char arg) {
- Args.ArgumentMarshaler am = booleanArgs.get(arg);
- return am != null && am.getBoolean();

	//get은 Object형식을 반환하므로 형변환 필요.
	Args.ArgumentMarshaler am = booleanArgs.get(arg);
  return am != null && (Boolean)am.get();
}
```

##### marshalers 통합 시도

```java
//8. 제거
- private Map<Character, ArgumentMarshaler> booleanArgs = new HashMap<Character, ArgumentMarshaler>();
private Map<Character, ArgumentMarshaler> stringArgs = new HashMap<Character, ArgumentMarshaler>();
private Map<Character, ArgumentMarshaler> integerArgs = new HashMap<Character, ArgumentMarshaler>();
// 1. 추가
+ private Map<Character, ArgumentMarshaler> marshalers = new HashMap<Character, ArgumentMarshaler>();
```

```java
private boolean setArgument(char argChar) throws ArgsException { 
  if (isBooleanArg(argChar))
    setBooleanArg(argChar, true); 
  else if (isStringArg(argChar))
    setStringArg(argChar); 
  else if (isIntArg(argChar))
    setIntArg(argChar); 
  else
    return false;
  
  return true; 
}

private void parseBooleanSchemaElement(char elementId) {
- booleanArgs.put(elementId, new BooleanArgumentMarshaler());
  //2. 변경
  ArgumentMarshaler m = new BolleanArgumentMarshaler();
  marshalers.put(elementId, m);
}
  
...
private void setBooleanArg(char argChar) {
  booleanArgs.get(argChar).set("true") 
}
  
...
public boolean getBoolean(char arg) {
  Args.ArgumentMarshaler am = booleanArgs.get(arg);
  return am != null && (Boolean)am.get();
}

//3. 1차 변경
private boolean isBooleanArg(char argChar) { 
  //return booleanArgs.containsKey(argChar);
  ArgumentMarshaler m = marshalers.get(argChar);
  return m instanceof BooleanArgumentMarshlaer;
}

Integer / String 도 동일방식
```

##### setArgument 변경

```java
//4. setArgument 1차변경
private boolean setArgument(char argChar) throws ArgsException { 
  //1. 추가 및 변경
+ ArgumentMarshaler m = marshalers.get(argChar);
  if (isBooleanArg(**m**))
    setBooleanArg(argChar); 
  else if (isStringArg(**m**))
    setStringArg(argChar); 
  else if (isIntArg(**m**))
    setIntArg(argChar); 
  else
    return false;
  
  return true; 
}

private boolean isBooleanArg(**ArgumentMarshaler m**) { 
- ArgumentMarshaler m = marshalers.get(argChar);
  return m instanceof BooleanArgumentMarshlaer;**
}

//5. setArgument 2차변경
private boolean setArgument(char argChar) throws ArgsException { 
  ArgumentMarshaler m = marshalers.get(argChar);
  if (m instanceof BooleanArgumentMarshlaer)
    setBooleanArg(argChar); 
  else if (m instanceof StringArgumentMarshaler))
    setStringArg(argChar); 
  else if (m instanceof IntergerArgumentMarshaler)
    setIntArg(argChar); 
  else
    return false;
  
  return true; 
}
```

#### 유형별 HashMap을 marshalers HashMap으로 교체

```java

//6. 예외 관리 코드 추가
private boolean setArgument(char argChar) throws ArgsException { 
	
	ArgumentMarshaler m = marshalers.get(argChar);
	try {
    if (**m instanceof BooleanArgumentMarshlaer**)
      setBooleanArg(argChar); 
    else if (**m instanceof StringArgumentMarshaler**))
      setStringArg(argChar); 
    else if (**m instanceof IntergerArgumentMarshaler**)
      setIntArg(argChar); 
    else
      return false;
	} catch (ArgsException e) {
		valid = false;
		errorArgumentId = argChar;
		throw e;
	}
  return true; 
}

//7. setBooleanArg 인수 변경
private void setBooleanArg(**ArgumentMarshaler m**) {
- booleanArgs.get(argChar).set("true")
	m.set("true")
}

//8. 예외처리 추가
public boolean getBoolean(char arg) {
  Args.ArgumentMarshaler am = booleanArgs.get(arg);
- return am != null && (Boolean)am.get();
  boolean b = false;
  try {
    b = am != null && (Boolean)am.get();
  } catch {
    b = false;
  }
  return am != null && (Boolean)am.get();
}
```

동일한 방식으로 stringArgs / IntegerArgs→ marshaler로 통합 —> 첫번째 리펙토링 끝

위와 같이 저자는 marshaller로 한 번에 통합하지 않고, 추가 먼저 진행 후 완전히 booleanArgs가 사용되지 않을 때까지 테스트를 통과하게 된 후 마지막에 제거했다. 

위와 같은 식으로 한 번에 구조를 변경하는 것이 아니라 점진적인 개선을 할 수 있다. 

### 결론

적절한 장소를 만들어 코드만 분리해도 설계가 좋아진다. 관심사를 분리하면 코드를 이해하고 보수하기 훨씬 더 쉬워진다.

그리고 그저 돌아가는 코드만으로는 부족하다.

돌아가는 코드가 심하게 망가지는 사례는 흔하다. 단순히 돌아가는 코드에 만족하는 프로그래머는 전문가 정신이 부족하다.

나쁜 코드보다 더 오랫동안 더 심각하게 개발 프로젝트에 악영향을 미치는 요인도 없다. 나쁜 일정은 다시 짜면 된다.

그러므로 코드는 언제나 최대한 깔끔하고 단순하게 정리하자. 절대로 썩어가게 방치하면 안된다.

## 참고
- 
