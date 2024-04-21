---
layout: single
title: 리팩터링2 스터디 - 012 - 2
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 12장 상속 다루기

## 12.8 슈퍼클래스 추출하기

### 배경

비슷한 일을 수행하는 두 클래스가 보이면 상속 메커니즘을 이용해서 비슷한 부분을 공통의 슈퍼클래스로 옮겨 담을 수 있다.

상속은 프로그램이 성장하면서 깨우쳐가게 되며, 슈퍼클래스로 끌어올리고 싶은 공통 요소를 찾았을 때 수행하는 사례가 잦다.

### 절차

1. 빈 슈퍼클래스를 만든다. 원래의 클래스들이 새 클래스를 상속하도록 한다.
  1. 필요하다면 생성자에 함수 선언 바꾸기를 적용한다.
2. 테스트한다
3. 생성자 본문 올리기, 메서드 올리기, 필드 올리기를 차례로 적용하여 공통 원소를 슈퍼클래스로 옮긴다.
4. 서브클래스에 남은 메서드들을 검토한다. 공통되는 부분이 있다면 함수로 추출한 다음 메서드 올리기를 적용한다.
5. 원래 클래스들을 사용하는 코드를 검토하여 슈퍼클래의 인터페이스를 사용하게 할지 고민해본다.

### 예시

```java
// ASIS
class Department {
    get totalAnnualCost() {}
    get name() {}
    get headCount() {}
}

class Employee {
    get annualCost() {}
    get name() {}
    get id() {}
}

// TOBE
class Party {
    get name() {}
    get annualCost() {}
}

class Department extends Party {
    get annualCast() {}
    get headCount() {}
}

class Employee extends Party {
    get annualCast() {}
    get id() {}
}

// 예시
// ASIS
class Employee {
    constructor(name, id, monthlyCost) {
        this._id = id
        this._name = name
        this._monthlyCost = monthlyCost
    }
    get monthlyCost() { return this._monthlyCost; } // 월간 비용
    get name() { return this._name; } // 이름
    get id() { return this._id; } 

    get annualCost() {  // 연간 비용
        return this.monthlyCost * 12
    }
}

class Department {
    constructor(name, staff) {
        this._name = name
        this._staff = staff
    }
    get staff() { return this._staff; }
    get name() { return this._name; } // 이름

    get totalMonthlyCost() { // 총 월간 비용
        return this.staff
        .map(e => e.monthlyCost)
        .reduce((sum, cost) => sum + cost)
    }
    get headCount() {
        return this.staff.length
    }
    get totalAnnualCost() { // 총 연간 비용
        return this.totalMonthlyCost * 12
    }
}

// TOBE
// Step 1 빈 슈퍼클래스를 만들고 두 클래스가 확장하도록 만든다.
class Party {}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        this._id = id
        this._name = name
        this._monthlyCost = monthlyCost
    }
    get monthlyCost() { return this._monthlyCost; } // 월간 비용
    get name() { return this._name; } // 이름
    get id() { return this._id; } 

    get annualCost() {  // 연간 비용
        return this.monthlyCost * 12
    }
}

class Department extends Party {
    constructor(name, staff) {
        this._name = name
        this._staff = staff
    }
    get staff() { return this._staff; }
    get name() { return this._name; } // 이름

    get totalMonthlyCost() { // 총 월간 비용
        return this.staff
        .map(e => e.monthlyCost)
        .reduce((sum, cost) => sum + cost)
    }
    get headCount() {
        return this.staff.length
    }
    get totalAnnualCost() { // 총 연간 비용
        return this.totalMonthlyCost * 12
    }
}

// Step 2 데이터부터 변경 생성자를 만져 이름 속성 올리기
class Party {
    constructor(name) {
        this._name = name
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super(name)
        this._id = id
        this._monthlyCost = monthlyCost
    }
    get monthlyCost() { return this._monthlyCost; } // 월간 비용
    get name() { return this._name; } // 이름
    get id() { return this._id; } 

    get annualCost() {  // 연간 비용
        return this.monthlyCost * 12
    }
}

class Department extends Party {
    constructor(name, staff) {
        super(name)
        this._staff = staff
    }
    get staff() { return this._staff; }
    get name() { return this._name; } // 이름

    get totalMonthlyCost() { // 총 월간 비용
        return this.staff
        .map(e => e.monthlyCost)
        .reduce((sum, cost) => sum + cost)
    }
    get headCount() {
        return this.staff.length
    }
    get totalAnnualCost() { // 총 연간 비용
        return this.totalMonthlyCost * 12
    }
}
// Step 3 옮긴 데이터와 관련된 메서들을 옮긴다

class Party {
    constructor(name) {
        this._name = name
    }
    get name() {
        return this._name
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super(name)
        this._id = id
        this._monthlyCost = monthlyCost
    }
    get monthlyCost() { return this._monthlyCost; } // 월간 비용
    get id() { return this._id; } 

    get annualCost() {  // 연간 비용
        return this.monthlyCost * 12
    }
}

class Department extends Party {
    constructor(name, staff) {
        super(name)
        this._staff = staff
    }
    get staff() { return this._staff; }

    get totalMonthlyCost() { // 총 월간 비용
        return this.staff
        .map(e => e.monthlyCost)
        .reduce((sum, cost) => sum + cost)
    }
    get headCount() {
        return this.staff.length
    }
    get totalAnnualCost() { // 총 연간 비용
        return this.totalMonthlyCost * 12
    }
}

// Step 4 동일한 의도를 가진 annualCost, totalAnnualCost를 함수 선언 바꾸기로 이름을 통일한다.
// 이름을 동일하게 맞추기 
class Party {
    constructor(name) {
        this._name = name
    }
    get name() {
        return this._name
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super(name)
        this._id = id
        this._monthlyCost = monthlyCost
    }
    get monthlyCost() { return this._monthlyCost; } // 월간 비용
    get id() { return this._id; } 

    get annualCost() {  // 연간 비용
        return this.monthlyCost * 12
    }
}

class Department extends Party {
    constructor(name, staff) {
        super(name)
        this._staff = staff
    }
    get staff() { return this._staff; }

    get monthlyCost() { // 총 월간 비용, 이름 변경
        return this.staff
        .map(e => e.monthlyCost)
        .reduce((sum, cost) => sum + cost)
    }
    get headCount() {
        return this.staff.length
    }
    get annualCast() { // 총 연간 비용, 이름 통일 
        return this.monthlyCost * 12
    }
}

// Step 4-2 동일한 함수 올리기
class Party {
    constructor(name) {
        this._name = name
    }
    get name() {
        return this._name
    }

    get annualCast() { // 총 연간 비용, 이름 통일 
        return this.monthlyCost * 12
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super(name)
        this._id = id
        this._monthlyCost = monthlyCost
    }
    get monthlyCost() { return this._monthlyCost; } // 월간 비용
    get id() { return this._id; } 
}

class Department extends Party {
    constructor(name, staff) {
        super(name)
        this._staff = staff
    }
    get staff() { return this._staff; }

    get monthlyCost() { // 총 월간 비용, 이름 변경
        return this.staff
        .map(e => e.monthlyCost)
        .reduce((sum, cost) => sum + cost)
    }
    get headCount() {
        return this.staff.length
    }
}
```

