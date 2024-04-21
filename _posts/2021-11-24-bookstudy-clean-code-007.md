---
layout: single
title: 클린코드 스터디 - 007
categories: 
  - bookstudy - cleancode
tags: 
  - cleancode
---

## 7장 오류처리

> 뭔가 잘못될 가능성은 늘 존재한다. 뭔가 잘못되면 바로 잡을 책임은 바로 우리 프로그래머에게 있다.

깨끗한 코드와 오류 처리는 확실히 연관성이 있는데, 상당수 코드 기반은 전적으로 오류 처리 코드에 좌우된다. 

코드 기반이 오류 처리만 한다는 의미가 아니라 흩어진 오류 처리 코드 때문에 실제 코드가 하는 일을 파악하기가 거의 불가능한 경우가 대부분이다.

오류 처리는 상당히 중요한데 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워 진다면 깨끗한 코드라고 부를 수 없다!

### 오류 코드보다 예외를 사용하라

예외를 지원하지 않는 언어는 오류를 처리 하는 방법이 제한적이었음. 아래 처럼 오류 플래그를 설정하거나 오류 코드를 반환하는 방법이 전부였음

```java
public class DeviceController {
...
	public void sendShutDown(){
		DeviceHandle handle = getHandle(DEV1);
		// 디바이스 상태를 점검한다.
		if(handle != DeviceHandle.INVALID){
			// 레코드 필드에 디바이스 상태를 저장한다.
			retrieveDeviceRecord(handle);
			// 디바이스가 일시정지 상태가 아니라면 종료한다.
			if(record.getStatus() != DEVICE_SUSPENDED){
				pauseDevice(handle);
				clearDeviceWorkQueue(handle);
				closeDevice(handle);
			} else {
				logger.log("Device suspended. Unable to shut down");
			}
		} else {
			logger.log("Invalid handle for: " + DEV1.toString());
		}
	}
	...
}
```

오류 코드를 반환 하는 것 보다 예외를 던지는 것이 좋다.

```java
public class DeviceController {
...
	public class void sendShutDown(){
		try{
			tryToShutDown();
		} catch (DeviceShutDownError e) {
			logger.log(e);
		}
	}
	
	private void tryToShutDown() throws DeviceShutDownError {
		DeviceHandle handle = getHandle(DEV1);
		DeviceRecord record = retrieveDeviceRecord(handle);

		pauseDevice(handle);
		clearDeviceWorkQueue(handle);
		closeDevice(handle);
	}
	
	private DeviceHandle hetHandle(DeviceID id) {
	...
	throw new DeviceShutDownError("Invalid handle for: " +id.toString());
	...
	}
...
}
```

### Try-Catch-Finally 문부터 작성하라

try 블록은 트랙잭션과 비슷하다 

→ try 블록에서 무슨 일이 생기든 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다.

예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하는 편이 낫다

파일이 없으면 예외를 던지는지 알아보는 단위 테스트이다

```java
@Test(expected = storageException.class)
public void retrieveSectionShouhldThrowOnInvalidFileName(){
	sectionStore.retrieveSection("invalid - file");
}
```

예외 유형을 좁혀보자

```java
public List<RecordedGrip> retrieveSection(String sectionName){
	try{
		FileInputStream stream = new FileInputStream(sectionName);
		stream.close();
	catch (FileNotFoundException e){
		throw new StorageException("retrieval error", e);
	}
	return new ArrayList<RecordedGrip>();
}
```

⇒ 강제로 예외를 일으키는 테스트 케이스를 먼저 작성한 다음 테스트를 통과 하게 끔 작성하면 그 다음 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에 트랜잭션 본질을 유지하기 쉬워 진다.

### 미확인 (unchecked) 예외를 사용하라

(질문 134p 3Line) 코드가 메서드를 사용하는 방식이 메서드 선언과 일치하지 않으면 아예 컴파일도 못했다 

→ 오버라이딩이나 오버로딩을 지원하지 않았다는 뜻일까요?

(질문 134p 5Line) 확인된 예외가 반드시 필요하지 않다는 사실이 분명해 졌다의 논리가 C#,C++, Python, 루비가 확인된 예외를 지원하지 않지만 안정적인 소프트웨어를 구현하기에 무리가 없기 때문이다? 

→ why?

(논의해보자) 134p 4번째 문단, 대규모 시스템에서 ~

### 예외에 의미를 제공하라

예외를 던질 때는 전후 상황을 충분히 덧붙인다 

→ 실패한 코드의 의도를 파악하려면 호출 스택만으론 부족하다.

