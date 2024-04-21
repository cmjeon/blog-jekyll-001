---
layout: single
title: 리팩터링2 스터디 - 007 - 1
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 7장 캡슐화

모듈을 분리하는 가장 중요한 기준은 각 모듈이 다른 모듈을 상대로 자신의 비밀을 얼마나 잘 숨기느냐에 있다.

데이터 구조는 ‘레코드 캡슐화하기'와 ‘컬렉션 캡슐화하기'로 숨긴다.

기본형 데이터 역시 ‘기본형을 객체로 바꾸기'로 캡슐화할 수 있다.

임시 변수가 많아 길이가 너무 긴 함수는 ‘임시 변수를 질의 함수로 바꾸기' 로 쪼갤 수 있다.

클래스는 본래 정보를 숨기는 용도로 설계되었다. '클래스 추출하기'와 ‘클래스 인라인하기’

클래스는 내부 정보뿐 아니라 클래스 사이의 연결 관계를 숨기는 데도 유용하다. ‘위임 숨기기', ‘중개자 제거하기’

함수의 구현을 캡슐화 하기 위한 '알고리즘 교체하기’

## 7.1 레코드 캡슐화 하기 Encapsulate Record

```js
// ASIS
organization = {
	name:"애크미 구스베리",
	country: "GB"
}

// TOBE
class Organization {
	constructor(data) {
		this._name = data.name;
		this._country = data.country;
	}
	get name() {
		return this._name;
	}
	set name(name) {
		this._name = name;
	}
	get country() {
		return this._country;
	}
	set country(country) {
		this._country = country;
	}
}
```

### 배경

레코드는 연관된 여러 데이터를 직관적인 방식으로 묶을 수 있어서 데이터를 각각 취급할 때보다 훨씬 의미 있는 단위로 전달할 수 있게 해준다.

하지만 레코드는 계산해서 얻을 수 있는 값과 그렇지 못한 값을 명확히 구분해야 한다.

객체를 사용해서 캡슐화하면 무엇을 저장했는지를 감추고 메서드로 제공할 수 있다.

저자는 값이 불변 데이터일 때는 레코드로 저장을 하지만 가변 데이터일 때는 객체를 선호한다.

레코드 구조가 필드 이름을 노출하지 않는 형태도 괜찮지만 복잡해질 때는 불분명함으로 문제가 발생할 수 있다.

이럴 때는 클래스는 사용하는 편이 낫다.

코드를 작성하다 보면 중첩된 리스트나 해시맵을 받아서 JSON 이나 XML 같은 포맷으로 직렬화 Serialize 할 때가 많다.

이런 구조도 캡슐화 해둔다면 포맷을 바꾸거나 추적하기 어려운 데이터를 수정하기 수월해진다.

### 절차

1. 레코드를 담은 변수를 캡슐화한다. 함수의 이름은 검색하기 쉽게 지어준다.
2. 레코드를 감싼 단순한 클래스로 해당 변수의 내용을 교체한다. 원본 레코드를 반환하는 접근자를 만들고 변수를 캡슐화하는 함수들이 이 접근자를 사용하도록 수정한다.
3. 테스트한다.
4. 원본 레코드 대신 새로 정의한 클래스의 객체를 반환하는 함수들을 새로 만든다.
5. 레코드를 반환하는 예전 함수를 사용하는 코드를 새 함수를 사용하도록 바꾼다. 한 부분을 바꿀 때마다 테스트한다.
6. 클래스에서 원본 데이터를 반환하는 접근자와 원본 레코드를 반환하는 함수를 제거한다.
7. 테스트한다.
8. 레코드의 필드도 중첩 구조라면 ‘레코드 캡슐화하기'와 ‘컬렉션 캡슐화하기'를 재귀적으로 적용한다.

### 예시: 간단한 레코드 캡슐화하기

```js
// ASIS
const organization = {
  name: "애크미 구스베리", 
  country: "GB"
}

// 클라이언트
// 읽기 예
result += `<h1>${organization.name}</h1?`;
// 쓰기 예
organization.name = newName;
```

이 상수를 임시함수로 캡슐화하고, 사용하는 코드를 그에 맞게 변경한다.