## 12.9 계층 합치기

### 배경

어떤 클래스와 그 부모가 너무 비슷해져서 더는 독립적으로 존재해야 할 이유가 사라지는 경우 둘을 합쳐야 한다.

### 절차

1. 두 클래스 중 제거할 것을 고른다.
  1. 미래를 생각하여 더 적합한 이름의 클래스를 남기자, 둘 다 적절치 않다면 임의로 하나를 고른다.
2. 필드 올리기와 메서드 올리기 혹은 필드 내리기와 메서드 내리기를 적용하여 모든 요소를 하나의 클래스로 옮긴다.
3. 제거할 클래스를 참조하던 모든 코드가 남겨질 클래스를 참조하도록 고친다.
4. 빈 클래스를 제거한다.
5. 테스트한다.

### 예시

```java
// ASIS
class Employee {}

class Salesperson extends Employee {}

// TOBE
class Employee {}
```

## 12.10 서브클래스를 위임으로 바꾸기

### 배경

속한 갈래에 따라 동작이 달라지는 객체들은 상속으로 표현하는 게 자연스럽다.

공통 데이터와 동작은 모두 슈퍼클래스에 두고 서브클래스에 맞게 기능을 추가하거나 오버라이드하면 된다.

상속에는 단점이 있다. 가장 명확한 단점은 한 번만 쓸 수 있는 카드라는 것이다.

무언가가 달라져야 하는 이유가 여러 개여도 상속에서는 그중 단 하나의 이유만 선택해 기준으로 삼을 수밖에 없다.

예컨데 사람 객체의 동작을 ‘나이대'와 ‘소득 수준'에 따라 달리 하고 싶다면 서브클래스는 젊은이와 어르신이 되거나, 부자와 서민이 된다. 둘 다는 안 된다.

또 다른 문제로, 상속은 클래스들의 관계를 아주 긴말하게 결합한다. 부모를 수정하면 이미 존재하는 자식들의 기능을 해치기가 쉽다.

위임은 위 두 문제를 모두 해결해준다.

위임은 객체 사이의 일반적인 관계이므로 상호작용에 필요한 인터페이스를 명확히 정의할 수 있다. 즉 상속보다 결합도가 훨씬 약하다.

유명한 원칙이 있다. “(클래스) 상속보다는 (객체) 컴포지션을 사용하라"

여기서 컴포지션(Composition)은 사실상 위임과 같은 말이다.

상속은 필요할 때 언제든지 위임으로 변경이 가능하기 떄문에 처음에 접근은 상속으로 하고 문제가 생기면 위임으로 갈아 타면 된다.

‘디자인 패턴'에 익숙한 사람이라면 이 리팩터링을 서브클래스를 상태 패턴(State Pattern)이나 전략 패턴(Strategy Pattern)으로 대체한다고 생각하면 된다.

### 절차

