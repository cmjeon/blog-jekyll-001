---
layout: single
title: 리팩토링2 스터디 - 006 - 2
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 6장 기본적인 리팩터링

## 변수 캡슐화하기 Encapsulate Variable

```js
// ASIS
let defaultOwner = { firstName: "마틴", lastName: "파울러" };

// TOBE
let defaultOwnerData = { firstName: "마틴", lastName: "파울러" };
export function defaultOwner() { return defaultOwnerData; }
export function setDefaultOwner(arg) { defaultOwnerData = arg; }
```

### 배경

데이터는 기존 데이터를 둔 채로 전달하는 방식을 사용하지 못하기 때문에 다루기가 까다롭다.

데이터는 참조하는 모든 부분을 한 번에 바꿔야 코드가 제대로 작동한다.

데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화하는 것이 가장 좋은 방법일 때가 많다.

데이터 재구성이라는 어려운 작업을 함수 재구성이라는 더 단순한 작업을 변환하는 것이다.

데이터 캡슐화의 장점

- 데이터를 변경하고 사용하는 코드를 감시할 수 있는 통로가 되어주기 때문에 데이터 변경 전 검증이나 변경 후 추가 로직을 쉽게 끼어 넣을 수 있다
- 데이터 캡슐화를 통해 데이터에 대한 결합도가 높아지는 일을 막을 수 있다

클래스 안에서 필드를 참조할 때조차 반드시 접근자를 통하게 하는 자가 캡슐화(self-encapsulation)를 주장하는 사람도 있지만, 자가 캡슐화가 필요할 정도로 클래스가 크다면 클래스를 잘게 쪼개야 한다.

불변 데이터는 가변 데이터 보다 캡슐화할 이유가 적다.

### 절차

1. 변수로의 접근과 갱신을 전담하는 캡슐화 함수들을 만든다.
2. 정적 검사를 수행한다.
3. 변수를 직접 참조하던 부분을 모두 적절한 캡슐화 함수 호출로 바꾼다. 하나씩 바꿀 때마다 테스트 한다
4. 변수의 접근 범위를 제한한다
    
    → 변수로의 직접 접근을 막을 수 없을 때도 있다. 그럴 때는 변수 이름을 바꿔서 테스트해보면 해당 변수를 참조하는 곳을 쉽게 찾아낼 수 있다
    
5. 테스트한다
6. 변수 값이 레코드라면 레코드 캡슐활하기를 적용할지 고려해본다

### 예시

```js
let defaultOwner = {firstName: "마틴", lastName: "파울러"};
//참조하는 코드
spaceship.owner = defaultOwner;

//갱신하는 코드
defaultOwner = {firstName: "레베카", lastName: "파슨스"};

//데이터를 읽고 쓰는 함수 정의
function defaultOwner() {return defaultOwnerData;}
function setDefaultOwner(arg) {defaultOwnerData = arg;}

//getter, setter로 호출부 변경
spaceship.owner = defaultOwner();
setDefaultOwner({firstName: "레베카", lastName: "파슨스"});

//가시범위 제한
let defaultOwnerData = {firstName: "마틴", lastName: "파울러"};
export function defaultOwner() {return defaultOwnerData;}
export function setDefaultOwner(arg) {defaultOwnerData = arg;}

//값 캡슐화하기
const owner1 = defaultOwner();
assert("파울러", owner1.lastName, "처음 값 확인");
const owner2 = defaultOwner();
owner2.lastName = "파슨스";
assert.equal("파슨스", owner1.lastName, "owner2를 변경한 후"); //성공할까? 
console.log(defaultOwner())

//게터가 데이터의 복제본을 반환하도록 함수 수정
export function defaultOwner() {return Object.assign({},defaultOwnerData);}

//레코드 캡슐화하기
class Person {
  constructor(data) {
    this._lastName = data.lastName;
    this._firstName = data.firstName;
  }

  get lastName() { return this._lastName; }
  get firstName() { return this._firstName; }
}
```

즉 사용하는 상황에 따라 데이터의 복제본을 만들지, 원본 데이터를 변경할지 결정해야 한다.

단, 복제본 만들기가 성능에 주는 영향을 대체로 미미하지만, 원본을 그대로 사용하면 나중에 디버깅하기가 어렵고 시간도 오래 걸릴 위험이 있다.

복제본 만들기와 클래스로 감싸는 방식은 레코드 구조에서 깊이가 1인 속성들까지만 효과가 있다. 더 깊이 들어가면 복제본과 객체 래핑 단계가 더 늘어나게 된다