```js
function getRawDataOfOrganization() { // here
  return organization;
}
    
// 클라이언트
result += `<h1>${getRawDataOfOrganization().name}<h1>`;
    
getRawDataOfOrganization().name = newName;
```

레코드를 클래스로 변경한다.

```js
// Organization 클래스
class Organization { // here
  constructor(data) {
    this._data = data;
  }
}

const organization = new Organization({ // here
  name: "애크미 구스베리",
  country: "GB"
});

function getRawDataOfOrganization() {
  return organization._data;
}

function getOrganization() {
  return organization;
}
```

게터와 세터를 만든다.

```js
// Organization 클래스
set name(name) {
  this._data.name = name;
}

get name() {
  return this._data.name;
}
```

레코드를 사용하던 코드를 모두 세터를 사용하도록 고친다.

```js
// 클라이언트
result += `<h1>${getOrganization().name}<h1>`;

getOrganization().name = newName;
```

임시 함수를 제거한다.

```js
// Organization 클래스
function getRawDataOfOrganization() { // 삭제
  return organization._data;
}
```

_data 필드를 객체 안에 바로 펼친다.

```js
// TOBE
class Organization {
  constructor(data) {
    this._name = data.name;
    this._country = data.country;
  }
  get name() {
    return this._name;
  }
  set name(name) {
    this._name = name;
  }
  get country() {
    return this._country;
  }
  set country(countryCode) {
    this._name = countryCode;
  }
}
```

### 예시: 중첩된 레코드 캡슐화하기

```js
// ASIS
"1920": {
	name: "마틴 파울러",
	id: "1920",
	usages: {
		"2016": {
			"1": 50,
			"2": 55.
			...
		},
		"2015": {
			"1": 70,
			"2": 63,
			...
		},
	}
"38673": {
	name: "닐 포드",
	id: "38673",
	...
}

// 쓰기 예
customerData[customerID].usages[year][month] = amount;
// 읽기 예
function compareUsage(customerID, lateYear, month) {
	const later = customerData[customerID].usage[laterYear][month];
	const earlier = customerData[customerID].usage[laterYear-1][month];
	return {
		laterAmount: later, change: later - earlier
	};
}
```

변수 캡슐화를 시작한다.

```js
// 최상위
function getRawDataOfCustomers() { // here
	return customerData;
}

function setRawDataOfCustomers(arg) { // here
	customerData = arg;
}

// 쓰기 예
getRawDataOfCustomers()[customerID].usage[year][month] = amount;
// 읽기 예
function compareUsage(customerID, laterYear, month) {
	const later = getRawDataOfCustomers()[customerID].usages[laterYear][month];
	const earlier = getRawDataOfCustomers()[customerID].usages[laterYear - 1][month];
	return {
		laterAmount: later,
		change:later - earlier
	};
}
```

전체 데이터 구조를 표현하는 클래스를 정의하고 반환하는 함수를 만든다.

```js
// CustomerData 클래스
class CustomerData { // here
	constructor(data) {
		this._data = data;
	}
}

// 최상위
function setRawDataOfCustomers(arg) {
	customerData = new CustomerData(arg);
}

function getCustomerData() {
	return customerData;
}

function getRawDataOfCustomers() {
	return customerData._data;
}
```

데이터 구조 안으로 들어가는 코드를 세터로 뽑아내는 작업을 하고, CustomerData 클래스로 옮긴다.

```js
// CustomerData 클래스
setUsage(customerID, year, month, amount) {
	this._data[customerID].usages[year][month] = amount;
}

// 쓰기 예
getCustomerData().setUsage(customerID, year, month, amount);
```

덩치 큰 데이터 구조를 다룰 때는 쓰기 부분에 집중해야 한다. 캡슐화에서는 값을 수정하는 부분을 명확하게 드러내고 한 곳에 모아두는 일이 중요하다.

- 쓰기 부분을 수정하고 확인하는 방법
  1. lodash 의 cloneDeep 을 활용하여 깊은 복사
    
     ```js
     get rawData() {
       return _.cloneDeep(this._data);
     }
     ```
     
  2. 데이터 구조의 읽기전용 프록시를 반환

읽기를 다루는 방법도 있다.

첫번째 세터 때와 같은 방법이 있다. 즉, 읽는 코드를 모두 독립 함수로 추출하고 CustomerData 클래스로 옮긴다.

