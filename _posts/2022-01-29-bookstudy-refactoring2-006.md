---
layout: single
title: 리팩토링2 스터디 - 006
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 6장 기본적인 리팩터링

가장 기본적인 리팩터링들로 시작한다.

‘함수 추출하기’와 ‘변수 추출하기’를 가장 많이 사용하게 된다. 이 기법의 반대인 ‘함수 인라인하기’, ‘변수 인라인하기’도 자주 사용한다.

추출은 결국 이름 짓기이며, 코드 이해도가 높아지면 이름을 바꿔야할 때가 생긴다.

함수 구성과 이름 짓기는 가장 기본적인 저수준 리팩터링이다. 일단 함수를 만들고 나면 다시 고수준으로 묶어야 한다.

## 함수 추출하기 Extract Function

```js
// ASIS
function printOwing(invoive) {
	printBanner();
	let outstanding = calculateOutstanding();
	
	// 세부 사항 출력
	console.log(`고객명: ${invoice.customer}`);
	console.log(`채무액: ${outstanding}`);

// TOBE
function printOwing(invoice) {
	printBanner();
	let outstanding = calculateOutstanding();
	printDetails(outstanding);
	
	function printDetails(outstanding) {
		console.log(`고객명: ${invoice.customer}`);
		console.log(`채무액: ${outstanding}`);
	}
}
```

### 배경

코드를 언제 독립된 함수로 묶어야 할까? 

저자는 ‘목적과 구현을 분리’하는 방식이 가장 합리적 기준이라고 봤다.

코드를 보고 무슨 일을 하는지 파악하고, 그 부분을 함수로 추출한 뒤 ‘무슨 일’에 걸맞는 이름을 짓는다.

```js
// (목적)무슨 일을 하는가? > 강조(hightlist)
function highlight() {
	// (구현)어떻게 하는가? > 반전(reverse)
	reverse();
}

// 무슨 일을 하는가? > 반전(reverse)
function reverse() {
	// 어떻게 하는가?
	...
}
```

이렇게 하면 나중에 코드를 다시 읽을 때 함수의 목적이 눈에 확 들어오고, 함수 본문 코드에 대해서는 더 이상 신경쓰지 않아도 된다.

함수를 짧게 만들면 함수 호출이 많아져서 성능이 느려질까 걱정하지 마라.

성능 최적화에 대해서는 항상 일반 지침을 따르도록 하자

> 최적화를 할 때는 다음 두 규칙을 따르기 바란다. 첫 번째, 하지 마라. 두 번째, 아직 하지 마라

짧은 함수의 이점은 이름을 잘 지어야만 발휘되므로, 이름 짓기에 특별히 신경써야 한다.

### 절차

1. 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다. (’어떻게’가 아닌 ‘무엇을’ 하는지 드러나도록 한다)
2. 추출할 코드를 원본 함수에서 복사하여 새 함수를 붙여넣는다.
3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사하고, 있으면 파라미터로 전달하라.
4. 변수를 다 처리했다면 컴파일한다.
5. 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꾼다. (위임한다)
6. 테스트한다.
7. 다른 코드에 방금 추출한 것과 똑같거나 비슷한 코드가 없는지 살핀다. 있다면 방금 추출한 새 함수를 호출하도록 바꿀지 검토한다.

#### 예시: 지역 변수의 값을 변경할 때

```js
// ASIS
function printOwing(invoice) {
	...
	let outstanding = 0
	for (const o if invoice.orders) {
		outstanding += o.amount;
	}
	...
}

// TOBE
function printOwing(invoice) {
	...
	const outstanding = calculateOutstanding(invoice);
	...
}

function calculateOutstanding(invoice) {
	let outstanding = 0
	for (const o if invoice.orders) {
		outstanding += o.amount;
	}
	return outstanding;
}
```

## 함수 인라인하기 Inline Function

```js
// ASIS
function getRating(driver) {
	return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver) {
	return driver.numberOFLateDeliveries > 5
}

// TOBE
function getRating(driver) {
	return (driver.numberOfLateDelieries > 5) ? 2 : 1;
}
```

### 배경

목적이 분명히 드러나는 이름의 함수를 이용하기를 권한다.

간접 호출은 유용할 수도 있지만 쓸데없는 간접 호출은 거슬릴 뿐이다.

함수들간의 위임관계가 복잡하게 얽히면 인라인하는 것이 좋다.

### 절차

1. 다형 메서드인지 확인한다.
2. 인라인할 함수를 호출하는 곳을 모두 찾는다.
3. 각 호출문을 함수 본문으로 인라인한다.
4. 하나씩 교체할 때마다 테스트한다.
5. 원래 함수를 삭제한다.

### 예시

```js
// ASIS
function reportLines(aCustomer) {
	const lines = [];
	gatherCustomerData(lines, aCustomer);
	return lines;
}

function gatherCustomerData(out, aCustomer) {
	out.push(["name", aCustomer.name]);
	out.push(["location", aCustomer.location]);
}

// TOBE
function reportLines(aCustomer) {
	const lines = [];
	out.push(["name", aCustomer.name]);
	out.push(["location", aCustomer.location]);
	return lines;
}
```