### 결론

데이터 캡슐화의 구체적인 대상과 방법은 캡슐화할 데이터를 사용하는 방식과 그 데이터를 어떻게 변경하려는 지에 따라 달라진다. 하지만 분명한 사실은  데이터의 사용 범위가 넓을수록 적절히 캡슐화하는 게 좋다는 것이다.

## 6.7 변수 이름 바꾸기 Rename Variable

### 절차

1. 폭넓게 쓰이는 변수라면 변수 캡슐화하기를 고려한다
2. 이름을 바꿀 변수를 참조하는 곳을 모두 찾아서, 하나씩 변경한다
	1. 다른 코드베이스에서 참조하는 변수는 외부에 공개된 변수이므로 이 리팩토링을 적용할 수 없다
	2. 변수 값이 변하지 않는다면 다른 이름으로 복제본을 만들어서 하나씩 점진적으로 변경한다. 하나씩 바꿀 때마다 테스트한다
3. 테스트한다

### 예시1

```js
// 예시 변수 이름 변경
// ASIS
let hpHd = "untitled";

result += `<h1>${hpHd}</h1>`;

tpHd = obj['articleTitle'];

// TOBE
// Step 1 캡슐화
result += `<h1>${title()}</h1>`;
setTitle(obj['articleTitle']);

function title() { return tpHd; }; 
function setTitle(arg) { tpHd = arg; }

// Step 2 변수이름 변경
let _title = "untitled"

function title() { return _title; }; 
function setTitle(arg) { _title = arg; }
```

변수를 읽기도하고 수정도 하는 경우 

1. 변수 캡슐화 하기
2. 게터, 세터를 만들어서 읽기 쓰기를 함수로 사용하도록 변경한다
3. 변수이름을 변경한다
4. 모든 호출자가 변수에 변경된 변수에 직접 접근하게 하는 방법도 있지만, 이름을 바꾸기 위해 캡슐화부터 해야 할 정도로 널리 사용되는 변수라면 나중을 위해서라도 함수 안에 캡슐화된 채로 두는 편이 좋다

### 예시2

```js
// 예시 상수 이름 변경
// ASIS
const cpyNm = "애크미 구스베리";

// TOBE
const companyName = "애크미 구스베리";
const cpyNm = companyName;
```

1. 원본의 이름을 바꾼다
2. 원본의 기존 이름과 같은 복제본을 만든다
3. 기존 이름을 참조하는 코드들을 새 이름으로 점진적으로 바꾼다

## 6.8 매개변수 객체 만들기 Introduce Parameter Object

### 배경

데이터 뭉치를 데이터 구조로 묶으면 데이터 사이의 관계가 명확해진다는 이점이 있다.

함수가 데이터 구조를 받으면 매개변수 수가 줄어든다.

같은 데이터 구조를 사용하는 모든 함수가 원소를 참조할 때 항상 똑같은 이름을 사용하기 때문에 일관성도 높여준다.

### 절차

1. 적당한 데이터 구조가 아직 마련되어 있지 않다면 새로 만든다
    1. 클래스로 만들면 동작까지 함께 묶기 좋아진다
    2. 데이터 구조를 값 객체(Value Object)로 만든다
2. 테스트한다
3. 함수 선언 바꾸기로 새 데이터 구조를 매개변수로 추가한다
4. 테스트한다
5. 함수 호출 시 새로운 데이터 구조 인스턴스를 넘기도록 수정한다. 하나씩 수정할 때마다 테스트한다
6. 기존 매개변수를 사용하던 코드를 새 데이터 구조의 원소를 사용하도록 바꾼다
7. 다 바꿨다면 기존 매개변수를 제거하고 테스트한다

### 예시

온도 측정값 배열에서 정상 작동 범위를 벗어난 것이 있는지 검사하는 코드

```js
// 예시
// ASIS
const station = {
    name: "ZB1",
    readings: [
        {temp: 47, time: "2016-11-10 09:10"},
        {temp: 53, time: "2016-11-10 09:20"},
        {temp: 58, time: "2016-11-10 09:30"},
        {temp: 53, time: "2016-11-10 09:40"},
        {temp: 51, time: "2016-11-10 09:50"},
    ]
};

function readingsOutsideRange(station, min, max) {
    return station.readings.filter(r => r.temp < min || r.temp > max);
}

alerts = readingsOutsideRange(station, operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling);
```