```js
// CustomerData 클래스
getUsage(customerID, year, month) {
	return this._data[customerID].usages[year][month];
}

// 최상위
function compareUsage(customerID, laterYear, month) {
	const later = getCustomerData().getUsage(customerID, laterYear, month);
	const earlier = getCustomerData().getUsage(customerID, laterYear - 1, month);
	return {
		laterAmount: later,
		change: later - ealier
	};
}
```

위 방법은 클래스의 모든 쓰임을 명시적인 API로 제공한다는 것이다. 하지만 읽는 패턴이 다양하면 코드량이 늘어난다.

두번째 클라이언트가 데이터 구조를 요청할 때 실제 데이터를 제공하지만 클라이언트가 데이터를 직접 수정하지 못하게 막을 방법이 없기 때문에 복제된 데이터를 제공하는 것이다.

```js
// CustomerData 클래스
get rawData() {
	return _.cloneDeep(this._data)
}

// 최상위
function compareUsage(customerID, laterYear, month) {
	const later = getCustomerData().rawData[customerID].usages[laterYear][month];
	const earlier = getCustomerData().rawData[customerID].usages[laterYear - 1][month];
	return {
		laterAmount: later,
		change: later - ealier
	};
}
```

위 방법은 간단하지만 데이터 구조가 커지면 복제비용이 커진다.

세번째 레코드 캡슐화를 재귀적으로 수행하는 것이다.

   1. 고객 정보 데이터를 클래스로 바꾼 뒤, 레코드를 다루는 코드를 ‘컬렉션 캡슐화하기’ 로 고객 정보를 다루는 클래스를 생성한다.
   2. 자유로운 갱신을 막기위해 접근자를 만든다.
 
이 방법은 데이터 구조가 거대하면 일이 상당히 커지고, 데이터 구조를 사용할 일이 없다면 효과도 별로 없다. 
새로 만든 클래스와 게터를 잘 혼합해서 게터는 데이터 구조를 깊이 탐색하게 만들되 객체로 감싸서 반환하는게 효과적일 수 있다.

## 7.2 컬렉션 캡슐화하기 Encapsulate Collecetion

```js
// ASIS
class Person {
	get courses() {
		return this._courses;
	}
	set courses(aList) {
		this._courses = aList;
	}
}

// TOBE
class Person {
	get coureses() {
		return this._courses.slice();
	}
	addCourse(aCourse) {
		...
	}
	removeCourse(aCourse) {
		...
	}
}
```

### 배경

가변 데이터는 캡슐화하는 편이 좋다. 데이터 구조가 언제 어떻게 수정되는지 파악하기 쉬워서 필요한 시점에 데이터 구조를 변경하기도 쉬워지기 때문이다.

하지만 클래스에서 컬렉션 변수로의 접근을 캡슐화하면서 게터가 컬렉션 자체를 반환해버리면 그 컬렉션을 감싼 클래스가 눈치채지 못한 상태에서 컬렉션의 원소들이 바뀔 수 있다.

따라서 컬렉션 자체도 add() 와 remove() 라는 이름의 컬렉션 변경자 메서드를 통하도록 개선하면 좋다.

그리고 컬렉션 게터가 원본 컬렉션을 반환하지 않도록 만드는 것도 좋다.

내부 컬렉션을 직접 수정하지 못하게 막는 방법으로 컬렉션을 읽기 전용으로만 제공하거나 컬렉션 게터를 제공하되 복제본을 반환하게 하는 방법이 있다.

읽기 전용이든 복제본 반환이든 전체 코드베이스에서 일관성이 있어야 한다.

### 절차

1. 컬렉션을 캡슐화하지 않았다면 ‘변수 캡슐화하기’부터 한다.
2. 컬렉션에 원소를 추가/제거하는 함수를 추가한다.
    1. 컬렉션 자체를 통째로 바꾸는 세터는 제거해야 함.
3. 정적 검사를 수행한다.
4. 컬렉션을 참조하는 부분을 모두 찾는다.
    1. 컬렉션의 변경자를 호출하는 코드를 컬렉션에 원소를 추가/제거하는 함수를 호출하도록 수정한다.
    2. 하나씩 수정할 때마다 테스트한다.
