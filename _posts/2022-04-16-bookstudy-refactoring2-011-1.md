---
layout: single
title: 리팩터링2 스터디 - 011 - 1
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 11장 API 리팩터링

모듈과 함수는 소프트웨어를 구성하는 빌딩 블록이고, API 는 이 블록들을 끼워 맞추는 연결부다.

좋은 API 는 데이터를 갱신하는 함수와 그저 조회만 하는 함수를 명확히 구분한다.

클래스는 대표적인 모듈이고, 객체가 되도록 불변으로 존재하면 좋다.

## 11.1 질의 함수와 변경 함수 분리하기 Separate Query from Modifier

```jsx
// AIIS
function getTotalOutstandingAndSendBill() {
	const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
	sendBill();
	return result;
}

// TOBE
function totalOutstanding() {
	return customer.invoices.reduce((total, each) => each.amount + total, 0);
}
function sendBill() {
	emailGateway.send(formatBill(customer));
}
```

### 배경

우리는 외부에서 관찰할 수 있는 겉보기 부수효과 observable side effect 가 전혀 없이 값을 반환해주는 함수를 추구해야 한다.

겉보기 부수효과가 있는 함수와 없는 함수는 명확히 구분하는 것이 좋다. 이를 위한 한 가지 방법은 ‘질의 함수(읽기 함수)는 모두 부수효과가 없어야 한다' 는 규칙을 따르는 것이다.

이를 명령-질의 분리 command-query separation 이라 한다.

값을 반화하면서 부수효과도 있는 함수를 발견하면 상태를 변경하는 부분과 질의하는 부분을 분리하려 시도해라.

흔히 쓰는 최적화 기법 중 요청된 값을 캐시해두고 다음번 호출 때 빠르게 응답하는 방법이 있는데, 이러한 캐싱도 객체의 상태를 변경하지만 객체 밖에서는 관찰할 수 없다.

즉, 겉보기 부수효과 없이 어떤 순서로 호출하든 모든 호출에 항상 똑같은 값을 반환할 뿐이다.

### 절차

1. 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다.
   함수 내부를 살펴보며 무엇을 반환하는지 찾아서 단서로 삼는다.
2. 새 질의 함수에서 부수효과를 모두 제거한다.
3. 정적 검사를 수행한다.
4. 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아낸다. 호출하는 곳에서 반환 값을 사용한다면 질의 함수를 호출하도록 바꾸고, 원래 함수를 호출하는 코드를 바로 아래 줄에 새로 추가한다. 하나 수정할 때마다 테스트한다.
5. 원래 함수에서 질의 관련 코드를 제거한다.

### 예시

이름 목록을 조회하여 악당 miscreant 을 찾는 함수이다. 악당을 찾으면 그 사람의 이름을 반환하고 경고를 울린다.

```jsx
function alertForMiscreant(people) {
	for(const p of people) {
		if(p === "조커") {
			setOffAlarms();
			return "조커";
		}
		if(p === "사루만") {
			setOffAlarms();
			return "사루만";
		}
	}
	return "";
}
```

1. 첫 단계는 함수를 복제하고 질의 목적에 맞는 이름을 짓는다.

```jsx
function findMiscreant(people) { // here
	for(const p of people) {
		if(p === "조커") {
			setOffAlarms();
			return "조커";
		}
		if(p === "사루만") {
			setOffAlarms();
			return "사루만";
		}
	}
	return "";
}
```

1. 새 질의 함수에서 부수효과를 만드는 부분을 제거한다.

```jsx
function findMiscreant(people) {
	for(const p of people) {
		if(p === "조커") {
			// setOffAlarms();  // here
			return "조커";
		}
		if(p === "사루만") {
			// setOffAlarms(); // here
			return "사루만";
		}
	}
	return "";
}
```

1. 원래 함수를 호출하는 곳을 찾아서 새로운 질의 함수를 호출하도록 변경한다. 그리고 원래의 함수를 호출하는 코드 바로 아래에 삽입한다.