1. 먼저 묶은 데이터를 표현하는 클래스 선언
2. 새로만든 객체를 readingsOutsideRange()함수 선언문에 추가
3. 새로운 객체 생성 후 호출 문에 추가
4. readingsOutsideRange 함수 선언문에서 min, max 변수를 객체로 전환
5. 호출문도 변경

```js
/ TOBE
// Step 1 묶은 데이터를 표현하는 클래스 선언
class NumberRange {
    constructor(min, max) {
        this._data = {min: min, max: max};
    }
    get min() {return this._data.min;}
    get max() {return this._data.max;}
}

// Step 2 readingOutsideRange() 매개변수에 객체 추가
function readingOutsideRange(station, min, max, range) {
    return station.readings.filter(r => r.temp < min || r.temp > max);
}

alerts = readingsOutsideRange(station, operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling, null);

// Step 3 호출문 변경
const range = new NumberRange(operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling);
alerts = readingsOutsideRange(station, operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling, range);

// Step 4 매개변수 max 변경
function readingOutsideRange(station, min, range) {
    return station.readings.filter(r => r.temp < min || r.temp > range.max);
}

alerts = readingsOutsideRange(station, operatingPlan.temperatureFloor, range);

// Step 5 매개변수 min 변경
function readingOutsideRange(station, range) {
    return station.readings.filter(r => r.temp < range.min || r.temp > range.max);
}

alerts = readingsOutsideRange(station, range);
```

### 진정한 값 객체로 거듭나기

매개변수 그룹을 객체로 교체하는 일은 진짜 값진 작업의 준비단계이다.

클래스로 만들어두면 관련 동작들을 이 클래스로 옮길 수 있다는 이점이 생긴다.

위에 예에서 온도가 허용 범위 안에 있는지 검사하는 메서드를 클래스에 추가할 수 있다.

```js
// Step 6 허용범위 메서드 추가
class NumberRange {
    constructor(min, max) {
        this._data = {min: min, max: max};
    }
    get min() {return this._data.min;}
    get max() {return this._data.max;}

    contain(arg) {
        return (arg >= this.min || arg <= this.max);
    };
}
```

코드에 범위 개념이 필요하면 최댓값과 최솟값 쌍을 사용하는 코드를 범위 객체로 바꾸자.

이 값 쌍이 어떻게 사용되는지 살펴보면 다른 유용한 동작도 범위 클래스로 옮겨서 코드베이스 전반에서 값을 활용하는 방식을 간소화할 수 있다.

동치성 검사 메소드(equality method)를 추가 하자

## 6.9 여러 함수를 클래스로 묶기 Combine Functions into Class

### 배경

클래스는 대다수의 최신 프로그래밍 언어가 제공하는 기본적인 빌딩 블록이다.

데이터와 함수를 하나의 공유 환경으로 묶은 후, 다른 프로그램 요소와 어우러질 수  있도록 그중 일부를 외부에 제공한다.

객체 지향 언어의 기본인 동시에 다른 패러다임 언어에도 유용하다.

함수 호출 시 인수로 전달되는 공통 데이터를 중심으로 긴밀하게 엮어 작동하는 함수 무리를 클래스로 묶으면 함수들이 공유하는 공통 환경을 더 명확하게 표현할 수 있고, 함수에 전달되는 인수를 줄여서 객체 안에서의 함수 호출을 간결하게 만들 수 있다.

또한, 이런 객체를 시스템의 다른 부분에 전달하기 위한 참조를 제공할 수 있다.

이 리팩토링은 이미 만들어진 함수들을 재구성할 때는 물론, 새로 만든 클래스와 관련하여 놓친 연산을 찾아서 새 클래스의 메서드로 뽑아내는데 유용하다.

클래스의 장점은 클라이언트가 객체의 핵심 데이터를 변경할 수 있고, 파생 객체들을 일관되게 관리할 수 있다는 것이다.

### 절차

1. 함수들이 공유하는 공통 데이터 레코드를 캡슐화한다.
    1. 공통 데이터가 레코드 구조로 묶여 있지 않다면 먼저 매개변수 객체 만들기로 데이터를 하나로 묶는 레코드를 만든다.
2. 공통 레코드를 사용하는 함수 각각을 새 클래스로 옮긴다
    1. 공통 레코드의 멤버는 함수 호출문의 인수 목록에서 제거한다.