1. 생성자를 호출하는 곳이 많다면 생성자 팩터리 함수로 바꾼다.
2. 위임으로 활용할 빈 클래스를 만든다. 이 클래스의 생성자는 서브클래스에 특화된 데이터를 전부 받아야만 하며, 보통은 슈퍼클래스를 가르키는 역참조(back-reference)도 필요하다.
3. 위임을 저장할 필드를 슈퍼클래스에 추가한다.
4. 서브클래스 생성 코드를 수정하여 위임 인스턴스를 생성하고 위임 필드에 대입해 초기화한다.
  1. 이 작업은 팩터리 함수가 수행한다. 혹은 생성자가 정확한 위임 인스턴스를 생성할 수 있는 게 확실하다면 생성자에서 수행할 수도 있다.
5. 서브클래스의 메서드 중 위임 클래스로 이동할 것을 고른다.
6. 함수 옮기기를 적용해 위임 클래스로 옮긴다. 원래 메서드에서 위임하는 코드는 지우지 않는다.
  1. 이 메서드가 사용하는 원소 중 위임으로 옮겨야 하는 게 있다면 함께 옮긴다. 슈퍼클래스에 유지해야 할 원소를 참조한다면 슈퍼클래스를 참조하는 필드를 위임에 추가한다.
7. 서브클래스 외부에도 원래 메서드를 호출하는 코드가 있다면 서브클래스의 위임 코드를 슈퍼클래스로 옮긴다. 이때 위임이 존재하는지를 검사하는 보호 코드로 감싸야 한다. 호출하는 외부 코드가 없다면 원래 메서드는 죽은 코드가 되므로 제거한다.
  1. 서브클래스가 둘 이상이고 서브클래스에서 중복이 생겨나기 시작했다면 슈퍼클래스를 추출한다. 이렇게 하여 기본 동작이 위임 슈퍼클래스로 옮겨졌다면 슈퍼클래스의 위임 메서드들에는 보호 코드가 필요 없다.
8. 테스트한다.
9. 서브클래스의 모든 메서드가 옮겨질 때까지 5~8과정을 반복한다.
10. 서브클래스들의 생성자를 호출하는 코드를 찾아서 슈퍼클래스의 생성자를 사용하도록 수정한다.
11. 테스트한다.
12. 서브클래스를 삭제한다(죽은 코드 제거하기)

### 예시 1

