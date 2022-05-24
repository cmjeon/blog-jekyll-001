---
layout: single
title: 리팩터링2 스터디 - 011 - 12
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 11장 API 리팩터링

## 11.7 세터 제거하기 Remove Setting Method

```jsx
// ASIS
class Person {
  get name() { ... }
  set name(aString) { ... }
}

// TOBE
class Person {
  get name() { ... }
}
```

### 배경

객체 생성 후에 수정되지 않길 원하는 필드라면 세터 메소드가 삭제되어야 한다.

세터 제거하기 리팩터링이 필요한 상황은 주로 두 가지이다.

첫째로 무조건 접근자 메서드를 통해서만 필드를 다루게 하려고 할 때이다.

세터를 제거해서 객체가 생성된 후에는 필드가 바뀌면 안된다는 뜻을 분명히 한다.

두번째는 클라이언트에서 생성 스크립트를 사용해 객체를 생성할 때다.

세터를 제거해서 생성 스크립트가 완료된 뒤에는 그 객체의 필드가 변하면 안된다는 뜻을 분명히 한다.

### 절차

1. 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다. 그런 다음 생성자 안에서 적절한 세터를 호출한다.
2. 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 때마다 테스트한다.
3. 세터 메서드를 인라인한다. 가능하다면 해당 필드를 불변으로 만든다.
4. 테스트한다.

### 예시

Person 클래스

Person 클래스에서 id 는 변경되면 안된댜.

```jsx
// Person 클래스
get name() {
  return this._name;
}
set name(arg) {
  this._name = arg;
}
get id() {
  return this._id;
}
set id(arg) {
  this._id = arg;
}

// 생성 스크립트
const martin = new Person();
martin.name = "마틴";
martin.id = "1234";
```

1. 생성자에서 id 를 받도록 한다.

```jsx
// Person 클래스
constructor(id) {
  this.id = id;
}
...
```

1. 생성 스크립트가 생성자를 통해 id 를 설정하게끔 수정한다.

```jsx
// 생성 스크립트
const martin = new Person("1234");
martin.name = "마틴";
// martin.id = "1234";
```

1. 생성 스크립트가 있는 모든 곳에서 수행하며, 수정할 때마다 테스트한다. 모두 수정하면 세터 메소드를 인라인한다.

```jsx
// Person 클래스
constructor(id) {
  this._id = id; // here
}
get name() {
  return this._name;
}
set name(arg) {
  this._name = arg;
}
get id() {
  return this._id;
}
// set id(arg) {
//   this._id = arg;
// }
```

## 11.8 생성자를 팩토리 함수로 바꾸기 Replace Constuctor with Factory Function

```jsx
// ASIS
leadEngineer = new Employee(document.leadEngineer, 'E');

// TOBE
leadEngineer = createEngineer(document.leadEngineer);
```

### 배경

생성자는 객체를 초기화하는 특별한 용도의 함수다.

생성자에는 일반 함수에는 없는 제약이 있다.

예컨데 자바에서는 생성자를 정의한 클래스의 인스턴스를 반환해야 한다.

서브클래스의 인스턴스나 프록시를 반환할 수 없다.

생성자의 이름도 고정되어 있다.

팩토리 함수에는 이런 제약이 없다.

### 절차

1. 팩토리 함수를 만든다. 팩토리 함수의 본문에서는 원래의 생성자를 호출한다.
2. 생성자를 호출하던 코드를 팩토리 함수 호출로 바꾼다.
3. 하나씩 수정할 때마다 테스트한다.
4. 생성자의 가시 범위가 최소가 되도록 제한한다.

### 예시

employee 클래스가 있다

```jsx
// Employee 클래스
constructor(name, typeCode) {
  this._name = name;
  this._typeCode = typeCode;
}

get name() { return this._name; }
get type() {
  return Employee.legalTypeCodes[this._typeCode];
}
static get legalTypeCodes() {
  return {
    "E":"Engineer",
    "M":"Manager",
    "S":"Salesperson"
  }
}

// 클라이언트
candidate = new Employee(document.name, document.empType);

const leadEngineer = new Employee(document.leadEngineer, 'E');
```

1. 팩토리 함수를 만든다.