```jsx
// 클라이언트
// ASIS
const found = alertForMiscreant(people);

// TOBE
const found = findMiscreant(people);
alertForMiscreant(found);

```

1. 이제 원래의 변경 함수에서 질의 관련 코드를 삭제한다.

```jsx
function findMiscreant(people) { // here
	for(const p of people) {
		if(p === "조커") {
			setOffAlarms();
			return; // here
		}
		if(p === "사루만") {
			setOffAlarms();
			return; // here
		}
	}
	return "";
}
```

## 더 가다듬기

변경 함수와 질의 함수에 있는 중복 코드는 변경 함수에서 질의 함수를 사용하도록 고쳐서 해결한다

```jsx
function alertForMiscreatn(people) {
	if(findMiscreant(people) !== "") setOffAlarms();
}
```

## 11.2 함수 매개변수화하기 Parameterize Function

```jsx
// ASIS
function tenPercentRaise(aPerson) {
	aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
	aPerson.salary = aPerson.salary.multiply(1.05);
}

// TOBE
function raise(aPerson, factor) {
	aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

### 배경

두 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면 그 다른 값만 매개변수로 받아 처리하는 함수 하나로 합쳐서 중복을 없앨 수 있다.

매개변수 값만 바꿔서 여러 곳에 쓸 수 있는 함수로 바뀌어 유용성이 커진다.

### 절차

1. 비슷한 함수 중 하나를 선택한다.
2. 함수 선언 바꾸기로 리터럴들을 매개변수로 추가한다.
3. 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다.
4. 테스트한다.
5. 매개변수로 받은 값을 사용하도록 함수 본문을 수정한다. 하나 수정할 때마다 테스트한다.
6. 비슷한 다른 함수를 호출하는 코드를 찾아 매개변수화된 함수를 호출하도록 하나씩 수정한다. 하나 수정할 때마다 테스트한다.

### 예시

```jsx
function baseCharge(usage) {
	if(usage < 0) return usd(0);
	const amount = bottomBand(usage) * 0.03
				+ middleBand(usage) * 0.05
				+ topBand(usage) * 0.07;
	return usd(amount);
}

function bottomBand(usage) {
	return Math.min(usage, 100);
}

function middleBand(usage) {
	return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}

function topBand(usage) {
	return usage > 200 ? usage - 200 : 0;
}
```

대역 band 을 다루는 세 함수의 로직이 상당히 비슷해 보인다.

1. 대상 함수 중 하나를 골라 매개변수를 추가한다. 여기서는 middleBand() 에 매개변수를 추가하고, 다른 호출들을 여기에 맞춰본다.
2. middleBand() 가 사용하는 리터럴 두 개(100, 200) 를 호출 시점에 입력하도록 변경한다. middleBand() 함수 이름도 적절한 이름인 withinBand 로 수정한다.

```jsx
function withinBand(usage, bottom, top) { // here
	return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}

function baseCharge(usage) {
	if(usage < 0) return usd(0);
	const amount = bottomBand(usage) * 0.03
				+ withinBand(usage, 100, 200) * 0.05 // here
				+ topBand(usage) * 0.07;
	return usd(amount);
}
```

1. 함수에서 사용하던 리터럴들을 매개변수로 대체한다.

```jsx
function withinBand(usage, bottom, top) {
	return usage > bottom ? Math.min(usage, top) - bottom : 0; // here
}
```

1. 대역의 하한을 호출하는 부분도 새로 만든 매개변수와 함수를 호출하도록 변경한다.

```jsx
function baseCharge(usage) {
	if(usage < 0) return usd(0);
	const amount = withinBand(usage, 0, 100) * 0.03  // here
				+ withinBand(usage, 100, 200) * 0.05
				+ topBand(usage) * 0.07;
	return usd(amount);
}
```

1. 대역의 상한을 호출하는 부분도 새로 만든 매개변수와 함수를 호출하도록 하는데 최대값은 Infinity 를 사용한다.

```jsx
function baseCharge(usage) {
	if(usage < 0) return usd(0);
	const amount = withinBand(usage, 0, 100) * 0.03  
				+ withinBand(usage, 100, 200) * 0.05
				+ withinBand(usage, 200, Infinity) * 0.07; // here
	return usd(amount);
}
```

1. 기존에 있던 bottomBand(), topBand() 함수는 제거한다.

## 11.3 플래그 인수 제거하기 Remove Flag Argument

```jsx
// ASIS
function setDimension(name, value) {
	if(name === "height") {
		this._height = value;
		return;
	}
	if(name === "width") {
		this._width = value;
		return;
	}
}