## 변수 추출하기 Extract Variable

```js
// ASIS
return order.quantity * order.itemPrice - Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 + Math.min(order.quantity * order.itemPrice * 0.1, 100);

// TOBE
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

### 배경

표현식이 너무 복잡해서 이해하기 어려우면 지역 변수를 활용해서 표현식을 관리하기 더 쉽게 만들 수 있다.

이를 통해 코드의 목적을 훨씬 명확하게 드러낼 수 있다.

변수 추출을 고려한다면 문맥을 고려해서 이름을 붙여야 한다.

### 절차

1. 추출하려는 표현식에 부작용은 없는지 확인한다.
2. 불변 변수를 하나 선언하고 이름을 붙일 표현식의 복제본을 대입한다.
3. 원본 표현식을 새로 만든 변수로 교체한다.
4. 테스트한다.
5. 표현식을 여러 곳에서 사용한다면 각각을 새로 만든 변수로 교체한다. 하나 교체할 때마다 테스트 한다.

### 예시: 클래스 안에서

```js
// ASIS
class Order {
	constructor(aRecord) {
		this._data = aRecord;
	}
	
	get quantity() {
		return this._data.quantity;
	}
	
	get itemPrice() {
		return this._data.itemPrice;
	}
	
	get price() {
		return order.quantity * order.itemPrice - Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 + Math.min(order.quantity * order.itemPrice * 0.1, 100);
	}
}

// TOBE
class Order {
	constructor(aRecord) {
		this._data = aRecord;
	}

	get quantity() {
		return this._data.quantity;
	}
	
	get itemPrice() {
		return this._data.itemPrice;
	}
	
	get price() {
		return basePrice - quantityDiscount + shipping;
	}
	
	get basePrice() {
		return order.quantity * order.itemPrice;
	}
	
	get quantityDiscount() {
		return Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
	}

	get shipping() {
		return Math.min(basePrice * 0.1, 100);
	}
}
```

객체는 특정 로직과 데이터를 외부에 공유하려할 때 정보를 설명해주는 문맥이 되어줄 수 있는 장점이 있다.

## 변수 인라인하기 Inline Variable

```js
// ASIS
let basePrice = anOrder.basePrice;
return (basePrice > 1000);

// TOBE
return anOrder.basePrice > 1000;
```

### 배경

원래 표현식과 다를 바 없는 변수는 주변 코드를 살피는데 방해가 되기도 한다.

### 절차

1. 표현식에서 부작용이 없는지 살핀다.
2. 변수가 불변으로 선언되지 않았다면 불변으로 만들고 테스트한다.
3. 이 변수를 가장 처음 사용하는 코드를 찾아서 표현식으로 바꾼다.
4. 테스트한다.
5. 변수를 사용하는 부분을 모두 교체할 때까지 이 과정을 반복한다.
6. 변수 선언문과 대입문을 지운다.
7. 테스트한다.

## 함수 선언 바꾸기 Change Function Declaration

```js
// ASIS
function circum(radius) {
	...
}

// TOBE
function circumference(radius) {
	...
}
```

### 배경

함수는 프로그램을 작은 부분으로 나누는 수단이다.

함수 선언은 각 부분이 서로 맞물리는 방식을 표현하며, 소프트웨어의 구성 요소를 조립하는 연결부 역할을 한다.

연결부를 잘 정의하면 시스템에 새로운 부분을 추가하기가 쉬워지는 반면, 잘못 정의하면 지속적인 방해 요인으로 작용한다.

한수 선언에서 가장 중요한 요소는 함수 이름이다.

이름이 잘못된 함수를 발견하면 더 나은 이름이 떠오르는 즉시 바꿔라.

그래야 그 코드를 다시 볼 때 무슨 일을 하는지 ‘또’ 고민하지 않게 된다.

> 좋은 이름을 떠올리는 데 효과적인 방법이 하나 있다. 바로 주석을 이용해 함수의 목적을 설명해보는 것이다. 그러다 보면 주석이 멋진 이름으로 바뀌어 되돌아올 때가 있다.

함수의 파라미터도 외부 세계와 어우러지는 방식을 정의한다.

잘 정의된 파라미터를 가진 함수는 활용 범위가 넓어지고, 다른 모듈과의 결합을 제거할 수도 있다.

파라미터를 올바르게 선택하기가 어렵다.

따라서 어떻게 연결하는 것이 더 나은지에 대해 더 잘 이해하게 될 때마다 개선할 수 있도록 리팩터링과 친숙해져야 한다.

### 절차

함수 선언 바꾸기는 변경사항을 살펴보고 함수 선언과 호출문들을 단번에 고칠 수 있다면 ‘간단한 절차’를 따르고, 호출하는 곳이 많거나, 호출 과정이 복잡하는 등 점진적으로 고쳐야 한다면 ‘마이그레이션 절차’를 따라 수정한다.

#### 간단한 절차

1. 파라미터를 제거하려거든 먼저 함수 본문에서 제거 대상 파라미터를 참조하는 곳은 없는지 확인한다.
2. 메서드 선언을 원하는 형태로 바꾼다.
3. 기존 메서드 선언을 참조하는 부분을 모두 찾아서 바뀐 형태로 수정한다.
4. 테스트한다.

#### 마이그레이션 절차

1. 이어지는 추출 단계를 수월하게 만들어야 한다면 함수의 본문을 적절히 리팩터링한다.
2. 함수 본문을 새로운 함수로 추출한다.
3. 추출한 함수에 파라미터를 추가해야 한다면 ‘간단한 절차’를 따라 추가한다.
4. 테스트한다.
5. 기존 함수를 인라인한다.
6. 이름을 임시로 붙여뒀다면 ‘함수 선언 바꾸기’를 한 번 더 적용해서 원래 이름으로 되돌린다.
7. 테스트한다.

다형성을 구현한 클래스의 메서드를 변경할 때는 다형 관계인 다른 클래스에도 변경이 반영되어야 한다.

먼저 원하는 형태의 메서드를 새로 만들어서 원래 함수를 호출하게 한다.

공개된 API 를 리팩터링할 때는 새 함수를 추가한 다음 예전 함수를 폐기 대상으로 지정하고 모든 클라이언트가 새 함수로 이전시킨다.

클라이언트들이 모두 새 함수로 이전하면 예전 함수를 지운다.

### 예시: 함수 이름 바꾸기(간단한 절차)

```js
// ASIS
function circum(radius) {
	return 2 * Math.PI * radius;
}

