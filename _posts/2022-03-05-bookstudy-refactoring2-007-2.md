---
layout: single
title: 리팩터링2 스터디 - 007 - 2
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 7장 캡슐화

## 7.6 클래스 인라인하기 Inline Class

```js
// ASIS
class Person {
  get officeAreaCode() {
    return this._telephoneNumber.areaCode;
  }
  get officeNumber() {
    return this._telephoneNumber.number;
  }
}
class TelephoneNumber {
  get areaCode() {
    return this._areaCode;
  }
  get number() {
    return this._number;
  }
}

// TOBE
class Person {
  get officeAreaCode() {
    return this._officeAreaCode;
  }
  get officeNumber() {
    return this._officeNumber;
  }
}
```

### 배경

클래스 인라인하기는 클래스 추출하기를 거꾸로 돌리는 리팩터링이다.

더 이상 제 역할을 못해서 그대로 두면 안되는 클래스는 인라인하는 것이 좋다.

주로 역할을 옮기는 리팩터링을 하고 나니 특정 클래스에 남는 역할이 없을 때 쓰면 좋다.

두 클래스의 기능을 지금과 다르게 배분하고 싶을 때도 클래스 인라인한다.

일단 인라인한 후 클래스 추출하기 하는게 쉬울 수도 있기 때문이다.

### 절차

1. 소스 클래스의 각 public 메소드에 대응하는 메소드들을 타깃 클래스에 생성한다. 이 메소드들은 단순히 작업을 소스 클래스로 위임해야 한다.
2. 소스 클래스의 메소드를 사용하는 코드를 모드 타깃 클래스의 위임 메소드를 사용하도록 바꾼다. 하나씩 바꿀 때마다 테스트한다.
3. 소스 클래스의 메소드와 필드를 모두 타깃 클래스로 옮긴다. 하나씩 옮길 때마다 테스트한다.
4. 소스 클래스를 삭제하고 조의를 표한다.

### 예시: 간단한 레코드 캡슐화하기

배송 추적 정보를 표현하는 TrackingInformation 클래스

```js
class TrackingInfomation {
  get shippingCompany() {
    return this._shippingCompany;
  }
  set shippingCompany(arg) {
    this._shippingCompany = arg;
  }
  get trackingNumber() {
    return this._trackingNumber;
  }
  set trackingInfomation(arg) {
    this._trackingNumber = arg;
  }
  get display() {
    return `${this.shippingComapny}: ${this.trackingNumber}`
  }
}
```

TrackingInformation 클래스는 Shipment 클래스의 일부처럼 사용됨

```js
// Shipment 클래스
get trackingInfo() {
	return this._trackingInfomation.display;
}
get trackingInfomation() {
	return this._trackingInfomation;
}
set trackingInformation(aTrackingInformation) {
  this._trackingInformation = aTrackingInformation;
}
```

TrackingInformation 을 Shipment 클래스로 인라인한다.

1. TrackingInformation 의 메소드를 호출하는 코드를 찾는다.

```js
// 클라이언트
aShipment.trackingInformation.shippingCompany = request.vendor;
```

2. Shipment 에 위임함수를 만든다.

```js
// Shipment 클래스
set shippingCompany(arg) {
  this._trackingInformation.shippingCompany = arg;
}
```

3. 클라이언트가 이를 호출하도록 수정한다.

```js
// 클라이언트
aShipment.shippingCompany = request.vendor;
```

4. 클라이언트에서 사용하는 TrackingInformation 의 모든 요소를 위처럼 처리하고, 이 후 TrackingInformation 의 모든 요소를 Shipment 로 옮긴다.

5. Display() 메소드를 인라인한다.

```js
// Shipment 클래스
get trackingInfo() {
  return `${this.shippingCompany}: ${this.trackingNumber}`;
}
```

6. shippingCompany 을 인라인한다.

```js
// Shipment 클래스
get shippingCompany() {
  return this._shippingCompany;
}
set shippingCompany(arg) {
  this._shippingCompany = arg;
}
```

7. TrackingInformation 의 모든 요소를 옮겼다면 TrackingInformation 클래스를 삭제한다.

```js
// Shipment 클래스
get trackingInfo() {
  return `${this.shippingCompany}: ${this.trackingNumber}`;
}
get shippingCompany() {
  return this._shippingCompany;
}
set shippingCompany(arg) {
  this._shippingCompany = arg;
}
get trackingNumber() {
  return this._trackingNumber;
}
set trackingNumber(arg) {
  this._trackingNumber = arg;
}
```

## 7.7 위임 숨기기 Hide Delegate

```js
// ASIS
manager = aPerson.department.manager;

// TOBE
manager = aPerson.manager;

class Person {
  get manager() {
    return this.department.manager;
  }
}
```

### 배경

모듈화 설계의 핵심은 캡슐화이다.

캡슐화가 잘 되어 있으면 변경 시에 고려해야할 모듈 수가 적어져서 코드를 변경하기 쉽다.

캡슐화란 필드를 숨기는 것 외에도 더 많은 역할이 있다.

예컨데 서버 객체의 필드가 가리키는 객체의 메서드를 호출하려면 클라이언트는 이 위임 객체를 알아야 한다.

이 위임 객체의 인터페이스가 바뀌면 클라이언트의 코드 수정이 필요하다.

만약 서버 자체에 위임 메소드를 만들어서 위임 객체의 존재를 숨길 수 있다면 위임 객체가 수정되더라도 서버 코드만 고치면 될 것이다.

### 절차

1. 위임 객체의 각 메소드에 해당하는 위임 메소드를 서버 객체에 생성한다.
2. 클라이언트가 위임 객체를 대신 서버 객체를 호출하도록 수정한다. 하나씩 바꿀 때마다 테스트한다.
3. 모두 수정했다면, 서버로부터 위임 객체를 얻는 접근자를 제거한다.
4. 테스트한다.

### 예시

사람과 부서가 정의되어 있다.

```js
// Person 클래스
constructor(name) {
  this._name = name;
}
get name() {
  return this._name;
}
get department() {
  return this._department;
}
set department(arg) {
  this._department = arg;
}

// Department 클래스
get chargeCode() {
  return this._chargeCode;
}
set chargeCode(arg) {
  this._chargeCode = arg;
}
get manager() {
  return this._manager;
}
set manager(arg) {
  this._manager = arg;
}

// 클라이언트
manager = aPerson.department.manager;
manager = aPerson.department.chargeCode;
```

1. Person 클래스에 위임 메소드를 정의한다.

```js
// Person 클래스
get manager() {
  return this._department.manager;
}
```

2. 클라이언트가 Person 클래스의 메소드를 사용하도록 고친다.

```js
// 클라이언트
manager = aPerson.manager;
```

3. 같은 방식으로 chargeCode 도 Person 클래스에 정의하고, 클라이언트도 수정한다.

```js
// Person 클래스
get chargeCode() {
  return this._department.chargeCode;
}

// 클라이언트
manager = aPerson.chargeCode;
```

4. Person 클래스의 department() 접근자를 삭제한다.

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}