// TOBE
function setHeight(value) { this._height = value; }
function setWidth(value) { this._width = value; }
```

### 배경

플래그 인수 flag argument 란 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수다.

```jsx
function bookConcert(aCustomer, isPremium) {
	if(isPremium) {
		// 프리미엄 예약용 로직
	} else {
		// 일반 예약용 로직
	}
}

// 클라이언트
bookConcert(aCustomer, true);
```

플래그 인수가 있으면 함수들의 기능 차이가 잘 드러나지 않는다.

사용할 함수를 선택하고도 플래그 인수로 어떤 값을 넘겨야 하는지 또 알아내야 한다.

플래그 인수라고 불릴려면 호출하는 쪽에서 불리언 값으로 리터럴 값을 전달하고, 또한 호출되는 함수는 그 인수를 제어 흐름을 결정하는데 사용해야 해야 한다.

### 절차

1. 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성한다.
2. 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정한다.

### 예시

배송일자를 계산하는 호출을 보자

```jsx
// 클라이언트
aShipment.deliveryDate = deliveryDate(anOrder, true);
```

매개변수로 전달되는 불리언 값은 무엇을 의미하는가?

```jsx
function deliveryDate(anOrder, isRush) {
	if(isRush) {
		let deliveryTime;
		if(["MA", "CT"].includes(anOrder.deliveryState)) deliveryTime = 1;
		else if(["NY", "NH"].includes(anOrder.deliveryState)) deliveryTime = 2;
		else deliveryTime = 3;
		return anOrder.placeOn.plusDays(1 + deliveryTime);
	} else {
		let deliveryTime;
		if(["MA", "CT", "NY"].includes(anOrder.deliveryState)) deliveryTime = 2;
		else if(["ME", "NH"].includes(anOrder.deliveryState)) deliveryTime = 3;
		else deliveryTime = 4;
		return anOrder.placeOn.plusDays(2 + deliveryTime);
	}
}
```

호출하는 쪽에서 불리언 리터럴 값을 이용해서 어느 쪽 코드를 실행할지 결정하는 전형적인 플래그 인수이다.

1. 조건문을 분해한다

```jsx
function deliveryDate(anOrder, isRush) {
	if(isRush) return rushDeliveryDate(anOrder);
	else return regularDeliveryDate(anOrder);
}

function rushDeliveryDate(anOrder) {
	let deliveryTime;
	if(["MA", "CT"].includes(anOrder.deliveryState)) deliveryTime = 1;
	else if(["NY", "NH"].includes(anOrder.deliveryState)) deliveryTime = 2;
	else deliveryTime = 3;
	return anOrder.placeOn.plusDays(1 + deliveryTime);
}