```java
// 예시 1 공연 예약 클래스
// ASIS
class Booking {
    constructor(show, date) {
        this._show = show
        this._date = date 
    }
    get hasTalkback() {
        return this._show.hasOwnProperty('talkback') && !this.isPeakDay;
    }
    get basePrice() {
        let result = this._show.price;
        if (this.isPeakDay) result += Math.round(result * 0.15);
        return result;
    }
}
// 추가 비용을 다양하게 설정할 수 있는 프리미엄 예약용 서브클래스
class PremiumBooking extends Booking {
    constructor(show, date, extras) {
        super(show, date)
        this._extras = extras
    }
    get hasTalkback() { // 오버라이드
        return this._show.hasOwnProperty('talkback')
    }
    get basePrice() { // 오버라이드
        return Math.round(super.basePrice + this._extras.premiumFee)
    }
    get hasDinner() {
        return this._extras.hasOwnProperty('dinner') && !this.isPeakDay
    }
}

// 클라이언트(일반 예약)
aBooking = new Booking(show, date)

// 클라이언트(프리미엄 예약)
aBooking = new PremiumBooking(show, date, extras);

// TOBE
// Step 1 먼저 생성자를 팩터리 함수로 바꿔서 생성자 호출 부분을 캠슐화하자

function createBooking(show, date) {
    return new Booking(show, date)
}

function createPremiumBooking(show, date, extras) {
    return new PremiumBooking(show, date, extras)
}

// 클라이언트(일반 예약)
aBooking = createBooking(show, date)
// 클라이언트(프리미엄 예약)
aBooking = createPremiumBooking(show, date, extras)

// Step 2 위임 클래스를 만든다. 위임 클래스의 생성자는 서브클래스가 사용하던 매개변수와 예약 객체로의 역참조(back-reference)를 매개변수로 받는다.
// 역참조가 필요한 이유는 서브클래스 메서드 중 슈퍼클래스에 저장된 데이터를 사용하는 경우가 있기 때문이다.
class PremiumBookingDelegate {
    constructor(hostBooking, extras) {
        this._host = hostBooking
        this._extras = extras
    }
}

// Step 3 새로운 위임을 예약 객체와 연결한다. 프리미엄 예약을 생성하는 팩터리 함수를 수정하면 된다.
function createPremiumBooking(show, date, extras) {
    const result = new PremiumBooking(show, date, extras)
    result._bePremium(extras)
    return result
}

class Booking {
    constructor(show, date) {
        this._show = show
        this._date = date 
    }
    get hasTalkback() {
        return this._show.hasOwnProperty('talkback') && !this.isPeakDay;
    }
    get basePrice() {
        let result = this._show.price;
        if (this.isPeakDay) result += Math.round(result * 0.15);
        return result;
    }
    _bePremium(extras) {
        this._premiumDelegate = new PremiumBookingDelegate(this, extra)
    }
}
// _bePremium() 메서드 이름 앞에 밑줄을 붙여 이 메서드가 Booking의 공개 인터페이스가 되어서는 안된다는 의도를 밝힌다.
// 리팩터링의 목적이 일반 예약과 프리미엄 예약을 상호 변환할 수 있게 하는 것이라면 이 메서드는 public 되어도 된다.

// Step 4 기능 옮기기 함수 옮기기를 적용해 서브클래스의 메서드를 위임으로 옮긴다.
// 슈퍼클래스의 데이터를 사용하는 부분은 모두 _host를 통하도록 고친다.
class PremiumBookingDelegate {
    constructor(hostBooking, extras) {
        this._host = hostBooking
        this._extras = extras
    }
    get hasTalkback() { // 위임으로 서브클래스의 함수 옮기기
        return this._host._show.hasOwnProperty('talkback')
    }
}

class PremiumBooking extends Booking {
    constructor(show, date, extras) {
        super(show, date)
        this._extras = extras
    }
    get hasTalkback() { // PremiumDelegate.hasTalkback으롤 사용하도록 변경
        return this._premiumDelegate.hasTalkback;  
    }
    get basePrice() { 
        return Math.round(super.basePrice + this._extras.premiumFee)
    }
    get hasDinner() {
        return this._extras.hasOwnProperty('dinner') && !this.isPeakDay
    }
}

// Step 5 테스트 후 서브클래스의 메서드 삭제
class PremiumBooking extends Booking {
    constructor(show, date, extras) {
        super(show, date)
        this._extras = extras
    }
    // hasTalkback 함수 삭제
    get basePrice() { 
        return Math.round(super.basePrice + this._extras.premiumFee)
    }
    get hasDinner() {
        return this._extras.hasOwnProperty('dinner') && !this.isPeakDay
    }
}
// Step 6 위임이 존재하면 위임을 사용하는 분배 로직을 슈퍼클래스 메서드에 추가
class Booking {
    constructor(show, date) {
        this._show = show
        this._date = date 
    }
    get hasTalkback() { // 위임을 사용하는 분배 로직을 슈퍼클래스 메서드에 추가
        return (this._premiumDelegate)
        ? this._premiumDelegate.hasTalkback
        : this._show.hasOwnProperty('talkback') && !this.isPeakDay
    }
    get basePrice() {
        let result = this._show.price;
        if (this.isPeakDay) result += Math.round(result * 0.15);
        return result;
    }
    _bePremium(extras) {
        this._premiumDelegate = new PremiumBookingDelegate(this, extra)
    }
}

// Step 7 basePrice 함수 옮기기
// super을 호출하는 성가신 부분이 존재, 단순히 this._host._basePrice라고 사용하면 무한 재귀에 빠지고 만다.
// 방법 1. 슈퍼클래스의 계산 로직을 함수로 추출하여 가격 계산과 분배 로직을 분리
class Booking {
    constructor(show, date) {
        this._show = show
        this._date = date 
    }
    get hasTalkback() { // 위임을 사용하는 분배 로직을 슈퍼클래스 메서드에 추가
        return (this._premiumDelegate)
        ? this._premiumDelegate.hasTalkback
        : this._show.hasOwnProperty('talkback') && !this.isPeakDay
    }
    get basePrice() {
        return (this._premiumDelegate)
        ? this._premiumDelegate.basePrice
        : this._privateBasePrice
    }

    get _privateBasePrice() {
        let result = this._show.price;
        if (this.isPeakDay) result += Math.round(result * 0.15);
        return result;
    }

    _bePremium(extras) {
        this._premiumDelegate = new PremiumBookingDelegate(this, extra)
    }
}
// 방법 2 위임의 메서드를 기반 메서드의 확장 형태로 재호출
class Booking {
    constructor(show, date) {
        this._show = show
        this._date = date 
    }
    get hasTalkback() { // 위임을 사용하는 분배 로직을 슈퍼클래스 메서드에 추가
        return (this._premiumDelegate)
        ? this._premiumDelegate.hasTalkback
        : this._show.hasOwnProperty('talkback') && !this.isPeakDay
    }
    get basePrice() {
        let result = this._show.price;
        if (this.isPeakDay) result += Math.round(result * 0.15);
        return (this._premiumDelegate)
        ? this._premiumDelegate.extendBasePrice(result)
        : result;
    }
    _bePremium(extras) {
        this._premiumDelegate = new PremiumBookingDelegate(this, extra)
    }
		
		get hasDinner() {
		        return (this._premiumDelegate)
		        ? this._premiumDelegate.hasDinner
		        : undefined
		    }
}

class PremiumBookingDelegate {
    constructor(hostBooking, extras) {
        this._host = hostBooking
        this._extras = extras
    }
    get hasTalkback() { // 위임으로 서브클래스의 함수 옮기기
        return this._host._show.hasOwnProperty('talkback')
    }
    extendBasePrice(base) { // 확장 메서드
        return Math.round(base + this._extras.premiumFee)
    }
}

// Step 8 서브클래스에만 존재하는 메서드 처리
class PremiumBooking extends Booking {
    constructor(show, date, extras) {
        super(show, date)
        this._extras = extras
    }
    get basePrice() { 
        return Math.round(super.basePrice + this._extras.premiumFee)
    }
    get hasDinner() { // 위임으로 옮기기 
        return this._extras.hasOwnProperty('dinner') && !this.isPeakDay
    }
}

class PremiumBookingDelegate {
    constructor(hostBooking, extras) {
        this._host = hostBooking
        this._extras = extras
    }
    get hasTalkback() { // 위임으로 서브클래스의 함수 옮기기
        return this._host._show.hasOwnProperty('talkback')
    }
    extendBasePrice(base) { // 확장 메서드
        return Math.round(base + this._extras.premiumFee)
    }
}
// Step 9 팩터리 메서드가 슈퍼클래스를 반환하도록 수정
function createPremiumBooking(show, date, extras) {
    const result = new Booking(show, date, extras)
    result._bePremium(extra);
    return result
}

// 위임으로 변경함으로써 분배 로직과 양방향 참조가 더해지는 등 복잡도가 올라갔지만, 동적으로 프리미엄 예약으로 바꿀 수 있는 장점이 생겼고, 상속을 다른 목적으로 사용할 수 있게 되었다.
```