3. 데이터를 조작하는 로직들은 함수로 추출해서 새 클래스로 옮긴다

### 예시

정부에서 차 tea 를 제공하는 예

1. 레코드를 클래스로 변환하기 위해 레코드를 캡슐화한다
2. 이미 만들어져 있는 calculateBaseCharge 함수를 옮긴다.
3. 함수의 이름을 변경한다 → baseCharge
4. 세금을 부과할 소비량을 계산하는 코드를 함수로 추출한다
5. 세금 계산 함수를 Reading 클래스로 옮긴다.

```js
// 예시 정부에서 차를 수돗물처럼 제공하는 예
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 클라이언트 1
const aReading = acquireReading();
const baseCharge = baseRate(aReading.month, aReading.year) * aReading.quantity;

// 클라이언트 2
const aReading = acquireReading();
const base = (baseRate(aReading.month, aReading.year) * aReading.quantity);
const taxableCharge = Math.max(0, base - taxThreshold(aReading.year));

// 클라이언트 3
const aReading = acquireReading();
const basicChargeAmount = calculateBaseCharge(aReading);

function calculateBaseCharge(aReading) {
    return baseRate(aReading.month, aReading.year) * aReading.quantity;
}

// Step 1 레코드를 클래스로 변환하기 위해 레코드 캡슐화
class Reading {
    constructor(data) {
        this._customer = data.customer;
        this._quantity = data.quantity;
        this._month = data.month;
        this._year = data.year;
    }

    get customer() {return this._customer;}
    get quantity() {return this._quantity;}
    get month() {return this._month;}
    get year() {return this._year;}
}

// Step 2 calculateBaseCharge() 옮기기
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const basicChargeAmount = calculateBaseCharge(aReading);

// Step 3 새로 만든 클래스로 calucateBaseCharge() 옮기기
class Reading {
    constructor(data) {
        this._customer = data.customer;
        this._quantity = data.quantity;
        this._month = data.month;
        this._year = data.year;
    }

    get customer() {return this._customer;}
    get quantity() {return this._quantity;}
    get month() {return this._month;}
    get year() {return this._year;}

    get calculateBaseCharge() {
        return baseRate(this.month, this.year) * this.quantity;
    }
}

const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const basicChargeAmount = aReading.calculateBaseCharge;

// Step 4 함수 이름 변경
class Reading {
    constructor(data) {
        this._customer = data.customer;
        this._quantity = data.quantity;
        this._month = data.month;
        this._year = data.year;
    }

    get customer() {return this._customer;}
    get quantity() {return this._quantity;}
    get month() {return this._month;}
    get year() {return this._year;}

    get baseCharge() {
        return baseRate(this.month, this.year) * this.quantity;
    }
}

const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const basicChargeAmount = aReading.baseCharge;

// Step 5 클라이언트 2 세금 계산하는 클라이언트 인라인
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const taxableCharge = Math.max(0, aReading.baseCharge - taxThreshold(aReading.year));

// Step 6 세금을 부과할 소비량을 계산하는 코드를 함수로 추출
class Reading {
    constructor(data) {
        this._customer = data.customer;
        this._quantity = data.quantity;
        this._month = data.month;
        this._year = data.year;
    }

    get customer() {return this._customer;}
    get quantity() {return this._quantity;}
    get month() {return this._month;}
    get year() {return this._year;}

    get baseCharge() {
        return baseRate(this.month, this.year) * this.quantity;
    }
    get taxableCharge() {
        return Math.max(0, this.baseCharge - taxThreshold(this.year))
    }
}

// Step 7 클라이언트 3 수정
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const taxableCharge = aReading.taxableCharge;
```

## 6.10 여러 함수를 변환 함수로 묶기  Combine Functions into Transform

### 배경

도출 작업들을 한데 모아두면 검색과 갱신을 일관된 장소에서 처리할 수 있고 중복도 막을 수 있다.

변환 함수는 원본 데이터를 입력받아서 필요한 정보를 모두 도출한 뒤, 각각을 출력 데이터의 필드에 넣어 반환한다.

### 절차

1. 변환할 레코드를 입력받아서 값을 그대로 반환하는 함수를 만든다
    1. 이 작업은 대체로 깊은 복사로 처리해야 한다. 변환 함수가 원본 레코드를 바꾸지 않는지 검사하는 테스트를 마련해두면 도움이 될 때가 많다