function regularDeliveryDate(anOrder) {
	let deliveryTime;
	if(["MA", "CT", "NY"].includes(anOrder.deliveryState)) deliveryTime = 2;
	else if(["ME", "NH"].includes(anOrder.deliveryState)) deliveryTime = 3;
	else deliveryTime = 4;
	return anOrder.placeOn.plusDays(2 + deliveryTime);
}
```

1. 호출을 의도에 맞게 대체한다.

```jsx
// 클라이언트
aShipment.deliveryDate = rushDeliveryDate(anOrder);
```

### 예시: 매개변수를 까다로운 방식으로 사용할 때

매개변수가 함수 핵심 로직의 내부에 있으면 어떻게 해야할까?

```jsx
function deliveryDate(anOrder, isRush) {
	let result;
	let deliveryTime;
	if(anOrder.deliveryState === "MA" || anOrder.deliveryState === "CT")
		deliveryTime = isRush? 1 : 2;
	else if(anOrder.deliveryState === "NY" || anOrder.deliveryState === "NH") {
		deliveryTime = 2;
		if(anOrder.deliveryState === "NH" && !isRush)
			deliveryTime = 3;
	} 
	else if(isRush)
		deliveryTime = 3;
	else if(anOrder.deliveryState === "ME")
		deliveryTime = 3;
	else 
		delivertTiem = 4;
	result = anOrder.placeOn.plusDays(2 + deliveryTime);
	if(isRush) result = result.minusDays(1);
	return result;
}
```

위 코드에서 isRush 를 최상위 조건으로 뽑아내려면 생각보다 일이 커질 수도 있다.

이럴 때는 기존 deliveryDate() 는 놔두고, deliveryDate() 를 감싸는 래핑 함수를 생각해볼 수 있다.

```jsx
function rushDeliveryDate(anOrder) {
	return deliveryDate(anOrder, true);
}

function regularDeliveryDate(anOrder) {
	return deliveryDate(anOrder, false);
}
```

## 11.4 객체 통째로 넘기기 Preserve Whole Object

```jsx
// ASIS
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if(aPlan.withinRange(low, high))