⇒오류 메시지에 정보를 담아 함께 던진다. (연산 이름, 실패 유형 등을 언급하고 logging 기능을 사용한다면 catch 블록에서 오류를 기록할 수 있도록 충분한 정보를 넘겨준다)

호출자를 고려해 예외 클래스를 정의하라

오류를 분류하는 방법은 수없이 많다.

- 오류가 발생한 위치로 분류가 가능하다

- 오류의 유형으로 분류가 가능하다

예를 들어, device failed, network failed, programming failed ...

⇒ 프로그래머에게서 가장 중요한 관심사는 오류를 잡아내는 방법이 되어야 한다.

호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환하면 된다.

ex)
```java
...
	catch (ProDeviceFailure e) {
		reportError(e);
		logger.log(e.getMessage(), e);
	}
	catch ...
...
```

=>
```java
...
	catch (ProDeviceFailure e) {
		throws new PortDeviceFailure(e);
	}
	catch ...
...
```

### 정상 흐름을 정의하라

비즈니스 로직과 오류 처리가 잘 분리된 고드가 나오면 코드가 깨끗하고 간결한 알고리즘으로 보이기 시작한다.

예외 상황을 만들려고 하지말고 비즈니스 로직으로 처리할 수 있다면 비즈니스 로직으로 처리해라

```java
try { 
	MealExpenses  expenses = expenseReportDAO.getMeals(employee.getID());
	m_total += expeneses.getTotal();
} catch(MealExpensesNotFound e){
	m_total += getMealPerDiem();
} // 예외가 노리를 따라가기 어렵게 만드는 경우
```
-------------------------------------------------------------------------

ExpenseReportDAO 를 고쳐서 언제나 MealExpenses 객체를 반환하게 하게 하되 

예외의 상황에 대한 비즈니스 로직을 DAO 에서 처리

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

⇒ 특수 사례 패턴 (SPECIAL CASE PATTERN) 이라고 부른다. 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식 (예외 상황을 캡슐화로 처리)

### NULL을 반환하지 마라

안좋은 코드

```java
if(item != null){
	ItemRegistry registry = peristentStore.getItemRegisry();
	if(registry != null){
		...
		if(item3 != null){
		...
	}
}
```

→ 누구 한명이라도 null 확인을 빼먹는다면 Application 이 통제 불능 상태가 될 수도 있다.

⇒ peristentStore 가 null 라면... ?

API가 null을 반환한다면 감싸기 메서드를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려한다. 

⇒ 많은 경우가 특수 사례 객체가 손쉬운 해결책이다.

(질문) 다음 1,2 안 중에 어떤 코드가 효율 적일까 ?

```java
// 1안
List<Employee> employees = getEmployees();
if(employees != null){
	for(Employee e : employees){
		totalPay += e.getPay();
	}
}
```

```java
// 2안
List<Employee> employees = getEmployees();
for(Employee e : employees){
	totalPay += e.getPay();
}

public List<Employee> getEmployees(){
	if( .. 직원이 없다면 ..)
		return Collections.emptyList();
}
```

### NULL 을 전달하지 마라

```java
public class MetricsCalculator{
	public double xProjection(Point p1, Point p2){
		return (p2.x - p1.x) * 1.5;
	}
	...
}
```

위의 상황에서 .xProjection(null, new Point(12,13)); 을 누군가가 실수로 던진다면 ? 

⇒ 당연히 NullpointException 발생

```java
// 1안
public class MetricsCalculator{
	public double xProjection(Point p1, Point p2){
		if(p1 == null || p2 == null){
			throw InvalidArgumentException("Invalid argument for MetircesCalculator.xProjection");
		}
		return (p2.x - p1.x) * 1.5;
	}
}
```

NullPointException 은 안나겠지만 InvalidArgumentException에 대한 처리기가 필요함

```java
// 2안
public class MetricsCalculator{
	public double xProjection(Point p1, Point p2){
		assert p1 != null : "p1 should not be null";
		assert p2 != null : "p2 should not be null";
		return (p2.x - p1.x) * 1.6;
	}
}
```

⇒ 대다수 프로그래밍 언어가 호출자가 실수로 넘기는 null을 적절하게 처리할 방법이 없다. 따라서, 애초에 null을 넘기지 못하도록 정책이 합리적이다.

### 결론

깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다.

오류 처리와 비즈니스 로직은 분리 되어야 한다.

⇒ 유지 보수 효율성이 크게 높아 진다.

## 참고
- 