2. 묶을 함수 중 함수 하나를 골라서 본문 코드를 변환 함수로 옮기고, 처리 결과를 레코드에 새 필드로 기록한다. 그런 다음 클라이언트 코드가 이 필드를 사용하도록 수정한다.
    1. 로직이 복잡하면 함수 추출하기부터 한다
3. 테스트한다.
4. 나머지 관련 함수도 위 과정을 따라 처리한다.

### 예시

```js
// ASIS
function base(aReading) {}
function taxableCharge(aReading) {}

// TOBE
function enrichReading(argReading) {
    const aReading = _.cloneDeep(argReading)
    aReading.baseCharge = base(aReading);
    aReading.taxableCharge = taxableCharge(aReading);
    return aReading;
}

// 예시
reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// 클라이언트 1
const aReading = acquireReading();
const baseCharge = baseRate(aReading.month, aReading.year) * aReading.quantity;

// 클라이언트 2
const aReading = acquireReading();
const base = (baseRage(aReading.month, aReading.year) * aReading.quantity);
const taxableCharge = Math.max(0, base - taxThreshold(aReading.year));

// 클라이언트 3
const aReading = acquireReading();
const basicChargeAmount = calculateBaseCharge(aReading);

function calculateBaseCharge(aReading) {
    return baseRate(aReading.month, aReading.year) * aReading.quantity;
}

// Step 1 입력 객체를 그대로 복사해 반환하는 변환 함수 만듬
function enrichReading(original) {
    const result = _.cloneDeep(original);
    return result
}
// Step 2 계산 로직에 측정값을 전달하기 전에 부가 정보를 덧붙이도록 수정한다
// 클라이언트 3
const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const basicChargeAmount = calculateBaseCharge(aReading);

function enrichReading(original) {
    const result = _.cloneDeep(original);
    result.baseCharge = calculateBaseCharge(result);
    return result;   
}
// Step3 함수를 사용하던 클라이언트가 부가 정보를 담은 필드를 사용하도록 수정
// 클라이언트 3
const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const basicChargeAmount = aReading.baseCharge

// enrichReading() 처럼 정보를 추가해 반환할 때 원본 측정값 레코드는 변경하지 않아야 한다. 이를 위한 테스트 코드를 작성한다.
// Step 4
it('check reading unchanged', function () {
    const baseReading = {customer: "ivan", quantity: 15, month: 5, year: 2017};
    const oracle = _.cloneDeep(baseReading);
    enrichReading(baseReading);
    assert.deepEqual(baseReading, oracle);
});
// 클라이언트 1
const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const baseCharge = aReading.baseCharge;

// Step 5 세금 계산 
const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const taxableCharge = Math.max(0, aReading.baseCharge - taxThreshold(aReading.year));

function enrichReading(original) {
    const result = _.cloneDeep(original);
    result.baseCharge = calculateBaseCharge(result);
    result.taxableCharge = Math.max(0, aReading.baseCharge - taxThreshold(aReading.year));
    return result;   
}
// Step 6 계산 코드를 변환 함수로 옮겨서 새로 만든 필드를 사용하여 원본 코드 수정
const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const taxableCharge = aReading.taxableCharge;
```

## 6.11 단계 쪼개기 Split Phase

### 배경

서로 다른 두 대상을 한꺼번에 다루는 코드를 발견하면 각각을 별개의 모듈로 나누는 방법을 모색한다.

코드를 수정해야 할 때 두 대상을 동시에 생각할 필요 없이 하나에만 집중하기 위해서이다.

모듈이 잘 분리되어 있다면 다른 모듈의 상세 내용은 전혀 기억하지 못해도 원하는 대로 수정을 끝낼 수 있다.

분리하는 가장 쉬운 방법은 동작을 연이은 두 단계로 쪼개는 것이다.

입력이 처리 로직에 적합하지 않은 형태로 들어오면 본 작업에 들어가기 전에 입력값을 가공한다.

로직을 순차적인 단계들로 분리해도 된다.

### 절차

1. 두 번째 단계에 해당하는 코드를 독립 함수로 추출한다.
2. 테스트한다.
3. 중간 데이터 구조를 만들어서 앞에서 추출한 함수의 인수로 추가한다.
4. 테스트한다.
5. 추출한 두 번째 단계 함수의 매개변수를 하나씩 검토한다. 그중 첫 번째 단계에서 사용되는 것은 중간 데이터 구조로 옮긴다. 하나씩 옮길 때마다 테스트한다.
    1. 간혹 두 번째 단계에서 사용하면 안 되는 매개변수가 있다. 
    2. 이럴 때는 각 매개변수를 사용한 결과를 중간 데이터 구조의 필드로 추출하고, 이 필드의 값을 설정하는 문장을 호출한 곳으로 옮긴다.