// TOBE
if(aPlan.withinRange(aRoom.daysTempRange);
```

### 배경

레코드를 통째로 넘기면 변화에 대응하기 쉽다.

매개변수 목록이 짧아져서 함수 사용법을 이해하기 쉬워진다.

함수가 레코드 자체에 의존하기를 원치않을 때는 이 리팩터링을 수행하지 않는다.

어떤 객체로부터 값 몇 개를 얻은 후 그 값들만으로 무언가를 하는 로직이 있다면, 그 로직을 객체 안으로 집어넣어야 함을 알려주는 악취로 봐야 한다.

한 객체가 제공하는 기능 중 항상 똑같은 일부만을 사용하는 코드가 많다면, 그 기능만 따로 묶어서 클래스로 추출하라는 신호일 수 있다.

다른 객체의 메서드를 호출하면서 호출하는 객체 자신이 가지고 있는 데이터 여러 개를 건네는 경우이다.

이런 상황이면 데이터 여러 개 대신 객체 자신의 참조만 건네도록 수정할 수 있다.

### 절차

1. 매개변수들을 원하는 형태로 받는 빈 함수로 만든다.
2. 새 함수의 본문에서는 원래 함수를 호출하도록 하며, 새 매개변수와 원래 함수의 매개변수를 매핑한다.
3. 정적 검사를 수행한다.
4. 모든 호출자가 새 함수를 사용하게 수정한다. 하나씩 수정하며 테스트하자.
5. 호출자를 모두 수정했다면 원래 함수를 인라인한다.
6. 새 함수의 이름을 적절히 수정하고 모든 호출자에 반영한다.

### 예시

일일 최저, 최고 기온이 난방 계획에서 정한 범위를 벗어나는지 확인하는 실내온도 모니터링 시스템이 있다.

```jsx
// 호출자
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if(!aPlan.withinRange(low, high))
	alerts.push("방 온도가 지정 범위를 벗어났습니다.");

// HeatingPlan 클래스
withinRange(bottom, top) {
	return (bottom >= this._temperatureRange.low)
		&& (top <=this._temperatureRange.high);
}
```

1. 가장 먼저 원하는 인터페이스를 갖춘 빈 메서드를 만든다.

```jsx
// HeatingPlan 클래스
xxNewWithinRange(aNumberRange) {
}
```

이 메서드로 기존 withinRange() 메서드를 대체할 예정이기 때문에 똑같은 이름을 쓰되, 나중에 찾아 바꾸기 쉽도록 적당한 접두어만 붙여둔다.

1. 그런 다음 새 메서드의 본문은 기존 withinRange() 를 호출하는 코드로 채운다. 자연스럽게 새 매개변수를 기존 매개변수와 매핑하는 로직이 만들어진다.

```jsx
xxNewWithinRange(aNumberRange) {
	return this.withinRange(aNumberRange.low, aNumberRange.high);
}
```

1. 기존 함수를 호출하는 코드를 찾아서 새 함수를 호출하게 수정하자

```jsx
// 호출자
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if(!aPlan.xxNewWithinRange(aRoom.daysTempRange))
	alerts.push("방 온도가 지정 범위를 벗어났습니다.");
```

1. 기존 코드중에 죽은 코드는 제거한다.

```jsx
// 호출자
// const low = aRoom.daysTempRange.low;
// const high = aRoom.daysTempRange.high;
if(!aPlan.xxNewWithinRange(aRoom.daysTempRange))
	alerts.push("방 온도가 지정 범위를 벗어났습니다.");
```

1. 모두 새 함수로 대체했다면 원래 함수를 인라인한다.

```jsx
xxNewWithinRange(aNumberRange) {
	return (aNumberRange.low >= this._temperatureRange.low) &&
		(aNumberRange.high <= this._temperatureRange.high);
}
```

1. 새 함수에서 보기 흉한 접두어를 제거하고 호출자들에게도 모두 반영한다.

```jsx
withinRange(aNumberRange) {
	return (aNumberRange.low >= this._temperatureRange.low) &&
		(aNumberRange.high <= this._temperatureRange.high);
}

// 호출자
if(!aPlan.withinRange(aRoom.daysTempRange))
	alerts.push("방 온도가 지정 범위를 벗어났습니다.");
```

### 예시: 새 함수를 다른 방식으로 만들기

```jsx
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if(!aPlan.xxNewWithinRange(aRoom.daysTempRange))
	alerts.push("방 온도가 지정 범위를 벗어났습니다.");
```

코드를 재정렬해서 기존 코드 일부를 메서드로 추출하는 방식으로 새 메서드를 만들겠다.

1. 입력 매개변수를 추출한다.

```jsx
// 호출자
const tempRange = aRoom.daysTempRange; // here
const low = tempRange.low;
const high = tempRange.high;
const isWithinRange = aPlan.withinRange(low, high);
if(!isWithinRange)
	alerts.push("방 온도가 지정 범위를 벗어났습니다.");
```

1. 추출한 매개변수를 새로운 함수에 전달한다.

```jsx
// 호출자
const tempRange = aRoom.daysTempRange;
const isWithinRange = xxNewWithinRange(aPlan, tempRange);
if(!isWithinRange)
	alerts.push("방 온도가 지정 범위를 벗어났습니다.");

// 최상위
function xxNewWithinRange(aPlan, tempRange) {
	const low = tempRange.low;
	const high = tempRange.high;
	const isWithinRange = aPlan.withinRange(low, high);
	return isWithinRange;
}
```

호출자들을 수정한 다음 옛 메서드를 새 메서드 안으로 인라인 한다.

추출한 변수들도 인라인해주면 새로 추출한 메서드를 깔끔하게 분리할 수 있다.

```jsx
// 호출자
const tempRange = aRoom.daysTempRange;
// const low = tempRange.low;
// const high = tempRange.high;
const isWithinRange = aPlan.xxNewWithinRange(aPlan, tempRange);
if(!isWithinRange)
	alerts.push("방 온도가 지정 범위를 벗어났습니다.");

// HeatingPlan 클래스
xxNewWithinRange(aPlan, tempRange) {
	const low = tempRange.low;
	const high = tempRange.high;
	const isWithinRange = aPlan.withinRange(low, high);
	return isWithinRange;
}
```

## 11.5 매개변수를 질의 함수로 바꾸기 Replace Parameter with Query

```jsx
// ASIS
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
	...
}

//TOBE
availableVacation(anEmployee);