5. 컬렉션 게터를 수정해서 원본 내용을 수정할 수 없는 읽기 전용 프락시나 복제본을 반환하게 한다.

### 예시

```js
// ASIS
class Person {
	constructor(name) {
		this._name = name;
		this._courses = [];
	}
	get name() {
		return this._name;
	}
	get courses() {
		return this._courses;
	}
	set courses(aList) {
		this._courses = aList;
	}
}

class Course {
	construcctor(name, isAdvanced) {
		this._name = name;
		this._isAdvanced = isAdvanced;
	}
	get name() {
		return this._name;
	}
	get isAdvanced() {
		return this._isAdvanced;
	}
}
```

클라이언트는 Person 이 제공하는 수업 courses 컬렉션에서 수업 정보를 얻고 있다.

클라이언트가 수업 컬렉션을 통째로 설정하면 누구든 마음대로 수정할 수 있다.

```js
// 클라이언트
const basicCourseNames = readBasicCourseNames(fileName);
aPerson.courses = basicCourseNames.map(name => new Course(name, false));
```

클라이언트 입장에서는 아래와 같은 방법이 편할 수 있다.

```js
for(const name of readBasicCourseNames(filename)) {
	aPerson.courses.push(new Course(name, false));
}
```

이런 방식은 Person 클래스가 컬렉션을 제어할 수 없게되어 캡슐화가 깨지게 된다.

1. 클라이언트가 수업을 추가/제거할 수 있게 하는 메서드를 Person 에 추가한다.

```js
class Person {
	fnIfAbsent = () => { 
		throw new RangeError(); 
	}
	addCourse(aCourse) { // here
		this._courses.push(aCourse);
	}
	removeCourse(aCourse, fnIfAbsent) { // here
		const index = this._courses.indexOf(aCourse);
		if(index === -1) fnIfAbsent();
		else this._courses.aplice(index, 1);
	}
}
```

2. 컬렉션의 변경자를 호출하던 코드를 찾아서 방금 추가한 메서드를 사용하도록 변경한다.

```js
// 클라이언트
for(const name of readBasicCourseNames(filename)) {
	aPerson.addCourse(new Course(name, false));
}
```

1. setCourses() 는 사용할 일이 없으니 제거한다. 세터를 제공해야 할 이유가 있다면 인수로 받은 컬렉션의 복제본을 저장하게 한다. 게터도 복제본을 제공한다.

```js
class Person {
	set courses(aList) {
		this._courses = aList.slice(); // here
	}
	get courses() {
		return this._courses.slice(); // here
	}
}
```

컬렉션에 대해서는 어느 정도 강박증을 갖고 불필요한 복제본을 만드는 편이 나중의 오류보다는 낫다.

컬렉션 관리를 책임지는 클래스라면 항상 복제본을 제공해야 한다.

컬렉션을 변경할 가능성이 있는 작업을 할 때도 습관적으로 복제본을 만든다.

## 7.3 기본형을 객체로 바꾸기 Replace Primitive with Object

```js
// ASIS
orders.filter(o => "high" === o.priority || "rush" === o.priority);

// TOBE
orders.filter(o => o.priority.higherThan(new Priority("normal")))

```

### 배경

개발 초기에는 단순한 데이터였지만 이 후 포매팅이나 지역 코드 추출 같은 특별한 동작이 필요해질 수 있다.

이런 로직들은 중복코드로 작성되게 된다.

단순한 출력 이상의 기능이 필요해지는 순간 그 데이터를 표현하는 전용 클래스를 정의하는 게 좋다.

시작은 기본형 데이터를 단순히 감싼 것 같지만 나중에 특별한 동작이 필요해지면 이 클래스에 추가하면 되니 갈수록 유용하게 된다.

### 절차

1. 아직 변수를 캡슐화하지 않았다면 ‘캡슐화' 한다.
2. 단순한 값 클래스를 만든다. 생성자는 기존 값을 인수로 받아서 저장하고, 이 값을 반환하는 게터를 만든다.
3. 정적 검사를 수행한다.
4. 값 클래스의 인스턴스를 새로 만들어서 필드에 저장하도록 변수의 세터를 수정한다.
5. 새로 만든 값 클래스의 게터를 호출하도록 변수의 게터를 수정한다.
6. 테스트한다.
7. 함수 이름을 바꾸면 원본 접근자의 동작을 더 잘 드러낼 수 있는지 검토해본다..