```jsx
// 최상위
function createEmployee(name, typeCode) {
  return new Employee(name, typeCode);
}
```

1. 클라이언트의 생성자를 호출하는 곳을 찾아 수정한다.

```jsx
// 클라이언트
candidate = createEmployee(document.name, document.empType);

const leadEngineer = createEmployee(document.leadEngineer, 'E');
```

아래 코드는 직원 유형이 이름에 녹아있는 팩토리 함수를 만들어 대체할 수 있는 경우이다.

```jsx
// 최상위
function createEngineer(name) {
  return new Employee(name, 'E');
}

// 클라이언트
const leadEngineer = createEngineer(document.leadEngineer);
```

## 11.9 함수를 명령으로 바꾸기 Replace Function with Command

```jsx
// ASIS
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  ...
}

// TOBE
class Scorer {
  constuctor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }
  excute() {
    this._result = 0;
    this._healthLevel = 0;
    ...
  }
}
```

### 배경

함수는 프로그래밍의 기본적인 빌딩 블록 중 하나다.

함수를 함수만을 위한 객체 안으로 캡슐화하면 더 유용해질 수도 있다.

이런 객체를 가리켜 ‘명령 객체' 혹은 명령 command 라고 한다.

명령 객체는 대부분 메소드 하나로 구성된며, 이 메소드를 호출해 실행하는 것이 이 객체의 목적이다.

명령은 평범한 함수 메커니즘보다 훨씬 유연하게 함수를 제어하고 표현할 수 있다.

여기서 말하는 ‘명령’은 디자인 패턴의 ‘명령’ 패턴, ‘명령'-질의 분리 원칙의 ‘명령'들과 같은 뜻이다.

### 절차

1. 대상 함수의 기능을 옮길 빈 클래스를 만든다. 클래스 이름은 함수 이름에 기초해 짓는다.
2. 방금 생성한 빈 클래스로 함수를 옮긴다.
3. 함수의 인수들 각각은 명령의 필드로 만들어 생성자를 통해 설정할지 고민해본다.

### 예시

건강보험 애플리케이션에서 사용하는 점수 계산 함수이다.

```jsx
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  let highMedicalRiskFlag = false;

  if(medicalExam.isSmoker) {
    healthLevel += 10;
    highMedicalRiskFlag = true;
  }

  let certificateionGrade = "regular";
  if(scoringGuide.stateWithLowCertification(candidate.originState)) {
    certificateionGrade = "low";
    result -= 5;
  }
  // 비슷한 코드가 한참 이어짐
  result -= Math.max(healthLevel - 5, 0);
  return result;
}
```

1. 빈 클래스를 만들고 함수를 빈 클래스의 excute() 메소드로 옮긴다.

```jsx
function score(candidate, medicalExam, scoringGuide) {
  return new Scorer().execute(candidate, medicalExam, scoringGuide); // here
}

class Scorer { // here
  execute(candidate, medicalExam, scoringGuide) {
    let result = 0;
    let healthLevel = 0;
    let highMedicalRiskFlag = false;

	  if (medicalExam.isSmoker) {
	    healthLevel += 10;
	    highMedicalRiskFlag = true;
	  }
	
	  let certificationGrade = "regular";
	  if (scoringGuide.stateWithLowCertification(candidate.originState)) {
	    certificationGrade = "low";
	    result -= 5;
	  }
	  // 비슷한 코드가 한참 이어짐
	  result -= Math.max(healthLevel - 5, 0);
	  return result;
	}
}
```

주로 명령이 받는 인수들을 생성자로 옮겨서 execute() 메소드는 매개변수를 받지 않게 하는 편이 낫다.

1. 매개변수 옮기기를 한 번에 하나씩 수행한다

```jsx
function score(candidate, medicalExam, scoringGuide) {
  return new Scorer(candidate).execute(medicalExam, scoringGuide); // here
}

class Scorer {
	constructor(candidate) { // here
	  this._candidate = candidate;
	}
	
	execute(medicalExam, scoringGuide) { // here
	  let result = 0;
	  let healthLevel = 0;
	  let highMedicalRiskFlag = false;
	
	  if (medicalExam.isSmoker) {
	    healthLevel += 10;
	    highMedicalRiskFlag = true;
	  }
	
	  let certificationGrade = "regular";
	  if (scoringGuide.stateWithLowCertification(this._candidate.originState)) { // here
	    certificationGrade = "low";
	    result -= 5;
	  }
	  // 비슷한 코드가 한참 이어짐
	  result -= Math.max(healthLevel - 5, 0);
	  return result;
	}
}
```