6. 첫 번째 단계 코드를 함수로 추출하면서 중간 데이터 구조를 반환하도록 만든다.
    1. 이때 첫 번째 단계를 변환기 Transformer 객체로 추출해도 좋다.

### 예시1

상품의 결제 금액을 계산하는 코드로 시작

```jsx
// ASIS
const orderData = orderString.split(/\s+/);
const productPrice = priceList[orderData[0].split("-")[1]];
const orderPrice = parseInt(orderData[1]) * productPrice;

// TOBE
const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString) {
    const value = aString.split(/\s+/);
    return ({
        productID: value[0].split("-")[1],
        quantity: parseInt(values[1]),
    });
}

function price(order, priceList) {
    return order.quantity * priceList[order.productID];
}

// 예시 상품의 결제 금액을 계산하는 코드
// ASIS
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
    const shippingPerCase = (basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = basePrice - discount + shippingCost
    return price;
}

// Step 1 배송비 계산 부분을 함수로 추출
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
    const price = applyShipping(basePrice, shippingMethod, quantity, discount);
    return price
}

function applyShipping(basePrice, shippingMethod, quantity, discount) {
    const shippingPerCase = (basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = basePrice - discount + shippingCost;
    return price;
}

// Step 2 첫 번째 단계와 두 번째 단계가 주고받을 중간 데이터 구조를 만든다
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
    const priceData = {}; // 중간 데이터 구조
    const price = applyShipping(priceData, basePrice, shippingMethod, quantity, discount);
    return price
}

function applyShipping(priceData, basePrice, shippingMethod, quantity, discount) {
    const shippingPerCase = (basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = basePrice - discount + shippingCost;
    return price;
}

// Step 3 applyShipping()에 전달되는 매개변수 중 basePrice는 첫 번째 단계에서 생성되니 중간 데이터 구조로 옮기고 매개변수 목록에서 삭제
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
    const priceData = {basePrice: basePrice}; // 중간 데이터 구조
    const price = applyShipping(priceData, shippingMethod, quantity, discount);
    return price
}

function applyShipping(priceData, shippingMethod, quantity, discount) {
    const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = priceData.basePrice - discount + shippingCost;
    return price;
}

// Step 4 quantity를 중간 데이터로 이동
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
    const priceData = {basePrice: basePrice, quantity: quantity}; // 중간 데이터 구조
    const price = applyShipping(priceData, shippingMethod, discount);
    return price
}

function applyShipping(priceData, shippingMethod, discount) {
    const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    const price = priceData.basePrice - discount + shippingCost;
    return price;
}

// Step 5 Discount 처리
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
    const priceData = {basePrice: basePrice, quantity: quantity, discount: discount}; // 중간 데이터 구조
    const price = applyShipping(priceData, shippingMethod);
    return price
}

function applyShipping(priceData, shippingMethod) {
    const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    const price = priceData.basePrice - priceData.discount + shippingCost;
    return price;
}

// Step 6 첫 번째 단계 코드를 함수로 추출하고 데이터 구조 반환
function priceOrder(product, quantity, shippingMethod) {
    const priceData = calculatePricingData(product, quantity);
    const price = applyShipping(priceData, shippingMethod);
    return price
}

function calculatePricingData(product, quantity) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
    return {basePrice: basePrice, quantity: quantity, discount: discount}
}

function applyShipping(priceData, shippingMethod) {
    const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    const price = priceData.basePrice - priceData.discount + shippingCost;
    return price;
}

// Step 7 함수 인라인
function priceOrder(product, quantity, shippingMethod) {
    const priceData = calculatePricingData(product, quantity);
    return applyShipping(priceData, shippingMethod);
}

function calculatePricingData(product, quantity) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
    return {basePrice: basePrice, quantity: quantity, discount: discount}
}

function applyShipping(priceData, shippingMethod) {
    const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    return priceData.basePrice - priceData.discount + shippingCost;
}
```

### 예시2 명령줄 프로그램 쪼개기(자바)

JSON 파일에 담긴 주문의 개수를 세는 자바 프로그램

