---
layout: single
title: 리팩터링2 스터디 - 009 - 1
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 9장 데이터 조직화

## 9.1 변수쪼개기 Split Variable

```jsx
// ASIS
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);

// TOBE
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

### 배경

변수에 값을 여러 번 대입한다면 변수가 여러가지 역할을 한다는 신호이다.

역할이 둘 이상인 변수가 있다면 쪼개야 한다.

역할 하나당 변수 하나다.

여러 용도로 쓰인 변수는 코드를 읽는 이에게 커다란 혼란을 준다.

### 절차

1. 변수를 선언한 곳과 값을 처음 대입하는 곳에서 변수 이름을 새롭게 바꾼다.
  1. i = i + a 같은 수집변수와는 다르니 주의한다.
2. 가능하면 이때 불변으로 선언한다.
3. 이 변수에 두 번째로 값을 대입하는 곳 앞까지의 모든 참조를 새로운 변수 이름으로 바꾼다.
4. 두 번째 대입 시 변수를 원래 이름으로 다시 선언한다.
5. 테스트한다.
6. 반복한다. 매 반복에서 변수를 새로운 이름으로 선언하고 다음번 대입 때까지의 모든 참조를 새 변수명으로 바꾼다. 이 과정을 마지막 대입까지 반복한다.

### 예시

```jsx
// ASIS
function distanceTravelled(scenario, time) {
    let result;
    let acc = scenario.primaryForce / scenario.mass; // 가속도(a) = 힘(F) / 질량(m)
    let primaryTime = Math.min(time, scenario.delay);
    result = 0.5 * acc * primarytime * primaryTime; // 전파된 거리
    let secondaryTime = time - scenario.delay;
    if(secondaryTime > 0) { // 두 번째 힘을 반영해 다시 계산
        let primaryVelocity = acc * scenario.delay;
        acc = (scenario.primaryForce + scenario.secondaryTime) / scenario.mass;
        result += primaryVelocity + secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
    }
    return result;
}
```

acc 에 값이 두 번 대입되는 점을 확인한다.

첫번째 acc 는 초기 가속도를 저장하는 역할이고, 두번째는 두번째 힘까지 반영된 후의 가속도를 저장하는 역할이다.

> 함수나 파일에서 특정 심벌이 쓰인 위치를 시각적으로 강조해주는 코드 편집기를 사용하면 변수의 쓰임을 분석하는 데 도움이 된다. 물론 요즘 편집기들은 대부분 지원한다.

1. 변수에 새로운 이름을 지어주고 불변으로 만든다. 그리고 두 번째 대입 전까지 모든 참조를 새로운 이름으로 바꾼다. 그리고 두 번째로 대입할 때 변수를 다시 선언한다.

```jsx
function distanceTravelled(scenario, time) {
    let result;
    const primaryAcceleration = scenario.primaryForce / scenario.mass; // here
    let primaryTime = Math.min(time, scenario.delay);
    result = 0.5 * primaryAcceleration * primarytime * primaryTime; // here
    let secondaryTime = time - scenario.delay;
    if(secondaryTime > 0) {
        let primaryVelocity = primaryAcceleration * scenario.delay; // here
        let acc = (scenario.primaryForce + scenario.secondaryTime) / scenario.mass; // here
        result += primaryVelocity + secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
    }
    return result;
}
```

1. 두 번째 대입 역시 변수명을 지어주고  불변으로 만든다.

```jsx
function distanceTravelled(scenario, time) {
    let result;
    const primaryAcceleration = scenario.primaryForce / scenario.mass;
    let primaryTime = Math.min(time, scenario.delay);
    result = 0.5 * primaryAcceleration * primarytime * primaryTime;
    let secondaryTime = time - scenario.delay;
    if(secondaryTime > 0) {
        let primaryVelocity = primaryAcceleration * scenario.delay;
        const secondaryAccleration = (scenario.primaryForce + scenario.secondaryTime) / scenario.mass; // here
        result += primaryVelocity + secondaryTime + 0.5 * secondaryAccleration * secondaryTime * secondaryTime; // here
    }
    return result;
}
```

### 예시: 입력 매개변수의 값을 수정할 때

```jsx
// ASIS
function discount(inputValue, quantity) {
    if(inputValue > 50) inputValue = inputValue - 2;
    if(quantity > 100) inputValue = inputValue - 1;
    return inputValue;
}
```

inputValue 는 함수에 데이터를 전달하는 용도와 결과를 반환하는 용도로 쓰이고 있다.

1. inputValue 를 쪼갠다.

```jsx
function discount(originalInputValue, quantity) { // here
    let inputValue = originalInputValue; // here
    if(inputValue > 50) inputValue = inputValue - 2;
    if(quantity > 100) inputValue = inputValue - 1;
    return inputValue;
}
```

1. 변수에 맞는 이름으로 변경한다.

```jsx
function discount(inputValue, quantity) { // here
    let result = inputValue; // here
    if(inputValue > 50) result = inputValue - 2; // here
    if(quantity > 100) result = inputValue - 1; // here
    return result; // here
}
```

## 9.2 필드 이름 바꾸기 Rename Field

### 배경

데이터 구조는 프로그램을 이해하는 데 큰 역할을 한다.

데이터 구조가 중요한만큼 반드시 깔끔하게 관리되어야 한다.

이 과정에서 레코드의 필드명을 바꾸고 싶을 때가 있다.

### 절차

1. 레코드의 유효 범위가 제한적이라면 필드에 접근하는 모든 코드를 수정한 후 테스트한다. 이후 단계는 필요 없다.
2. 레코드가 캡슐화되지 않았다면 우선 레코드를 캡슐화 한다.
3. 캡슈로하된 객체 안의 private 필드명을 변경하고, 그에 맞게 내부 메서드들을 수정한다.
4. 테스트한다.
5. 생성자의 매개변수 중 필드와 이름이 겹치는 게 있다면 함수 선언 바꾸기로 변경한다.
6. 접근자들의 이름도 바꿔준다.

### 예시

```jsx
const organizatoin = {
    name = "애크미 구스베리",
    country: "GB"
}
```

여기서 name 을 title 로 변경해본다.

1. 먼저 organization 레코드를 캡슐화 한다.

```jsx
class Organization {
    constructor(data) {
        this._name = data.name;
        this._country = data.country;
    }
    get name() {
        return this._name;
    }
    set name(aString) {
        this._name = aString;
    }
    get country() {
        return this._country;
    }
    set country(aCountryCode) {
        this._country = aCountryCode;
    }
}
const organization = new Organization({
	  name:"애크미 구스베리", 
	  country: "GB"
})
```

레코드를 클래스로 캡슐화해서 이름을 변경할 곳이 네 곳이 되었다.

모든 변경을 한 번에 수행하는 대신 작은 단계들로 나눠 독립적으로 수행할 수 있게 되었다.

‘작은 단계'라 함은 각 단계에서 하는 일이 작아졌고 따라서 잘못될 일이 적어졌다는 뜻이다.

1. title 필드를 정의하고 생성자와 접근자에서 둘을 구분해 사용하도록 한다.

```jsx
class Organization {
    constructor(data) {
        this._title = data.name; // here
        this._country = data.country;
    }
    get name() {
        return this._title; // here
    }
    set name(aString) {
        this._title = aString; // here
    }
    ...
}
```

1. 생성자에서 title 을 받아들일 수 있도록 조치한다. title 이 들어오면 _title 에 title 을 대입해주고, title 이 없으면 _title 에 name 을 넣어준다.

```jsx
class Organization {
    constructor(data) {
        this._title = (data.title !== undefined) ? data.title : data.name; // here
        this._country = data.country;
    }
		...
}
```

1. 생성자를 호출하는 곳을 모두 찾아서 새로운 이름인 title 로 변경한다

```jsx
const organizaton = new Organization({
		title: "애크미 구스베리", // here
		country: "GB"
}
```

1. 모두 수정했다면 생성자에서 name 을 사용하던 코드를 제거한다.

```jsx
class Organization {
    constructor(data) {
        this._title = data.title // here
        this._country = data.country;
    }
		get title() {
        return this._title;
    }
    set title(aString) {
        this._title = aString;
    }
		...
}
```

가변 데이터 구조를 이용한다면 데이터를 복제하는 행위가 재앙으로 이어질 수 있다.

불편 데이터 구조가 널리 쓰이게 된 이유는 바로 이 재앙을 막기 위해서다.

## 9.3 파생 변수를 질의 함수로 바꾸기 Replace Derived Variable with Query

```jsx
// ASIS
get discountedTotal() {
	return this._discountedTotal
}
set discount(aNumber) {
    const old = this._discount;
    this._discount = aNumber;
    this._discountedTotal += old - aNumber;
}

// TOBE
get discountedTotal() {
    return this._baseTotal - this._discount;
}
set _discount(aNumber) {
    this._discount = aNumber;
}
```

### 배경

가변 데이터는 소프트웨어에 문제를 일으키는 가장 큰 골칫거리에 속한다.

가변 데이터의 유효 범위를 가능한 한 좁혀야 한다고 힘주어 주장해본다.

효과가 좋은 방법은 값을 쉽게 계산해낼 수 있는 변수를 모두 제거하는 것이다.

피연산자 데이터가 불변이라면 계산 과정도 일정하므로 역시 불변으로 만들 수 있다.

### 절차

1. 변수 값이 갱신되는 지점을 모두 찾는다. 필요하면 변수 쪼개기로 변수를 분리한다.
2. 해당 변수의 값을 계산해주는 함수를 만든다.
3. 해당 변수가 사용되는 모든 곳에 어서션을 추가하여 함수의 계산 결과가 변수의 값과 같은지 확인한다.
4. 테스트한다.
5. 변수를 읽는 코드를 모두 함수 호출로 대체한다.
6. 테스트한다.
7. 변수를 선언하고 갱신하는 코드를 죽은 코드 제거하기로 없앤다.

### 예시

```jsx
// ASIS
get production() {
    return this._production;
}
applyAdjustment(anAdjustment) {
    this._adjustments.push(anAdjustment);
    this._production += adAdjustment.amount;
}
```

1. 어서션을 추가한다.

```jsx
get production() {
    assert(this._production === this.calculatedProduction); // here
    return this._production;
}

get calculatedProduction() {
    return this._adjustments.reduce((sum, a) => sum+ a.amount, 0);
}
```

1. 어서션이 실패하지 않으면 코드를 수정하여 계산 결과를 직접 반환하도록 한다.

```jsx
get production() {
    return this.calculatedProduction; // here
}
```

1. calculatedProduction() 메서드를 인라인한다.

```jsx
get production() {
	this._adjustments.reduce((sum, a) => sum+ a.amount, 0);
}
```

1. 옛 변수를 참조하는 코드를 정리한다.

### 예시: 소스가 둘 이상일 때

```jsx
// ProductionPlan 클래스
constructor(production) {
	this._production = production;
	this._adjustments = [];
}

get production() {
	return this._production;
}

applyAdjustment(anAdjustment) {
    this._adjustments.push(anAdjustment);
    this._production += anAdjustment;
}
```

1. 변수 쪼개기를 적용한다.

```jsx
// ProductionPlan 클래스
constructor(production) {
  this._initialProduction = production; // here
	this._productionAccumulator = 0; // here
	this._adjustments = [];
}

get production() {
	return this._initialProduction + this._productionAccumulator; // here
}
```

1. 어서션을 추가한다.

```jsx
get production() {
  assert(this._productionAccumulator === this.calculatedProductionAccmulator); // here
	return this._initialProduction + this._productionAccumulator;
}

get CalculatedProductionAccmulator() { // here
	return this._adjustments.reduce((sum, a) => sum + a.amount, 0);
}
```

## 9.4 참조를 값으로 바꾸기 Change Reference to Value

```jsx
// ASIS
class Product {
	applyDiscount(arg) {
		this._price.amount -= arg;
	}

// TOBE
class Product {
	applyDiscount(arg) {
		this._price = new Money(this._price.amount - arg, this._price.currency);
	}
	...
}
```

### 배경

객체를 다른 객체에 중첩하면 내부 객체는 참조나 값으로 취급할 수 있다.

참조로 다루는 경우에는 내부 객체는 그대로 둔 채 그 객체의 속성만 갱신하며, 값으로 다루는 경우에는 새로운 속성을 담은 객체로 기존 내부 객체를 통째로 대체한다.

필드를 값으로 다루면 내부 객체의 클래스를 수정하여 값 객체로 만들 수 있다.

값 객체는 불변이기 때문에 자유롭게 활용하기 좋다.

그래서 값 객체는 분산 시스템과 동시성 시스템에서 특히 유용하다.

### 절차

1. 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인한다.
2. 각각의 세터를 하나씩 제거한다.
3. 이 값 객체의 필드들을 사용하는 동치성 비교 메서드를 만든다.

### 예시

사람 객체가 있고, 이 객체는 다음 코드처럼 생성 시점에는 전화번호가 올바로 설정되지 못하게 짜여있다.

```jsx
// ASIS
// Person 클래스
constructor() {
    this._telephoneNumber = new _telephoneNumber();
}

get officeAreaCode() {
    return this._telephoneNumber.areaCode;
}
set officeAreaCode(arg) {
    this._telephoneNumber.areaCode = arg;
}
get officeNumber() {
    return this._telephoneNumber.number;
}
set officeNumber(arg) {
    this._telephoneNumber.number = arg;
}

// TelephoneNumber 클래스
get areaCode() {
    return this._areaCode;
}
set areaCode(arg) {
    this._areaCode = arg;
}
get number() {
    return this._telephoneNumber;
}
set number)arg_ {
    this._number = arg;
}
```

클래스를 추출하다 보면 종종 발생하는 상황이다.

TelephoneNumber 객체를 갱신하는 메서드는 여전히 추출 전 클래스에 의존한다.

1. 전화번호 변수를 불변으로 만들고, 생성자를 통해 두 필드를 입력받아 설정한다. 그리고 필드들의 세터들을 제거한다.

```jsx
// TelephoneNumber 클래스
constructor(areaCode, number) {
	this._areaCode = areaCode;
	this._number =number;
}
get areaCode() {
  return this._areaCode;
}
get number() {
  return this._telephoneNumber;
}
```

1. 세터를 호출하는 쪽에서 전화번호를 다시 대입하도록 바꾼다.

```jsx
// Person 클래스
get officeAreaCode() {
  return this._telephoneNumber.areaCode;
}
set officeAreaCode(arg) {
  this._telephoneNumber = new Telephonenumber(arg, this.officeNumber); // here
}
get officeNumber() {
  return this._telephoneNumber.number;
}
set officeNumber(arg) {
	this._telephoennumber = new TelephoneNumber(this.officeAreaCode, arg); // here
}
```

1. 전화번호는 불변이 되었으니 진짜 ‘값’이 될 준비가 끝났다. 값 객체로 인정받으려면 동치성을 값 기반으로 평가해야 한다. javascript 에서는 equals 메서드를 직접 작성하는 것이다.

```jsx
// TelephoneNumber 클래스
equals(other) {
    if(!(other instanceof TelephoneNumber)) return false;
    return this._areaCode === other.areaCode && this.number === other.number;
}
```

1. 테스트 한다.

```jsx
// TelephoneNumber 클래스
equals(other) {
    if(!(other instanceof TelephoneNumber)) return false;
    return this._areaCode === other.areaCode && this.number === other.number;
}

if('telephone equals', function() {
    assert(new TelephoneNumber("312", "555-0142")
        .equals(new TelephoneNumber("212", "555-0142")));)
});
```

테스트의 핵심은 두개의 독립된 객체를 생성하고 동치성 검사를 수행했다는 점이다.

## 9.5 값을 참조로 바꾸기 Change Value to Reference

```jsx
// ASIS
let customer = new Customer(customerData);