### 예시 2

```java
// 예시 2 서브클래스가 여러 개일 때
// ASIS
function createBird(data) {
    switch (data.type) {
        case '유럽 제비':
            return new EuropeanSwallow(data)
        case '아프리카 제비':
            return new AfricanSwallow(data)
        case '노르웨이 파랑 앵무':
            return new NorwegianBlueParrot(data)
        defulat:
            return new Bird(data)
    }
}

class Bird {
    constructor(data) {
        this._name = name
        this._plumage = data.plumage
    }
    get name() {
        return this._name
    }
    get plumage() {
        return this._plumage
    }
    get airSpeedVelocity() {
        return null
    }
}

class EuropeanSwallow extends Bird {
    get airSpeedVelocity() {
        return 35
    }
}

class AfricanSwallow extends Bird {
    constructor(data) {
        super(data)
        this._numberOfCoconuts = data.numberOfCoconuts
    }
    get airSpeedVelocity() {
        return 40 - 2 * this._numberOfCoconuts
    }
}

class NorwegianBlueParrot extends Bird {
    constructor(data) {
        super(data)
        this._voltage = data.voltage
        this._isNailed = data.isNailed
    }
    get plumage() {
        if (this._voltage > 100) return '그을렸다'
        else return this._plumage || '예쁘다'
    }
    get airSpeedVelocity() {
        return (this._isNailed) ? 0 : 10 + this._voltage / 10;
    }
}

// 야생(wild) 조류와 사육(captivity) 조류를 구분짓기 위해 수정할 예정
// WildBird와 CaptiveBird라는 두 서브클래스로 모델링 하는 방법도 있으나, 상속은 한 번만 쓸 수 있으니 야생과 사육을 기준으로 나누려면 종에 따른 분류를 포기해야 한다.
// Step 1 빈 위임 클래스를 만들다
class EuropeanSwallowDelegate  {
}
// Step 2 위임 필드를 어디에서 초기화 할지 정한다. 생성자가 받는 유일한 인수인 data가 모든 정보를 가지고 있으니 생성자에서 처리하고, 여러 위임을 만들어야 하니 타입 코드를 기준으로 올바른 위임을 선택하는 메서드를 생성한다.
class Bird {
    constructor(data) {
        this._name = name
        this._plumage = data.plumage
        this._speciesDelegate = this.selectSpeciesDelegate(data)
    }
    get name() {
        return this._name
    }
    get plumage() {
        return this._plumage
    }
    get airSpeedVelocity() {
        return null
    }
    selectSpeciesDelegate(data) {
        switch(data.type) {
            case '유럽 제비':
                return new EuropeanSwallowDelegate()
            default:
                return null
        }
    }
}

// Step 3 유럽 제비 비행 속도 메서드를 위임으로 옮기기
class EuropeanSwallowDelegate  {
    get airSpeedVelocity() {
        return 35
    }
}

class EuropeanSwallow extends Bird {
    get airSpeedVelocity() {
        return this._speciesDelegate.airSpeedVelocity // 변경
    }
}
// 슈퍼클래스의 airSpeedVelocity()를 수정하여, 위임이 존재하면 위임의 메서드를 호출하도록 한다.
class Bird {
    constructor(data) {
        this._name = name
        this._plumage = data.plumage
        this._speciesDelegate = this.selectSpeciesDelegate(data)
    }
    get name() {
        return this._name
    }
    get plumage() {
        return this._plumage
    }
    get airSpeedVelocity() {
        return this._speciesDelegate 
				? this._speciesDelegate.airSpeedVelocity 
				: null // 수정
    }
    selectSpeciesDelegate(data) {
        switch(data.type) {
            case '유럽 제비':
                return new EuropeanSwallowDelegate()
            default:
                return null
        }
    }
}
// 유럽 제비 클래스 제거
function createBird(data) {
    switch (data.type) {
        case '아프리카 제비':
            return new AfricanSwallow(data)
        case '노르웨이 파랑 앵무':
            return new NorwegianBlueParrot(data)
        defulat:
            return new Bird(data)
    }
}
// Step 4 아프리카 제비 위임 클래스 생성
class AfricanSwallowDelegate  {
}
class Bird {
    constructor(data) {
        this._name = name
        this._plumage = data.plumage
        this._speciesDelegate = this.selectSpeciesDelegate(data)
    }
    get name() {
        return this._name
    }
    get plumage() {
        return this._plumage
    }
    get airSpeedVelocity() {
        return this._speciesDelegate ? this._speciesDelegate.airSpeedVelocity : null // 수정
    }
    selectSpeciesDelegate(data) {
        switch(data.type) {
            case '유럽 제비':
                return new EuropeanSwallowDelegate()
            case '아프리카 제비':
                return new AfricanSwallowDelegate(data)
            default:
                return null
        }
    }
}
// Step 5 airSpeedVelocity 함수를 옮긴다
class AfricanSwallowDelegate  {
    get airSpeedVelocity() {
        return 40 - 2 * this._numberOfCoconuts
    }
}
class AfricanSwallow extends Bird {
    constructor(data) {
        super(data)
        this._numberOfCoconuts = data.numberOfCoconuts
    }
    get airSpeedVelocity() {
        return this._speciesDelegate.airSpeedVelocity
    }
}
// Step 6 아프리카 제비 서브클래스 제거
function createBird(data) {
    switch (data.type) {
        case '노르웨이 파랑 앵무':
            return new NorwegianBlueParrot(data)
        defulat:
            return new Bird(data)
    }
}
// Step 7 노르웨이 파랑 앵무 처리
class Bird {
    constructor(data) {
        this._name = name
        this._plumage = data.plumage
        this._speciesDelegate = this.selectSpeciesDelegate(data)
    }
    get name() {
        return this._name
    }
    get plumage() {
        return this._plumage
    }
    get airSpeedVelocity() {
        return this._speciesDelegate ? this._speciesDelegate.airSpeedVelocity : null // 수정
    }
    selectSpeciesDelegate(data) {
        switch(data.type) {
            case '유럽 제비':
                return new EuropeanSwallowDelegate()
            case '아프리카 제비':
                return new AfricanSwallowDelegate(data)
            case '노르웨이 파랑 앵무':
                return new NorwegianBlueParrotDelegate(data)
            default:
                return null
        }
    }
}

class NorwegianBlueParrotDelegate {
    constructor(data) {
        this._voltage = data.voltage
        this._isNailed = data.isNailed
    }

    get airSpeedVelocity() {
        return (this._isNailed) ? 0: 10 + this._voltage / 10;
    }
}
// Step 8 오버라이드하는 plumage 떄문에 생성자에 Bird로의 역참조를 추가해야 한다
class NorwegianBlueParrot extends Bird {
    constructor(data) {
        super(data)
        this._voltage = data.voltage
        this._isNailed = data.isNailed
    }
    get plumage() {
        return this._speciesDelegate.plumage
    }
    get airSpeedVelocity() {
        return (this._isNailed) ? 0 : 10 + this._voltage / 10;
    }
}

class NorwegianBlueParrotDelegate {
    get plumage() {
        if (this._voltage > 100) 
						return "그을렸다"
        else 
						return this._bird._plumage || "예쁘다"
    }

    constructor(data, bird) {
        this._bird = bird
        this._voltage = data.voltage
        this._isNailed = data.isNailed
    }
}

// Step 9 서브클래스에서 plumage 메서드를 제거
class Bird {
    constructor(data) {
        this._name = name
        this._plumage = data.plumage
        this._speciesDelegate = this.selectSpeciesDelegate(data)
    }
    get name() {
        return this._name
    }
    get plumage() {
        if (this._speciesDelegate)
            return this._speciesDelegate.plumage
        else
            return this._plumage || "보통이다"
    }
    get airSpeedVelocity() {
        return this._speciesDelegate ? this._speciesDelegate.airSpeedVelocity : null // 수정
    }
    selectSpeciesDelegate(data) {
        switch(data.type) {
            case '유럽 제비':
                return new EuropeanSwallowDelegate()
            case '아프리카 제비':
                return new AfricanSwallowDelegate(data)
            case '노르웨이 파랑 앵무':
                return new NorwegianBlueParrotDelegate(data, this) // 추가
            default:
                return null
        }
    }
}

class EuropeanSwallowDelegate  {
    get airSpeedVelocity() {
        return 35
    }
    get plumage() {
        return this._bird._plumage || "보통이다" // 중복 발생
    }
}

class AfricanSwallowDelegate  {
    get airSpeedVelocity() {
        return 40 - 2 * this._numberOfCoconuts
    }
    get plumage() {
        return this._bird._plumage || "보통이다" // 중복 발생
    }
}

// Step 10 중복을 해결하기 위해 상속을 사용한다
class SpeciesDelegate {
    constructor(data, bird) {
        this._bird = bird
    }
    get plumage() {
        return this._bird._plumage || "보통이다"
    }
}

class EuropeanSwallowDelegate extends SpeciesDelegate {
    get airSpeedVelocity() {
        return 35
    }
}

class NorwegianBlueParrotDelegate extends SpeciesDelegate {
    get plumage() {
        if (this._voltage > 100) return "그을렸다"
        else return this._bird._plumage || "예쁘다"
    }

    constructor(data, bird) {
        super(data, bird)
        this._voltage = data.voltage
        this._isNailed = data.isNailed
    }
}

// Step 11 Bird의 기본 동작 모두를 SpeciesDelegate 클래스로 옮긴다.
class Bird {
    constructor(data) {
        this._name = name
        this._plumage = data.plumage
        this._speciesDelegate = this.selectSpeciesDelegate(data)
    }
    get name() {
        return this._name
    }
    get plumage() {
        return this._speciesDelegate.plumage // 변경
    }
    get airSpeedVelocity() {
        return this._speciesDelegate.airSpeedVelocity // 변경
    }
    selectSpeciesDelegate(data) {
        switch(data.type) {
            case '유럽 제비':
                return new EuropeanSwallowDelegate()
            case '아프리카 제비':
                return new AfricanSwallowDelegate(data)
            case '노르웨이 파랑 앵무':
                return new NorwegianBlueParrotDelegate(data, this) // 추가
            default:
                return new SpeciesDelegate(data, this) // 변경
        }
    }
}

class SpeciesDelegate {
    constructor(data, bird) {
        this._bird = bird
    }
    get plumage() {
        return this._bird._plumage || "보통이다"
    }
    get airSpeedVelocity() {
        return null
    }
}
```