```java
// 예시 2 명령줄 프로그램 쪼개기(자바) JSON 파일에 담긴 주문의 개수를 세는 자바 프로그램
public static void main(String[] args) {
    try {
        if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
        String filename = args[args.length - 1];
        File input = Paths.get(filename).toFile();
        ObjectMapper mapper = new ObjectMapper();
        Order[] orders = mapper.readValue(input, Order[].class);
        if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
            System.out.println(Stream.of(orders)
                                    .filter(o -> "ready".equals(o.status))
                                    .count());
         else 
            System.out.println(orders.length);
    } catch (Exception e) {
        System.err.println(e);
        System.exit(1);
    }
}
// 위 코드는 두 가지 일을 한다.
// 하나 주문 목록을 읽어서 개수를 센다.
// 둘 명령줄 인수를 담은 배열을 읽어서 프로그램 동작을 결정한다.
// 이 두 가지 일을 분리하면 프로그램에 지정할 수 있는 옵션이나 스위치가 늘어나도 코드를 수정하기 쉽다.

// Step 1 자바로 작성된 명령줄 프로그램을 테스트하기 어려움, 과정이 느리고 복잡
// 일반적인 JUnit 호출로 자바 프로세스 하나에서 테스트 할 수 있는 상태로 만든다.
// 핵심 작업을 수행하는 코드 전부를 함수로 추출
public static void main(String[] args) {
    try {
        run(args)
    } catch (Exception e) {
        System.err.println(e);
        System.exit(1);
    }
}

static void run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    String filename = args[args.length -1];
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        System.out.println(Stream.of(orders)
                                .filter(o -> "ready".equals(o.status))
                                .count());
        else 
        System.out.println(orders.length);
}

// Step 2값을 리턴하고 표출하도록 수정
// 기본 동작을 망치지 않으면서 run() 메서드를 검사하는 JUnit 테스트 작성 가능
// 명령줄 호출과 표준 출력에 쓰는 느리고 불편한 적업과 자주 테스트해야 할 복잡한 동작을 분리
// 험블 객체 패턴(Humble Object Pattern)
// 단, 여기서는 객체가 아니라 main() 메서드에 적용
public static void main(String[] args) {
    try {
        System.out.println(run(args))
    } catch (Exception e) {
        System.err.println(e);
        System.exit(1);
    }
}

static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    String filename = args[args.length -1];
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

// Step 3 두 번째 단계에 해당하는 코드를 독립된 함수로 추출
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    String filename = args[args.length -1];
    return countOrders(args, filename);
}

private static long countOrders(String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

// Step 4 중간 데이터 구조 추가
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine();
    String filename = args[args.length -1];
    return countOrders(commandLine, args, filename);
}

private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static class CommandLine {}

// Step 5 countOrders로 전달되는 인수를 확인, args는 첫 번째 단계에서 사용하기 떄문에 두 번째 단계까지 올 필요가 없다.
// args를 사용하는 부분을 찾아서 그 결과를 추출한다
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine();
    String filename = args[args.length -1];
    return countOrders(commandLine, args, filename);
}

private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    boolean onlyCountReady = Stream.of(args).anyMatch(arg -> "-r".equals(arg));
    if (onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static class CommandLine {}

// Step 6 중간 데이터 구조로 옮긴다
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine();
    String filename = args[args.length -1];
    return countOrders(commandLine, args, filename);
}

private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    commandLine.onlyCountReady = Stream.of(args).anyMatch(arg -> "-r".equals(arg));
    if (commandLine.onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static class CommandLine {
    boolean onlyCountReady;
}

// Step 7 onlyCountReady에 값을 설정하는 문장을 호출한 곳으로 옮긴다
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine();
    String filename = args[args.length -1];
    commandLine.onlyCountReady = Stream.of(args).anyMatch(arg -> "-r".equals(arg));
    return countOrders(commandLine, args, filename);
}

private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (commandLine.onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static class CommandLine {
    boolean onlyCountReady;
}

// Step 8 filename 매개변수를 중간 데이터 구조인 CommandLine에 옮긴다.
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine();
    commandLine.filename = args[args.length -1];
    commandLine.onlyCountReady = Stream.of(args).anyMatch(arg -> "-r".equals(arg));
    return countOrders(commandLine);
}

private static long countOrders(CommandLine commandLine) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (commandLine.onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static class CommandLine {
    boolean onlyCountReady;
    String filename
}

// Step 9 첫번째 코드를 메서드로 추출한다
static long run(String[] args) throws IOException {
    CommandLine commandLine = parseCommandLine(args)
    return countOrders(commandLine);
}

private static CommandLine parseCommandLine(String[] args) {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine();
    commandLine.filename = args[args.length -1];
    commandLine.onlyCountReady = Stream.of(args).anyMatch(arg -> "-r".equals(arg));
    return commandLine
}

private static long countOrders(CommandLine commandLine) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (commandLine.onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static class CommandLine {
    boolean onlyCountReady;
    String filename
}

// Step 10 이름 바꾸기 인라인하기 적용
static long run(String[] args) throws IOException {
    return countOrders(parseCommandLine(args));
}

private static CommandLine parseCommandLine(String[] args) {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine result = new CommandLine();
    result.filename = args[args.length -1];
    result.onlyCountReady = Stream.of(args).anyMatch(arg -> "-r".equals(arg));
    return result
}

private static long countOrders(CommandLine commandLine) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (commandLine.onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static class CommandLine {
    boolean onlyCountReady;
    String filename
}
```