// TOVE
let customer = customerRepository.get(customerData.id);
```

### 배경

하나의 데이터 구조 안에 논리적으로 똑같은 제 3의 데이터 구조를 참조하는 레코드가 여러개 있을 때가 있다.

예컨대 주문 목록에 같은 고객이 요청한 주문이 여러 개 섞여 있을 수 있다.

이 때 고객을 값으로도 참조로도 다룰 수 있다.

값으로 다루면 고객 데이터가 각 주문에 복사되고, 참조로 다룬다면 여러 주문이 단 하나의 데이터 구조를 참조하게 된다.

데이터를 갱신해야 하는 상황이라면 참조를 사용하는 편이 좋다.

값을 참조로 바꾸면 엔티티 하나당 객체도 하나만 존재하게 된다.

이런 하나의 객체를 저장소에 한데 모아놓고 객체가 필요한 곳에서는 객체를 얻어쓰는 방식이 된다.

### 절차

1. 같은 부류에 속하는 객체들을 보관할 저장소를 만든다.
2. 생성자에서 이 부류의 객체들 중 특정 객체를 정확히 찾아내는 방법이 있는지 확인한다.
3. 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾도록 한다. 수정할 때마다 테스트한다.

### 예시

```jsx
// Order 클래스
constuctor(data) {
    this._number = data.number;
    this._customer = new Customer(data.customer); // here
}
get customer() {
    return this._customer;
}