function availableVacation(anEmployee) {
	const grade = anEmployee.grade;
}
```

### 배경

매개변수 목록은 함수의 변동 요인을 모아놓은 곳이다.

피호출 함수가 스스로 결정할 수 있는 값을 매개변수로 건네는 것도 일종의 중복이다.

매개변수가 있다면 결정 주체가 호출자가 되고, 매개변수가 없다면 피호출 함수가 된다.

저자는 호출하는 쪽을 간소하게 만드는 것을 선호한다.

피호출 함수가 그 역할을 수행하기 적합할 때는 책임 소재를 피호출 함수로 옮긴다.

하지만 매개변수를 제거해서 피호출함수에 원치 않는 의존성이 생기면 매개변수를 질의함수로 바꾸지 말아야 한다.

제거하려는 매개변수의 값을 다른 함수에 질의해서 얻을 수 있다면 안심하고 질의 함수로 바꿀 수 있다.

질의해서 얻고자 하는 함수는 참조 투명 referential transparency 해야 한다.

참조 투명이란 ‘함수에 똑같은 값을 건네 호출하면 항상 똑같이 동작한다'는 뜻이다.

### 절차

1. 필요하다면 대상 매개변수의 값을 계산하는 코드를 별도 함수로 추출해 놓는다.
2. 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다. 하나 수정할 때마다 테스트한다.
3. 함수 선언 바꾸기로 대상 매개변수를 없앤다.

### 예시

```jsx
// Order 클래스
get finalPrice() {
	const basePrice = this.quantity * this.itemPrice;
	let discountLevel;
	if (this.quantity > 100) discountLevel = 2;
	else discountLevel = 1;
	return this.discountedPrice(basePrice, discountLevel);
}

discountedPrice(basePrice, discountLevel) {
  switch (discountLevel) {
    case 1: return basePrice * 0.95;
    case 2: return basePrice * 0.9;
  }
}
```

임시 변수를 질의 함수로 바꾸기를 finalPrice 에 적용한다.

```jsx
// Order 클래스
get finalPrice() {
	const basePrice = this.quantity * this.itemPrice;
	// let discountLevel;
	// if (this.quantity > 100) discountLevel = 2;
	// else discountLevel = 1;
	return this.discountedPrice(basePrice, this.discountLevel); // here
}

get discountLevel() { // here
  return (this.quantity > 100) ? 2 : 1;
}

discountedPrice(basePrice, discountLevel) {
  switch (discountLevel) {
    case 1: return basePrice * 0.95;
    case 2: return basePrice * 0.9;
  }
}
```

1. 매개변수를 참조하는 코드를 모두 함수 호출로 바꾼다.

```jsx
// Order 클래스
discountedPrice(basePrice, discountLevel) {
  switch(this.discountLevel) { // here
    case 1: return basePrice * 0.95;
    case 2: return basePrice * 0.9;
  }
}
```

1. 함수 선언 바꾸기로 매개변수를 없앤다.

```jsx
// Order 클래스
get finalPrice() {
  const basePrice = this.quantity * this.itemPrice;
  return this.discountedPrice(basePrice);
}

get discountLevel() {
  return (this.quantity > 100) ? 2 : 1;
}

discountedPrice(basePrice) {
  switch(this.discountLevel) {
    case 1: return basePrice * 0.95;
    case 2: return basePrice * 0.9;
  }
}
```

## 11.6 질의 함수를 매개변수로 바꾸기 Replace Query with Parameter

```jsx
// ASIS
targetTemperature(aPlan)

function targetTemperature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
}