1. 다른 매개변수들도 하나씩 옮긴다.

## 더 가다듬기

복잡한 함수를 잘게 나누기를 수행한다

지역 변수를 필드로 바꾼다.

```jsx
constructor(candidate) {
  this._candidate = candidate;
  this._medicalExam = medicalExam;
  this._scoringGuide = scoringGuide;
}

execute() {
  this._result = 0;
  this._healthLevel = 0;
  this._highMedicalRiskFlag = false;

  if (this._medicalExam.isSmoker) {
    this._healthLevel += 10;
    this._highMedicalRiskFlag = true;
  }

  this._certificationGrade = "regular";
  if (this._scoringGuide.stateWithLowCertification(this._candidate.originState)) {
    this._certificationGrade = "low";
    this._result -= 5;
  }
  // 비슷한 코드가 한참 이어짐
  this._result -= Math.max(this._healthLevel - 5, 0);
  return this._result;
}
```

함수가 사용하던 변수나 그 유효범위에 구애받지 않고 리팩터링을 적용할 수 있다.

```jsx
execute() {
  this._result = 0;
  this._healthLevel = 0;
  this._highMedicalRiskFlag = false;

  this.scoreSmoking(); // here
  this._certificationGrade = "regular";

  if (this._scoringGuide.stateWithLowCertification(this._candidate.originState)) {
    this._certificationGrade = "low";
    this._result -= 5;
  }
  // 비슷한 코드가 한참 이어짐
  this._result -= Math.max(this._healthLevel - 5, 0);
  return this._result;
}

scoreSmoking() { // here
  if (this._medicalExam.isSmoker) {
    this._healthLevel += 10;
    this._highMedicalRiskFlag = true;
  }
}
```

## 11.10 명령을 함수로 바꾸기 Replace Command with Function

```jsx
// ASIS
class ChargeCalculator {
  constuctor(customer, usage) {
    this._customer = customer;
    this._usage = usage;
  }
  execute() {
    return this._customer.rate * this._usage;
  }
}

// TOBE
function charge(customer, usage) {
  return customer.rate * usage;
}
```

# 배경

명령 객체는 복잡한 연산을 다룰 수 있는 강력한 메커니즘을 제공한다.

구체적으로는 큰 연산 하나를 여러 개의 작은 메소드로 쪼개고 필드를 이용해 쪼개진 메소드들끼리 정보를 공유할 수 있다.

또한 어떤 메소드를 호출하냐에 따라 다른 효과를 줄 수 있고, 각 단계를 거치며 데이터를 조금씩 완성해갈 수도 있다.

명령의 이런 능력은 공짜가 아니다.

로직이 크게 복잡하지 않다면 명령 객체를 평범한 함수로 바꿔주는게 낫다.

# 절차

1. 명령을 생성하는 코드와 명령의 실행 메소드를 호출하는 코드를 함께 함수로 추출한다.
2. 명령의 실행 함수가 호출하는 보조 메소드들 각각을 인라인한다.
3. 함수 선언 바꾸기를 적용하여 생성자의 매개변수 모두를 명령의 실행 메서드로 옮긴다.
4. 명령의 실행 메소드에서 참조하는 필드들 대신 대응하는 매개변수를 사용하게끔 바꾼다. 하나씩 수정할 때마다 테스트한다.
5. 생성자 호출과 명령의  실행 메소드 호출을 호출자 안으로 인라인한다.
6. 테스트한다.
7. 죽은 코드 제거하기로 명령 클래스를 없앤다.

# 예시

명령 객체에서 시작한다.

```jsx
// 명령 객체
class ChargeCalculator {
  constructor(customer, usage, provider) {
    this._customer = customer;
    this._usage = usage;
    this._provider = provider;
  }
  get baseCharge() {
    return this._customer.baseRate * this._usage;
  }
  get charge() {
    return this.baseCharge + this._provider.connectionCharge;
  }
}

// 클라이언트
monthCharge = new ChargeCalculator(customer, usage, provider).charge;
```