### 예시

```js
// ASIS
class Order {
	constructor(data) {
		this.priority = data.priority;
		...
	}
}

// 클라이언트
highPriorityCount = orders.filter(o => "high" == o.priority || "rush" === o.priority).length;
```

1. 먼저 변수를 캡슐화 한다.

```js
class Order {
	get priority() {
		return this._priority;
	}
	set priority(aString) {
		this._priority = aString;
	}
```

2. Priority 클래스를 만든다.

```js
class Priority {
	constructor(value) {
		this._value = value;	
	}
	// 왜 게터가 아닌 toString ?
	toString() {
		return this._value;
	}
}
```

저자는 게터 (value()) 보다는 변환 함수 (toString()) 을 선호한다.

클라이언트 입장에서는 속성 자체를 받은게 아닌 속성을 문자열로 표현한 값을 요청한 것이 되기 때문이다.

3. Priority 클래스를 사용하도록 접근자를 수정한다.

```js
// Order 클래스
get priority() {
	return this._priority.toString();
}
set priority(aString) {
	this._priority = new Priority(aString);
}
```

4. Order 클래스의 게터명을 toString 을 반환하는 것을 명시하도록 변경한다.

```js
// Order 클래스
get priorityString() {
	return this._priority.toString();
}
set priority(aString) {
	this._priority = new Priority(aString);
}

// 클라이언트
highPriorityCount = orders.filter(o => "high" == o.priorityString || "rush" === o.priorityString).length;
```

### 더 가다듬기

Order 클래스에 Priority 객체를 제공하는 게터를 만드는 것이 좋을 수도 있다.

```js
// Order 클래스
get Priority() { // here
	return this._priority;
}
get priorityString() {
	return this._priority.toString();
}
set priority(aString) {
	this._priority = new Priority(aString);
}

// 클라이언트
highPriorityCount = orders.filter(o => "high" == o.priority.toString || "rush" === o.prioritys.toString).length;
```

Priority 클래스가 다른 곳에서도 유용할 수 있으니, Order 의 세터에서 Priority 인스턴스를 받도록 해주면 좋다.

이를 위해 Priority 생성자를 변경한다.

```js
// Priority 클래스
constructor(value) {
	if(value instanceof Priority) {
		return value;
	}
	this._value = value;
}
```

Priority 클래스가 새로운 동작을 담는 장소로 활용하기 위해서다.

따라서 우선순위 값을 검증하고 비교하는 로직을 추가해본다.

```js
// Priority 클래스
constructor(value) {
	if(value instanceof Priority) {
		return value; 
	}
	if(Priority.legalValues().includes(value)) {
		this._value = value;
	} else {
		throw new Error(`<${value}> is invalid for Priority`);
	}
}
toString() {
	return this._value;
}
get _index() {
	return Priority.legalValues().findIndex(s => s === this._value);
}
static legalValues() {
	return ['low', 'normal', 'high', 'rush'];
}
equals(other) {
	return this._index === other._index;
} 
higherThan(other) {
	return this._index > other._index;
}
lowerThan(other) {
	return this._index < other._index;
}

// 클라이언트
highPriorityCount = orders.filter(o => o.priority.higherThan(new Priority("normal"))).length;
```

## 7.4 임시 변수를 질의 함수로 바꾸기 Replace Temp with Query

```js
// ASIS
const basePrice = this._quantity * this._itemPrice;
if(basePrice > 1000) {
  return basePrice * 0.95;
} else {
  return basePrice * 0.98;
}

// TOBE
get basePrice() {
  this._quantity * this._itemPrice;
}
if(this.basePrice > 1000) {
  return this.basePrice * 0.95;
} else {
  return this.basePrice * 0.98;
}
```

### 배경

함수 안에서 계산식을 임시 변수로 만들고 뒤에서 참조하는 식으로 많이 쓰인다.

한 걸음 더 나아가 아예 함수로 만드는 편이 나을 때가 많다.

긴 함수의 한 부분을 별도 함수로 추출하고자 할 때 먼저 변수들을 각각의 함수로 만들어 두면 일이 수월하다.