// TOBE
targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
  
}
```

### 배경

전역 변수를 참조한다거나 제거하길 원하는 변수를 참조하는 경우가 있다.

이럴 경우 해당 참조를 매개변수로 바꿔 해결할 수 있다.

책임을 호출자로 옮기는 것이다.

특정 변수를 매개변수로 바꾸어 매개변수의 양을 늘리거나, 의존을 높이거나 하는 것은 프로그램을 더 잘 이해하게 되었을 때 개선하기 쉽게 설계해두는게 중요하다.

똑같은 값을 건네면 똑같은 결과를 내는 함수는 다루기 쉽다.

이런 성질을 ‘참조 투명성'이라 한다.

모듈을 개발할 때 순수 함수들을 따로 구분하고, 프로그램의 입출력과 기타 변수들을 다루는 로직으로 순수 함수를 감싸는 패턴을 많이 활용한다.

이 리팩터링을 활용하면 프로그램의 일부를 순수 함수로 바꿀 수 있다.

질의 함수를 매개변수로 바꾸면 호출자가 어떤 값을 제공해야하는지 알아야 한다.

### 절차

1. 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리한다.
2. 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출한다.
3. 방금 만든 변수를 인라인하여 제거한다.
4. 원래 함수도 인라인한다.
5. 새 함수의 이름을 원래 함수의 이름으로 고쳐준다.

### 예시

실내온도 제어 시스템이다. 사용자는 온도조절기 thermostat 로 온도를 설정하지만 목표 온도는 정해진 범위에서만 선택할 수 있다.

```jsx
// HeatingPlan 클래스
get targetTemperature() {
  if(thermostat.selectedTemperature > this._max) return this._max;
  else if(thermostat.selectedTemperature < this._min) return this._min;
  else return thermostat.selectedTemperature;
}

// 클라이언트
if (thePlan.targetTemperature > thermostat.currentTemperature) setToHeat();
else if (thePlan.targetTemperature < thermostat.currentTemperature) setToCool();
else setOff();
```

1. 변수 추출하기를 이용하여 메서드에서 사용할 매개변수 selectedTemperature 를 준비한다.

```jsx
// HeatingPlan 클래스
get targetTemperature() {
  const selectedTemperature = thermostat.selectedTemperature;
  if(selectedTemperature > this._max) return this._max;
  else if(selectedTemperature < this._min) return this._min;
  else return selectedTemperature;
}
```

1. 매개변수의 값을 구하는 코드 외에 나머지를 메서드로 추출한다.

```jsx
// HeatingPlan 클래스
get targetTemperature() {
  const selectedTemperature = thermostat.selectedTemperature;
  return this.xxNEWtargetTemperature(selectedTemperature); // here
}

xxNEWtargetTemperature(selectedTemperature) {
  if(selectedTemperature > this._max) return this._max;
  else if(selectedTemperature < this._min) return this._min;
  else return selectedTemperature;
}
```

1. 방금 추출한 매개변수 selectedTemperature 를 인라인하여 원래 메서드에는 단순한 호출만 남게 한다.

```jsx
// HeatingPlan 클래스
get targetTemperature() {
  return this.xxNEWtargetTemperature(thermostat.selectedTemperature);
}
```

1. 클라이언트에서 새로 만든 메서드를 인라인한다.

```jsx
// 클라이언트
if (thePlan.xxNewtargetTemperature(thermostat.selectedTemperature) > thermostat.currentTemperature) setToHeat();
else if (thePlan.xxNewtargetTemperature(targetTemperature.selectedTemperature) < thermostat.currentTemperature) setToCool();
else setOff();
```

1. 메서드의 이름을 원래로 변경한다.

```jsx
// HeatingPlan 클래스
targetTemperature(selectedTemperature) {
  if(selectedTemperature > this._max) return this._max;
  else if(selectedTemperature < this._min) return this._min;
  else return selectedTemperature;
}

// 클라이언트
if (thePlan.targetTemperature(thermostat.selectedTemperature) > thermostat.currentTemperature) setToHeat();
else if (thePlan.targetTemperature(targetTemperature.selectedTemperature) < thermostat.currentTemperature) setToCool();
else setOff();
```

이 리팩터링으로 인해 HeatingPlan 클래스는 불변이 되었다.

모든 필드가 생성자에서 설정되며, 필드를 변경할 수 있는 메서드는 없다.

targetTemperature() 메소드도 참조 투명하게 되었다.

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}