1. 클래스를 생성하고 호출하는 코드를 함수로 추출한다.

```jsx
// 클라이언트
monthCharge = charge(customer, usage, provider); // here

// 최상위
function charge(customer, usage, provider) {
  return new ChargeCalculator(customer, usage, provider).charge;
}
```

1. baseCharge() 메소드같은 값을 반환하는 보조 메소드는 반환할 값을 변수로 추출한다.

```jsx
class ChargeCalculator {
  ...
  get baseCharge() {
    return this._customer.baseRate * this._usage;
  }
  get charge() {
    const baseCharge = this.baseCharge; // here
    return this.baseCharge + this._provider.connectionCharge;
  }
}
```

1. 보조 메소드를 인라인한다.

```jsx
class ChargeCalculator {
  ...
  get charge() {
    const baseCharge = this._customer.baseRate * this._usage; // here
    return this.baseCharge + this._provider.connectionCharge;
  }
}
```

1. 생성자가 받는 모든 매개변수를 charge() 메소드로 옮긴다.

```jsx
class ChargeCalculator {
  constructor(customer, usage, provider) {
    this._customer = customer;
    this._usage = usage;
    this._provider = provider;
  }
  
  charge(customer, usage, provider) { // here
    const baseCharge = this._customer.baseRate * this._usage;
    return this.baseCharge + this._provider.connectionCharge;
  }
}

// 최상위
function charge(customer, usage, provider) {
  return new ChargeCalculator(customer, usage, provider)
      .charge(customer, usage, provider); // here
}
```

1. charge() 메소드의 본문에서 필드 대신 매개변수를 사용한다.

```jsx
class ChargeCalculator {
  constructor(customer, usage, provider) {
    // this._customer = customer;
    // this._usage = usage;
    // this._provider = provider;
  }
  
  charge(customer, usage, provider) {
    const baseCharge = customer.baseRate * usage; // here
    return this.baseCharge + provider.connectionCharge; // here
  }
}
```

1. 최상위 charge() 메소드를 인라인한다.

```jsx
// 최상위
function charge(customer, usage, provider) {
  const baseCharge = customer.baseRate * this._usage;
  return this.baseCharge + this._provider.connectionCharge;
}
```

1. 명령 클래스는 삭제한다.

## 11.11 수정된 값 반환하기 Return Modified Value

```jsx
// ASIS
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
  for (let i = 1; i < points.length; i++) {
    const verticalChange =points[i].elevation - points[i-1].elevation;
    totalAscent += (verticalChange > 0) ? verticalChange : 0;
  }
}

// TOBE
const totalAscent = calculateAscent();

function calculateAscent() {
  let result = 0;
  for (let  = 1; i < points.length; i++) {
    const verticalChange =points[i].elevation - points[i-1].elevation;
    totalAscent += (verticalChange > 0) ? verticalChange : 0;
  }
  return result;
}
```

### 배경

같은 데이터를 읽고 수정하는 코드가 여러 곳이라면 데이터 수정되는 흐름과 코드의 흐름을 일치시키기가 상당히 어렵다.

변수를 갱신하는 함수라면 수정된 값을 반환하여 호출자가 그 값을 담아두도록 하는 편이 좋다.

이 리팩터링은 값 하나를 계산한다는 분명한 목적이 있는 함수들에 효과적이다.

### 절차

1. 함수가 수정된 값을 반환하게 하여 호출자가 그 값을 자신의 변수에 저장하게 한다.
2. 테스트한다.
3. 피호출 함수 안에 반환할 갓을 가리키는 새로운 변수를 선언한다.
4. 테스트한다.
5. 계산이 선언과 동시에 이루어지도록 통합한다. (변수를 선언하고 계산 로직을 즉시 대입)
6. 테스트한다.
7. 피호출 함수의 변수 이름을 새 역할에 어울리도록 바꿔준다.
8. 테스트한다.

### 예시

GPS 위치 목록으로 다양한 계산을 수행하는 코드가 있다.

```jsx
let totalAscent = 0;
let totalTime = 0;
let totalDistance = 0;
calculateAscent();
calculateTime();
calculateDistance();
const pace = totalTime / 60 / totalDistance;
```