// Customer 클래스
constructor(id) {
    this._id = id;
}
get id() {
    return this._id;
}
```

위 방식으로 생성한 고객 객체는 값이다.

1. 저장소 객체를 만든다.

```jsx
let _repositoryData;

export function initialize() {
    _repositoryData = {};
    _repositoryData.customers = new Map();
}

export function registerCustomer(id) {
    if(!_repositoryData.customers.has(id))
        _repositoryData.customers.set(id, new Customer(id));
    return findCustomer(id);
}

export function findCustomer(id) {
    return _repositoryData.customers.get(id);
}
```

1. 주문 클래스의 생성자에서 고객 객체를 가져온다.

```jsx
// Order 클래스
constuctor(data) {
    this._number = data.number;
    this._customer = registerCutomer(data.customer); // here
}
...
```

## 9.6 매직 리터럴 바꾸기 Replace Magic Literal

```jsx
// ASIS
function potentialEnergy(mass, height) {
	return mass * 9.81 * height;
}

// TOBE
const STANDARD_GRAVITY = 9.81;
function potentialEnergy(mass, height) {
	return mass * STANDARD_GRAVITY * height;
}
```

### 배경

매직 리터럴이란 소스 코드에 등장하는 일반적인 리터럴 값을 말한다.

예컨데 움직임을 계산하는 코드에서는 9.80665 라는 숫자가 보인다.

이런 경우에는 상수를 정의하고 숫자 대신 상수를 사용하도록 하는게 좋다.

상수가 특별한 비교 로직에 주로 쓰이는 경우라면 함수호출로 바꾸는 것도 좋다.

```jsx
// ASIS
aValue === MALE_GENDER

// TOBE
isMale(aValue)
```

### 방법

1. 상수를 선언하고 매직 리터럴을 대입한다.
2. 해당 리터럴이 사용되는 곳을 모두 찾는다.
3. 찾으 곳 각각에서 리터럴이 새 상수와 똑같은 의미로 쓰였는지 확인하여, 같은 의미라면 상수로 대체한다.
4. 테스트한다.

## 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}