// TOBE
function circumference(radius) {
	return 2 * Math.PI * radius;
}

```

### 예시: 함수 이름 바꾸기(마이그레이션 절차)

```js
// ASIS
function circum(radius) {
	return 2 * Math.PI * radius;
}

// ING-1
function circum(radius) {
	return circumference(radius);
}

function circumference(radius) {
	return 2 * Math.PI * radius;
}

// 클라이언트들이 circum 함수에서 circumference 함수를 호출하도록 변경한다.

// TOBE
function circumference(radius) {
	return 2 * Math.PI * radius;
}
```

### 예시: 매개변수 추가하기

```js
// ASIS
addReservation(customer) {
	this._reservations.push(customer);
}
```

예약 시 우선순위 큐를 지원하라는 요구사항이 발생하였다. 예약정보에 우선순위 큐에 넣을지 여부를 판단하는 파라미터가 추가되어야 한다.

1. 새로운 함수를 생성한다.

```js
// ING-1
addReservation(customer) {
	this.zz_addReservation(customer);
}

zz_addReservation(customer) {
	this._reservations.push(customer);
}
```

1. 파라미터를 추가한다.

```js
// TOBE
addReservation(customer) {
	this.zz_addReservation(customer, false);
}

zz_addReservation(customer, isPriority) {
	assert(isPriority === true || isPriority === false);
	this._reservations.push(customer);
}
```

### 예시: 매개변수를 속성으로 바꾸기

```js
// ASIS
const newEnglanders = someCustomers.filter(c => isNewEngland(c));
function isNewEngland(aCustomer) {
	return ["MA","CT","ME","VT","NH","RI"].includes(aCustomer.address.state);
}
```

주 식별코드를 매개변수로 받도록 리팩터링 할 것이다.

1. 파라미터로 사용할 코드를 변수로 추출한다.

```js
// ING-1
function isNewEngland(aCustomer) {
	const stateCode = aCustomer.address.state;
	return ["MA","CT","ME","VT","NH","RI"].includes(stateCode);
}
```

1. 새로운 함수를 생성한다.

```js
// ING-2
function isNewEngland(aCustomer) {
	const stateCode = aCustomer.address.state;
	xxNewinNewEngland(stateCode);
}

function xxNewinNewEngland(stateCode) {
	return ["MA","CT","ME","VT","NH","RI"].includes(stateCode);
}
```

1. 변수를 인라인한다.

```js
// ING-3
function isNewEngland(aCustomer) {
	xxNewinNewEngland(aCustomer.address.state);
}

function xxNewinNewEngland(stateCode) {
	return ["MA","CT","ME","VT","NH","RI"].includes(stateCode);
}
```

1. 새 함수를 호출하도록 변경한다.

```js
// ING-4
const newEnglanders = someCustomers.filter(c => xxNewinNewEngland(c.address.state));

function isNewEngland(aCustomer) {
	xxNewinNewEngland(aCustomer.address.state);
}

function xxNewinNewEngland(stateCode) {
	return ["MA","CT","ME","VT","NH","RI"].includes(stateCode);
}
```

1. 새 함수의 이름을 변경한다.

```js
// TOBE
const newEnglanders = someCustomers.filter(c => inNewEngland(c.address.state));

function inNewEngland(stateCode) {
	return ["MA","CT","ME","VT","NH","RI"].includes(stateCode);
}
```

## 변수 캡슐화하기 Encapsulate Variable



# 참고
- 