고도 상승분 ascent 계산을 리팩터링할 예정이다.

```jsx
function calculateAscent() {
  for (let i = 1; i < points.length; i++) {
    const vericalChange = points[i].elevation - points[i-1].elevation;
    totalAscent += (vericalChange > 0) ? vericalChange : 0;
  }
}
```

calculateAscent() 안에서 totalAscent 가 갱신된다는 사실을 드러내기 위해 코드를 수정한다.

1. totalAscent 값을 반환하고, 호출한 곳에서 변수를 대입하게 고친다.

```jsx
let totalAscent = 0;
let totalTime = 0;
let totalDistance = 0;
totalAscent = calculateAscent(); // here
calculateTime();
calculateDistance();
const pace = totalTime / 60 / totalDistance;

function calculateAscent() {
  for (let i = 1; i < points.length; i++) {
    const vericalChange = points[i].elevation - points[i-1].elevation;
    totalAscent += (vericalChange > 0) ? vericalChange : 0;
  }
  return totalAscent; // here
}
```

1. calculateAscent() 안에 반환할 값을 담을 변수인 totalAscent 를 선언한 수, 일반 명명규칙에 맞게 수정한다.

```jsx
function calculateAscent() {
  let result = 0; // here
  for (let i = 1; i < points.length; i++) {
    const vericalChange = points[i].elevation - points[i-1].elevation;
    result += (vericalChange > 0) ? vericalChange : 0; // here
  }
  return result; // here
}
```

1. 변수 선언과 동시에 계산이 수행되도록 하고, 불변으로 만든다.

```jsx
const totalAscent = calculateAscent(); // here
let totalTime = 0;
let totalDistance = 0;
calculateTime();
calculateDistance();
const pace = totalTime / 60 / totalDistance;
```

최종적으로 다른 변수들도 리팩터링을 수행한다.

```jsx
const totalAscent = calculateAscent(); // here
const totalTime = calculateTime(); // here
const totalDistance = calculateDistance(); // here
const pace = totalTime / 60 / totalDistance;
```

## 11.12 오류 코드를 예외로 바꾸기 Replace Error Code with Exception

```jsx
// ASIS
if (data)
  return new ShippingRules(data);
else
  return -23;

// TOBE
if (data)
  return new ShippingRules(data);
else
  throw new OrderProcessingError(-23);
```

### 배경

예외는 프로그래밍 언어에서 제공하는 독립적인 오류 처리 메커니즘이다.

오류가 발견되면 예외를 던진다.

예외는 적절한 예외 핸들러를 찾을 때까지 콜스택을 타고 위로 전파된다.

예외는 정확히 예상 밖의 동작일 때만 쓰여야 한다.

예외를 던지는 코드를 프로그램 종료 코드로 바꿔도 프로그램이 여전히 정상 동작할지를 따져보는 것이 좋다.

### 절차

1. 콜스택 상위에 해당 예외를 처리할 예외 핸들러를 작성한다.
2. 테스트한다.
3. 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다.
4. 정적 검사를 수행한다.
5. catch 절을 수정하여 직접 처리할 수 있는 예외는 적절히 대처하고 그렇지 않은 예외는 다시 던진다.
6. 테스트한다.
7. 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다.
8. 테스트한다.
9. 모두 수정했다면 그 오류 코드를 콜스택 위로 전달하는 코드를 모두 제거한다.
10. 테스트한다.

### 예시

배송지의 배송 규칙을 알아내는 코드이다

```jsx
function localShippingRules(country) {
  const data = countryData.shippingRules[country];
  if (data) return new shippingRules(data);
  else return -23;
}

// 클라이언트
function calculateShippingCosts(anOrder) {
  ...
  const shippingRules = localShippingRules(anOrder.country);
  if (shippingRules < 0) return shippingRules; // 오류 전파
  ...
}

const status = calculateShippingCosts(orderData);
if (status < 0) errorList.push({order: orderData, errorCode: status});
```

가장 먼저 고려되어야 할 것인 이 오류가 ‘예상된 것이냐'는 것이다.

예상할 수 있는 정상 동작 범주라면 오류 코드를 예외로 바꾸는 리팩터링을 적용한다.