변수를 함수로 만들면 다른 함수에서도 쓸 수 있어서 중복이 줄어든다.

이 리팩터링은 클래스 안에서 적용할 때 효과가 가장 크다.

클래스 바깥의 최상위 함수에서 적용하면 매개변수가 너무 많아져서 장점이 줄어든다.

임시 변수는 값을 한 번만 계산하고 그 뒤로는 읽기만 해야 한다.

만약 임시 변수에 어떤 계산 로직을 대입하고 다음에 다른 계산 로직을 대입하는 경우는 별도의 변수로 분리해야 한다.

### 절차

1. 변수가 사용되기 전에 값이 확실히 결정되었는지, 변수를 사용할 때마다 계산 로직이 매번 다른 결과를 내지는 않는지 확인한다.
2. 읽기전용으로 만들 수 있는 변수는 읽기전용으로 만든다.
3. 테스트한다.
4. 변수 대입문을 함수로 추출한다.
    1. 변수와 함수의 이름이 같을 수 없다면 함수 이름을 임시로 짓는다.
    2. 추출한 함수가 부수적인 효과를 일으킨다면 ‘질의 함수와 변경 함수 분리하기'로 대응한다.
5. 테스트한다.
6. ‘변수 인라인하기'로 임시 변수를 제거한다.

### 예시

```js
// Order 클래스
constructor(quantity, item) {
  this._quantity = quantity;
  this._item = item;
}

get price() {
  var basePrice = this._quantity * this._item.price;
  var discountFactor = 0.98;

  if(basePrice > 1000) {
    discountFactor -= 0.03;
    return basePrice * discountFactor;
  } 
}
```

basePrice 와 discountFactor 를 메서드로 바꿔본다.

1. basePrice 에 const 를 붙여 읽기전용으로 만들고 테스트해본다.
    
    이렇게 하면 놓친 재대입 코드를 찾을 수 있다.
    

```js
// Order 클래스
constructor(quantity, item) {
  this._quantity = quantity;
  this._item = item;
}

get price() {
  const basePrice = this._quantity * this._item.price; // here
  var discountFactor = 0.98;
  if(basePrice > 1000) {
    discountFactor -= 0.03;
    return basePrice * discountFactor;
  }
}
```

2. 대입문의 우변을 게터로 추출한다.

```js
// Order 클래스
get price() {
  const basePrice = this.basePrice;
  var discountFactor = 0.98;
  if(basePrice > 1000) {
    discountFactor -= 0.03;
    return basePrice * discountFactor;
  }
}

get basePrice() { // here
  return this._quantity * this._item.price;
}
```

3. 테스트 한 뒤 변수를 인라인 한다.

```js
// Order 클래스
get price() {
  var discountFactor = 0.98;
  if(this.basePrice > 1000) {
    discountFactor -= 0.03;
    return this.basePrice * discountFactor; // here
  }
}
```

4. discountFactor 도 같은 순서로 처리하기 위해 const 로 선언하고 테스트해본 뒤, 문제 없으면 게터로 추출한다.

```js
// Order 클래스
get price() {
  const discountFactor = this.discountFactor; // here
  return this.basePrice * discountFactor;
}

get discountFactor() { // here
  var discountFactor = 0.98;
  if(this.basePrice > 1000) {
    discountFactor -= 0.03;
    return discountFactor;
  }
}
```

5. 변수를 인라인 한다.

```js
// Order 클래스
get price() {
  return this.basePrice * this.discountFactor;
}

get basePrice() {
  return this._quantity * this._item.price;
}

get discountFactor() {
  var discountFactor = 0.98;
  if(this.basePrice > 1000) {
    discountFactor -= 0.03;
    return discountFactor;
  }
}
```

## 7.5 클래스 추출하기 Extract Class

```js
// ASIS
class Person {
  get officeAreaCode() {
    return this._officeAreaCode;
  }
  get officeNumber() {
    return this._officeNumber;
  }
	...
}

// TOBE
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
```

### 배경

클래스는 반드시 명확하게 추상화되고 소수의 주어진 역할만 처리해야 하지만 실무에서는 종종 클래스가 점점 비대해지곤 한다.

클래스 내의 연관성이 높은 일부 데이터와 메서드가 따로 클래스로 분리될 수 있다면 분리하는게 좋다.