### 예시3 첫 번째 단계에 변환기 사용하기

명령줄 인수를 담은 문자열 배열을 두 번째 단계에 적합한 인터페이스로 바꿔주는 변환기(transformer) 객체를 만듬

```java
// 예시 3 첫 번째 단계에 변환기 사용하기
// 명령줄 인수를 담은 문자열 배열을 두 번째 단계에 적합한 인터페이스로 바꿔주는 변환기(transformer) 객체를 만듬

static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine();
    String filename = args[args.length -1];
    return countOrders(commandLine, args, filename);
}

private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static class CommandLine {}

// Step 1 레코드 구조 대신 동작까지 포함하는 최상위 클래스로 생성
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine();
    String filename = args[args.length -1];
    return countOrders(commandLine, args, filename);
}

private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

public class CommandLine {
    String[] args;

    public CommandLine(String[] args) {
        this.args = args;
    }
}

// Step 2 임시 변수를 질의 함수로 바꾸기
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine, args, filename(args));
}

private static String filename(String[] args) {
    return args[args.length -1];
}

private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

public class CommandLine {
    String[] args;

    public CommandLine(String[] args) {
        this.args = args;
    }
}

// Step 3 질의 함수를 클래스로 옮기기
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine, args, commandLine.filename());
}

private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException {
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

public class CommandLine {
    String[] args;

    public CommandLine(String[] args) {
        this.args = args;
    }

    String filename() {
        return args[args.length -1];
    }
}

// Step 4  함수 선언 바꾸기 
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine, args);
}

private static long countOrders(CommandLine commandLine, String[] args) throws IOException {
    File input = Paths.get(commandLine.filename()).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

public class CommandLine {
    String[] args;

    public CommandLine(String[] args) {
        this.args = args;
    }

    String filename() {
        return args[args.length -1];
    }
}

// Step 5 args 제거하기 위한 조건식 추출
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine, args);
}

private static long countOrders(CommandLine commandLine, String[] args) throws IOException {
    File input = Paths.get(commandLine.filename()).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (onlyCountReady(args))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

private static boolean onlyCountReady(String[] args) {
    return Stream.of(args).anyMatch(arg -> "-r".equals(arg))
}

public class CommandLine {
    String[] args;

    public CommandLine(String[] args) {
        this.args = args;
    }

    String filename() {
        return args[args.length -1];
    }
}

// Step 6 클래스에 함수 옮기기 
static long run(String[] args) throws IOException {
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine);
}

private static long countOrders(CommandLine commandLine) throws IOException {
    File input = Paths.get(commandLine.filename()).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (commandLine.onlyCountReady())
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

public class CommandLine {
    String[] args;

    public CommandLine(String[] args) {
        this.args = args;
    }

    String filename() {
        return args[args.length -1];
    }

    boolean onlyCountReady() {
        return Stream.of(args).anyMatch(arg -> "-r".equals(arg))
    }
}

// Step 7 문장을 함수로 옮기기
static long run(String[] args) throws IOException {
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine);
}

private static long countOrders(CommandLine commandLine) throws IOException {
    File input = Paths.get(commandLine.filename()).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (commandLine.onlyCountReady())
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else 
        return orders.length;
}

public class CommandLine {
    String[] args;
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");

    public CommandLine(String[] args) {
        this.args = args;
    }

    String filename() {
        return args[args.length -1];
    }

    boolean onlyCountReady() {
        return Stream.of(args).anyMatch(arg -> "-r".equals(arg))
    }
}
```

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}