1. 최상위에 예외 핸들러를 갖춘다.

```jsx
let status;
try {
  status = calculateShippingCosts(orderData);
} catch (e) {
  throw e;
}
if (status < 0) errorList.push({order: orderData, errorCode: status});
```

1. 예외를 처리하기 위한 예외 클래스를 생성한다.

```jsx
let status;
try {
  status = calculateShippingCosts(orderData);
} catch (e) {
  if (e instanceof OrderProcessingError) // here
    errorList.push({order: orderData, errorCode: e.code});
  else
    throw e;
}
if (status < 0) errorList.push({order: orderData, errorCode: status});

class OrderProcessingError extends Error {
  constructor(errorCode) {
    super(`주문 처리 오류: ${errorCode}`);
    this.code = errorCode;
  }
  get name() {
    return "OrderProcessingError";
  }
}
```

1. 오류 검출 코드를 수정하여 오류 코드 대신 이 예외를 던지도록 한다.

```jsx
function localShippingRules(country) {
  const data = countryData.localShippingRules[country];
  if (data) return new shippingRules(data);
  else throw new OrderProcessingError(-23);
}
```

1. 오류가 발생하지 않는다면 오류 코드를 전파하는 임시 코드를 제거한다.

```jsx
function calculateShippingCosts(anOrder) {
	...
	const data = countryData.localShippingRules[country];
	// if (status < 0) errorList.push({order: orderData, errorCode: status}); // here
	...
}
```

1. status 변수 역시 제거한다.

```jsx
// let status; // here
try {
  // status = calculateShippingCosts(orderData); // here
} catch (e) {
  if (e instanceof OrderProcessingError)
    errorList.push({order: orderData, errorCode: e.code});
  else
    throw e;
}
```

## 11.13 예외를 사전확인으로 바꾸기 Replace Exception with Precheck

```jsx
// ASIS
double getValueForPeriod (int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}

// TOBE
double getValueForPeriod (int periodNumber) {
  return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```

### 배경

함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 검사하도록 하면 된다.

### 절차

1. 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다. catch 블록의 코드를 조건문의 조건절 중 하나로 옮기고, 남은 try 블록의 코드를 다른 조건절을 옮긴다.
2. catch 블록에 어서션을 추가하고 테스트한다.
3. try 문과 catch 블록을 제거한다.
4. 테스트한다.

### 예시

이번 예시는 자바로 표현한다.

데이터베이스 연결같은 자원들을 관리하는 자원 풀 resource poll 클래스가 있다.

자원이 필요한 코드는 풀에서 하나씩 꺼내 사용한다.

```java
public Resource get() {
  Resource result;
  try {
    result = available.pop();
    allocated.add(result);
  } catch (NoSuchElementException e) {
    result = Resource.create();
    allocated.add(result);
  }
  return result;
}

private Deque<Resource> available;
private List<Resource> allocated;
```

1. 조건을 검사하는 코드를 추가하고, catch 블록의 코드를 조건문의 조건절로 옮기고, 남은 try 블록 코드를 다른 조건절로 옮긴다.

```java
public Resource get() {
  Resource result;
  if (available.isEmpty()) { // here
    result = Resource.create(); // here
    allocated.add(result); // here
  } else { // here
    try {
      result = available.pop();
      allocated.add(result);
    } catch ((NoSuchElementException e) {
    }
  }
	return result;
}
```

1. catch 절에는 어서션을 추가한다.

```java
public Resource get() {
  ...
    try {
      ...
    } catch ((NoSuchElementException e) {
			throw new AssertionError("도달 불가");
    }
  }
	return result;
}
```

1. 테스트가 통과하면 try / catch 를 삭제한다.

```java
public Resource get() {
  Resource result;
  if (available.isEmpty()) { // here
    result = Resource.create();
    allocated.add(result);
  } else {
    result = available.pop();
    allocated.add(result);
  }
  return result;
}
```

## 더 가다듬기

중복되는 코드를 추출하고, 3항 연산자로 바꾼다.

```java
public Resource get() {
  Resource result = available.isEmpty() ? Resource.create() : available.pop(); // here
  allocated.add(result);
  return result;
}
```

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}