## 12.11 슈퍼클래스를 위임으로 바꾸기

### 배경

상속을 잘못 적용한 예로 자바의 스택 클래스가 있다.

데이터를 저장하고 조작하는 리스트의 기능을 재활용하겠다는 생각에서 비롯되었다.

리스트의 연산 중 스택에는 적용되지 않는 게 많음에도 그 모든 연산이 스택 인터페이스에 그대로 노출되어 있다.

이 보다는 스택에 리스트 객체를 필드에 저장해두고 필요한 기능만 위임했다면 더 좋았을 것이다.

슈퍼클래스의 기능들이 서브클래스에 어울리지 않는다면 상속을 통해 이용하면 안된다는 신호이다.

제대로 된 상속이라면 서브클래스가 슈퍼클래스의 모든 기능을 사용함을 물론, 서브클래스의 인스턴스를 슈퍼클래스의 인스턴스로도 취급할 수 있어야 한다.

다시말해 슈퍼클래스가 사용 되는 모든 곳에서 서브클래스의 인스턴스를 대신 사용해도 이상없이 동작해야 한다.

타입-인스턴스 동형이의어(type-instance homonym): 기존 클래스에 기능을 추가하면 다른 클래스로 재활용 할 수 있을 것이라고 착각하는 것

일부 기능만 빌려오는 것이라면 위임을 사용해야 한다.