### 절차

1. 클래스의 역할을 분리할 방법을 정한다.
2. 분리될 역할을 담당할 클래스를 새로 만든다.
3. 원래 클래스의 생성자에서 새로운 클래스의 인스턴스를 생성하여 필드에 저장해둔다.
4. 분리될 역할에 필요한 필드들을 새 클래스로 옮긴다. 옮기면서 테스트한다.
5. 메서드들도 새 클래스로 옮긴다. 옮기면서 테스트 한다.
6. 양쪽 클래스의 인터페이스를 살펴보면서 불필요한 메서드를 제거하고 이름을 변경한다.
7. 새 클래스를 외부로 노출할지 정한다.

### 예시

```js
// Person 클래스
get name() {
  return this._name;
}
set name(arg) {
  this._name =arg
}
get telephoneNumber() {
  return `(${this.officeAreaCode}) ${this.officeNumber}`;
}
get officeAreaCode() {
  return this._officeAreaCode;
}
set officeAreaCode(arg) {
  this._officeAreaCode = arg;
}
get officeNumber() {
  return this._officeNumber;
}
set officeNumber(arg) {
  this._officeNumber = arg;
}
```

1. 전화번호 관련 동작만을 별도의 클래스로 추출하기위해 빈 전화번호를 표현하는 TelephoneNumber 클래스를 정의한다.

```js
class TelephoneNumber {
}
```

2. Person 클래스의 인스턴스를 생성할 때 전화번호 인스턴스도 함께 생성한다.

```js
// Person 클래스
constructor() {
  this._telephoneNumber = new TelephoneNumber();
}

// TelephoneNumber 클래스
get officeAreaCode() {
  return this._officeAreaCode;
}
set officeAreaCode(arg) {
  this._officeAreaCode = arg;
}

```

3. Person 클래스의 officeAreaCode 필드를 TelephoneNumber 클래스로 옮긴다.

```js
// Person 클래스
get officeAreaCode() {
  return this._telephoneNumber.officeAreaCode;
}
set officeAreaCode(arg) {
  this._telephoneNumber.officeAreaCode = arg;
}
```

4. TelephoneNumber 클래스에 officeNumber 를 생성하고, Person 클래스의 officeNumber 필드를 새 클래스로 옮긴다.

```js
// TelephoneNumber 클래스
get officeNumber() {
  return this._officeNumber;
}
set officeNumber(arg) {
  this._officeNumber = arg;
}

// Person 클래스
get officeNumber() {
  return this._telephoneNumber.officeNumber;
}
set officeNumber(arg) {
  this._telephoneNumber.officeNumber = arg;
}
```

5. 테스트 후 telephoneNumber() 메서드도 TelephoneNumber 클래스로 옮긴다.

```js
// TelephoneNumber 클래스
get telephoneNumber() {
  return `(${this.officeAreaCode}) ${this.officeNumber}`;
}

// Person 클래스
get telephoneNumber() {
  return this._telephoneNumber.telephoneNumber;
}
```

6. 새로 만든 클래스는 전화번호를 뜻하므로 office 라는 단어를 쓸 이유가 없고, 마찬가지로 전화번호라는 뜻도 메서드 이름에 쓸 이유가 없다.

```js
// TelephoneNumber 클래스
get areaCode() {
  return this._areaCode;
}
set areaCode(arg) {
  this._areaCode = arg;
}
get number() {
  return this._number;
}
set number(arg) {
  this._number = arg;
}

// Person 클래스
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
```

7. 마지막으로 사람이 읽기 좋은 포맷으로 출력하는 메소드를 추가한다.

```js
// TelephoneNumber 클래스
toString() {
  return `(${this.areaCode}) ${this.number}`;
}

// Person 클래스
get telephoneNumber() {
  return this._telephoneNumber.toString();
}
```

전화번호는 여러 군데에서 쓰일 일이 많으니 이 클래스는 클라이언트에 공개하는 것이 좋겠다.

그러면 ‘office’ 로 시작하는 메서드들을 없애고 TelephoneNumber 의 접근자를 바로 사용하도록 바꿀 수 있다.

이런 경우라면 전화번호를 값 객체로 만드는 것이 나으니 ‘참조를 값으로 바꾸기' 부터 적용한다.

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}