### 절차

1. 슈퍼클래스 객체를 참조하는 필드를 서브클래스에 만든다(이번 리팩터링을 끝마치면 슈퍼클래스가 위임 객체가 될 것이므로 이 필드를 ‘위임 참조'라 부르자) 위임 참조를 새로운 슈퍼클래스 인스턴스로 초기화한다.
2. 슈퍼클래스의 동작 각각에 대응하는 전달 함수를 서브클래스에 만든다(물론 위임 참조로 전달한다). 서로 관련된 함수끼리 그룹으로 묶어 진행하며, 그룹을 하나씩 만들 때마다 테스트한다.
  1. 대부분은 전달 함수 각각을 테스트할 수 있을 것이다. 하지만 예컨대 게터와 쌍은 둘 다 옮긴 후에야 테스트 할 수 있다.
3. 슈퍼클래스의 동작 모두가 전달 함수로 오버라이드되었다면 상속 관계를 끊는다.

### 예시

```java
// 예시 1 고대 스크롤을 보관하고 있는 도서관, 스크롤들의 상세정보는 이미 카탈로그로 분류해 있었는데, 각 스크롤에는 ID 번호와 제목이 있고 그외 여러 가지 태그가 붙어 있다
class CatalogItem {
    constructor(id, title, tags) {
        this._id = id
        this._title = title
        this._tags = tags
    }
    get id() { return this._id }
    get title() { return this._title }
    hasTag(arg) { return this._tags.includes(arg) }
}
// 스크롤에는 정기 세척 이력이 필요하여 카탈로그 아아템을 확장하여 세척 관련 데이터를 추가해 사용
class Scroll extends CatalogItem {
    constructor(id, title, tags, dataLastCleaned) {
        super(id, title, tags)
        this._lastCleaned = dataLastCleaned
    }

    needsCleaning(targetDate) {
        const threshold = this.hasTag("revered") ? 700 : 1500
        return this.daysSinceLastCleaning(targetDate) > threshold
    }

    daysSinceLastCleaning(targetDate) {
        return this._lastCleaned.until(targetDate, ChronoUnit.DAYS)
    }
}

// 물리적인 스크롤과 논리적인 카탈로그 아이템에는 차이가 있다.
// Step 1 Scroll에 카탈로그 아이템을 참조할 수 있는 속성을 만들고 슈퍼클래스의 인스턴스를 새로 하나 만들어 대입하자
class Scroll extends CatalogItem {
    constructor(id, title, tags, dataLastCleaned) {
        super(id, title, tags)
        this._catalogItem = new CatalogItem(id, title, tags)
        this._lastCleaned = dataLastCleaned
    }

    needsCleaning(targetDate) {
        const threshold = this.hasTag("revered") ? 700 : 1500
        return this.daysSinceLastCleaning(targetDate) > threshold
    }

    daysSinceLastCleaning(targetDate) {
        return this._lastCleaned.until(targetDate, ChronoUnit.DAYS)
    }
}
// Step 2 서브클래스에서 사용하는 슈퍼클래스의 동작 각각에 대응하는 전달 메서드를 만든다
class Scroll extends CatalogItem {
    constructor(id, title, tags, dataLastCleaned) {
        super(id, title, tags)
        this._catalogItem = new CatalogItem(id, title, tags)
        this._lastCleaned = dataLastCleaned
    }
    get id() {
        return this._catalogItem.id
    }
    get title() {
        return this._catalogItem.title
    }
    hasTag(aString) {
        return this._catalogItem.hasTag(aString)
    }
    needsCleaning(targetDate) {
        const threshold = this.hasTag("revered") ? 700 : 1500
        return this.daysSinceLastCleaning(targetDate) > threshold
    }

    daysSinceLastCleaning(targetDate) {
        return this._lastCleaned.until(targetDate, ChronoUnit.DAYS)
    }
}
// Step 3 상속 관계를 끊는다
class Scroll {
    constructor(id, title, tags, dataLastCleaned) {
        this._catalogItem = new CatalogItem(id, title, tags)
        this._lastCleaned = dataLastCleaned
    }
    get id() {
        return this._catalogItem.id
    }
    get title() {
        return this._catalogItem.title
    }
    hasTag(aString) {
        return this._catalogItem.hasTag(aString)
    }
    needsCleaning(targetDate) {
        const threshold = this.hasTag("revered") ? 700 : 1500
        return this.daysSinceLastCleaning(targetDate) > threshold
    }
    daysSinceLastCleaning(targetDate) {
        return this._lastCleaned.until(targetDate, ChronoUnit.DAYS)
    }
}
```

### 더 가다듬기

```java
// 더 가다듬기
// 값을 참조로 바꾸기 
// 값을 참조로 바꾸기 위해 스크롤은 자신만의 아이디를 사용하도록 한다
// Step 1
class Scroll {
    constructor(id, title, tags, dataLastCleaned) {
        this._id = id
        this._catalogItem = new CatalogItem(null, title, tags)
        this._lastCleaned = dataLastCleaned
    }
    get id() {
        return this._id // 자신의 아이디 전달
    }
    get title() {
        return this._catalogItem.title
    }
    hasTag(aString) {
        return this._catalogItem.hasTag(aString)
    }
    needsCleaning(targetDate) {
        const threshold = this.hasTag("revered") ? 700 : 1500
        return this.daysSinceLastCleaning(targetDate) > threshold
    }

    daysSinceLastCleaning(targetDate) {
        return this._lastCleaned.until(targetDate, ChronoUnit.DAYS)
    }
}
// 스크롤 데이터 읽기
const scrolls = aDocumnet
    .map(record => new Scroll(record.id,
                              record.catalogData.title,
                              record.catalogData.tags,
                              LocalDate.parse(record._lastCleaned)
    ))

// Step 2 값을 참조로 만들기 위해 저장소를 찾고 없으면 새로 만든다.
// ID로 색인된 카탈로그 아이템을 저장소로 사용
const scrolls = aDocumnet
    .map(record => new Scroll(record.id,
                              record.catalogData.title,
                              record.catalogData.tags,
                              LocalDate.parse(record._lastCleaned),
                              record.catalogData.id,
                              catalog
    ))
class Scroll {
    constructor(id, title, tags, dataLastCleaned, catalogID, catalog) {
        this._id = id
        this._catalogItem = catalog.get(catalogID) // 변경
        this._lastCleaned = dataLastCleaned
    }
    get id() {
        return this._id // 자신의 아이디 전달
    }
    get title() {
        return this._catalogItem.title
    }
    hasTag(aString) {
        return this._catalogItem.hasTag(aString)
    }
    needsCleaning(targetDate) {
        const threshold = this.hasTag("revered") ? 700 : 1500
        return this.daysSinceLastCleaning(targetDate) > threshold
    }

    daysSinceLastCleaning(targetDate) {
        return this._lastCleaned.until(targetDate, ChronoUnit.DAYS)
    }
}
// Step 3 생성자로 건네지던 제목과 태그는 제거
const scrolls = aDocumnet
    .map(record => new Scroll(record.id,
                              LocalDate.parse(record._lastCleaned),
                              record.catalogData.id,
                              catalog
    ))
class Scroll {
    constructor(id, dataLastCleaned, catalogID, catalog) {
        this._id = id
        this._catalogItem = catalog.get(catalogID) // 변경
        this._lastCleaned = dataLastCleaned
    }
    get id() {
        return this._id // 자신의 아이디 전달
    }
    get title() {
        return this._catalogItem.title
    }
    hasTag(aString) {
        return this._catalogItem.hasTag(aString)
    }
    needsCleaning(targetDate) {
        const threshold = this.hasTag("revered") ? 700 : 1500
        return this.daysSinceLastCleaning(targetDate) > threshold
    }

    daysSinceLastCleaning(targetDate) {
        return this._lastCleaned.until(targetDate, ChronoUnit.DAYS)
    }
}
```

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}
