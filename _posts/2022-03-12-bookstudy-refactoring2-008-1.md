---
layout: single
title: 리팩터링2 스터디 - 008 - 1
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 8장 기능 이동

## 8.1 함수 옮기기 Move Function

### 배경

좋은 소프트웨어 설계의 핵심은 모듈성(Modularity)이다.

모듈성이란 프로그램 어딘가 수정하려 할 때 해당 기능과 관련된 작은 일부만 이해해도 가능하게 해주는 능력이다.

모듈성을 높이려면 연관된 요소들을 묶고, 요소 사이의 연결 관계를 쉽게 찾고 이해할 수 있도록 해야 한다.

프로그램에 대한 이해도가 높을 수록 요소들을 더 잘 묶을 수 있다.

어떤 함수가 자신이 속한 모듈 A의 요소들보다 다른 모듈 B의 요소들을 더 많이 참조한다면 모듈 B로 옮겨줘야 마땅하다.

대상 함수가 호출하는 함수는 무엇이고, 함수를 호출하는 함수는 무엇인지, 사용하는 데이터는 무엇인지를 살펴보아야 한다.

서로 연관된 함수들을 묶을 때에는 클래스 추출하기나 묶기를 사용한다.

→ 한 컨텍스트에 함수들을 두고 여러번 옮겨가면서 적절한 장소를 찾아 언제든 자리를 옮겨준다.

### 절차

1. 선택한 함수가 현재 컨텍스트에서 사용 중인 모든 프로그램 요소를 살펴본다. 이 요소들 중에도 함께 옮겨야 할 게 있는지 고민해본다.
  1. 호출되는 함수 중 함께 옮길 게 있다면, 대체로 그 함수를 먼저 옮기는 게 낫다. 얽혀 있는 함수가 여러 개라면 다른 곳에 미치는 영향이 적은 함수부터 옮기도록 하자
  2. 하위 함수들의 호출자가 고수준 함수 하나뿐이면 먼저 하위 함수들을 고수준 함수에 인라인한 다음, 고수준 함수를 옮기고, 옮긴 위치에서 개별 함수들로 다시 추출하자.
2. 선택한 함수가 다형 메서드인지 확인한다.
  1. 객체 지향 언어에서는 같은 메서드가 슈퍼클래스나 서브클래스에도 선언되어 있는지까지 고려해야 한다.
3. 선택한 함수를 타깃 컨텍스트로 복사한다(이때 원래의 함수를 소스 함수(Source function)라 하고 복사해서 만든 새로운 함수를 타깃 함수(Target function)라 한다). 타깃 함수가 새로운 터전에 잘 자리 잡도록 다듬는다.
  1. 함수 본문에서 소스 컨텍스트의 요소를 사용한다면 해당 요소들을 매개변수로 넘기거나 소스 컨텍스트 자체를 참조로 넘겨준다.
  2. 함수를 옮기게 되면 새로운 컨텍스트에 어울리는 새로운 이름으로 바꿔줘야 할 경우가 많다. 필요하면 바꿔준다.
4. 정적 분석을 수행한다.
5. 소스 컨텍스트에서 타깃 함수를 참조할 방법을 찾아 반영한다.
6. 소스 함수를 타깃 함수의 위임 함수가 되도록 수정한다.
7. 테스트한다.
8. 소스 함수를 인라인할지 고민해본다.
  1. 소스 함수는 언제까지라도 위임 함수로 남겨둘 수 있다. 하지만 소스 함수를 호출하는 곳에서 타깃 함수를 직접 호출하는 데 무리가 없다면 중간 단계(소스 함수)는 제거하는 편이 낫다.

### 예시

중첩 함수를 최상위로 옮기기

```jsx

// 예시 중첩 함수를 최상위로 옮기기
// GPS 추적 기록의 총 거리를 계산하는 함수
// 중첩 함수인 calculateDistance()를 최상위로 옮겨서 추적 거리를 다른 정보와는 독립적으로 계산하고 싶다.
// ASIS
function trackSummary(points) {
    const totalTime =calculateTime();
    const totalDistance = calculateDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    };

    function calculateDistance() { // 총 거리 계산
        let result = 0;
        for (let i = 1; i < points.length; i++) {
            result += distance(points[i-1], points[i]);
        }
        return result;
    }

    function distance(p1, p2) {} // 두 지점의 거리 계산
    function radians(degress) {} // 라디안 값으로 변환
    function calculateTime() {} // 총 시간 계산
}

// Step 1 calculateDistance함수를 최상위로 복사
// 새로운 이름을 지어주면서 소스 함수와 타깃 함수를 구분한다.
function top_calculateDistance() {  //최상위로 복사하면서 새로운 (임시) 이름을 지어준다.
    let result = 0;
    for (let i = 1; i < points.length; i++) {
        result += distance(points[i-1], points[i]);
    }
    return result;
}

function trackSummary(points) {
    const totalTime =calculateTime();
    const totalDistance = calculateDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    };

    function calculateDistance() { // 총 거리 계산
        let result = 0;
        for (let i = 1; i < points.length; i++) {
            result += distance(points[i-1], points[i]);
        }
        return result;
    }

    function distance(p1, p2) {} // 두 지점의 거리 계산
    function radians(degress) {} // 라디안 값으로 변환
    function calculateTime() {} // 총 시간 계산
}

// Step 2 새 함수에 정의되어 있지 않은 points는 매개변수로 넘겨준다
function top_calculateDistance(points) {  // points를 매개변수로 넘겨줌.
    let result = 0;
    for (let i = 1; i < points.length; i++) {
        result += distance(points[i-1], points[i]);
    }
    return result;
}

function trackSummary(points) {
    const totalTime =calculateTime();
    const totalDistance = calculateDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    };

    function calculateDistance() { // 총 거리 계산
        let result = 0;
        for (let i = 1; i < points.length; i++) {
            result += distance(points[i-1], points[i]);
        }
        return result;
    }

    function distance(p1, p2) {} // 두 지점의 거리 계산
    function radians(degress) {} // 라디안 값으로 변환
    function calculateTime() {} // 총 시간 계산
}

// Step 3 distance함수 처리
// calculateDistance와 함께 옮기는 방식으로 처리할 예정이다.
// distance 함수 
function distance(p1, p2) {
    const EARTH_RADIUS = 3959;
    const dLat = radians(p2.lat) - radians(p1.lat);
    const dLon = radians(p2.lon) - radians(p2.lat);
    const a = Math.pow(Math,sin(dLat / 2), 2)
            + Math.cos(radians(p2.lat))
            * Math.cos(radians(p1.lat))
            * Math.pow(Math.sin(dLon / 2), 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    return EARTH_RADIUS * c;
}

function radians(degrees) {
    return degrees * Math.PI / 180;
}
// distance는 radians 함수만 사용하며 radians는 현재 컨텍스트에 있는 어떤 것도 사용하지 않는다.
// 두 함수를 매개변수로 넘기기보다는 함께 옮기는 것이 낫다.

// Step 4 현재 컨텍스트에서 이 함수들을 calculateDistance 함수 안으로 옮겨보자

function trackSummary(points) {
    const totalTime =calculateTime();
    const totalDistance = calculateDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    };

    function calculateDistance() { // 총 거리 계산
        let result = 0;
        for (let i = 1; i < points.length; i++) {
            result += distance(points[i-1], points[i]);
        }

        function distance(p1, p2) {} // 함수 옮김
        function radians(degress) {} // 함수 옮김

        return result;
    }

    function calculateTime() {} // 총 시간 계산
}

// Step 5 정적 분석과 테스트를 활용하여 문제 검증 후 이상 없으면 top_calculateDistance() 함수에도 복사
function top_calculateDistance(points) {  // points를 매개변수로 넘겨줌.
    let result = 0;
    for (let i = 1; i < points.length; i++) {
        result += distance(points[i-1], points[i]);
    }

    function distance(p1, p2) {} // 함수 옮김
    function radians(degress) {} // 함수 옮김

    return result;
}

//Step 6 calculateDistance()의 본문의 수정하여 top_calculateDistance()를 호출하게 하자
function top_calculateDistance(points) {  // points를 매개변수로 넘겨줌.
    let result = 0;
    for (let i = 1; i < points.length; i++) {
        result += distance(points[i-1], points[i]);
    }

    function distance(p1, p2) {} // 함수 옮김
    function radians(degress) {} // 함수 옮김

    return result;
}

function trackSummary(points) {
    const totalTime =calculateTime();
    const totalDistance = calculateDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    };

    function calculateDistance() { // 총 거리 계산
        top_calculateDistance(points) // 새로 만든 함수를 호출하게 변경, 테스트 필수!
    }
    function calculateTime() {} // 총 시간 계산
}

// Step 7 소스 함수를 대리자 역할로 그대로 둘지 정하는데, 이 예에서 소스함수는 호출자가 많지 않은, 지역화된 함수여서 제거하는 편이 낫다 제거하자!
function top_calculateDistance(points) {  // points를 매개변수로 넘겨줌.
    let result = 0;
    for (let i = 1; i < points.length; i++) {
        result += distance(points[i-1], points[i]);
    }

    function distance(p1, p2) {} // 함수 옮김
    function radians(degress) {} // 함수 옮김

    return result;
}

function trackSummary(points) {
    const totalTime =calculateTime();
    const totalDistance = top_calculateDistance(points);
    const pace = totalTime / 60 / totalDistance;
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    };
    //소스 함수 제거 됨
    function calculateTime() {} // 총 시간 계산
}

// Step 8 새 함수에 이름을 지어줄 시간, 최상단 함수는 가시성이 높으니 적합한 이름을 신중히 지어주어야 한다.
// 함수명을 totalDistance로 변경하고 변수 totalDistance를 제거한 후 인라인 하자!
function totalDistance(points) {  // points를 매개변수로 넘겨줌.
    let result = 0;
    for (let i = 1; i < points.length; i++) {
        result += distance(points[i-1], points[i]);
    }

    function distance(p1, p2) {} // 함수 옮김
    function radians(degress) {} // 함수 옮김

    return result;
}

function trackSummary(points) {
    const totalTime =calculateTime();
    const pace = totalTime / 60 / totalDistance(points); // 변수 제거 후 함수 인라인
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    };
    //소스 함수 제거 됨
    function calculateTime() {} // 총 시간 계산
}

// Step 9 distance와 radians도 totalDistance안에 의존성이 없으니 최상위로 옮겨준다.

function distance(p1, p2) {} // 함수 옮김
function radians(degress) {} // 함수 옮김

function totalDistance(points) {  // points를 매개변수로 넘겨줌.
    let result = 0;
    for (let i = 1; i < points.length; i++) {
        result += distance(points[i-1], points[i]);
    }

    return result;
}

function trackSummary(points) {
    const totalTime =calculateTime();
    const pace = totalTime / 60 / totalDistance(points); // 변수 제거 후 함수 인라인
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    };
    //소스 함수 제거 됨
    function calculateTime() {} // 총 시간 계산
}

// 예시 2 다른 클래스로 옮기기
class Account {
    get bankCharge() { //은행 이자 계산
        let result = 4.5;
        if (this._daysOverdrawn > 0) result += this.overdraftCharge;
        return result;
    }

    get overdraftCharge() { // 초과 인출 이자 계산
        if (this.type.isPremium) {
            const baseCharge = 10;
            if (this._daysOverdrawn <= 7)
                return baseCharge
            else
                return baseCharge + (this.daysOverdrawn - 7) * 0.85;
        }
        else
            return this.daysOverdrawn * 1.75;
    }
}
// 계좌 종류에 따라 이자 책정 알리고리즘이 달라지도록 고칠것이다.
// 마이너스 통장의 초과 인출 이자를 계산하는 overdraftCharge()를 계좌 클래스인 AccountType으로 옮기는게 자연스럽다
// overdraftCharg() 메소드가 사용하는 기능들을 살펴보고, 모두를 함께 옮길만한 가치가 있는지 고민해보자
// 이 예에서는 daysOverdrawn() 메서드는 Account 클래스에 남겨두어야 한다. (계좌 종류가 아닌) 계좌별로 달라지는 메서드이기 때문
// Step 1 overdraftCharge() 메서드 본문을 AccountType 클래스로 복사한 후 정리
// 새 보금자리에 맞추려면 두 개의 범위를 조정 isPremium은 단순히 this를 통해 호출
// daysOverdrawn은 값을 넘길지, 아니면 계좌채로 넘길지 정해야 한다.
// 우선은 값을 넘기는 방식으로 처리, 하지만 초과 인출된 일수 외에 다른 정보가 필요해지면 추후 계좌채로 넘기도록 변경 가능
class AccountType {
    overdraftCharge(daysOverdrawn) { // 초과 인출 이자 계산 값으로 넘김
        if (this.isPremium) {  // 변경
            const baseCharge = 10;
            if (daysOverdrawn <= 7) // 인수값 받아서 처리
                return baseCharge
            else
                return baseCharge + (daysOverdrawn - 7) * 0.85; // 인수값 받아서 처리
        }
        else
            return daysOverdrawn * 1.75; // 인수값 받아서 처리
    }
}

// Step 2 원래 메서드의 본문을 수정하여 새 메서드를 호출하도록 수정
class Account {
    get bankCharge() { //은행 이자 계산
        let result = 4.5;
        if (this._daysOverdrawn > 0) result += this.overdraftCharge;
        return result;
    }

    get overdraftCharge() { // 초과 인출 이자 계산
       return this.type.overdraftCharge(this.daysOverdrawn);
    }
}

// Step 3 소스 메소드를 남겨둘지 인라인 할지 결정 -> 인라인으로 선택
class Account {
    get bankCharge() { //은행 이자 계산
        let result = 4.5;
        if (this._daysOverdrawn > 0) result += this.type.overdraftCharge(this.daysOverdrawn);
        return result;
    }
}

// 만약 계좌에서 가져와야 할 데이터가 많다면 계좌 자체를 넘긴다
class Account {
    get bankCharge() { //은행 이자 계산
        let result = 4.5;
        if (this._daysOverdrawn > 0) result += this.type.overdraftCharge;
        return result;
    }

    get overdraftCharge() {
        return this.type.overdraftCharge(this); // 계좌 자체를 넘김
    }
}

class AccountType {
    overdraftCharge(account) { // 계좌 자체를 넘겨 받음
        if (this.isPremium) {  // 변경
            const baseCharge = 10;
            if (account.daysOverdrawn <= 7) // 계좌에서 인수값 받아서 처리
                return baseCharge
            else
                return baseCharge + (account.daysOverdrawn - 7) * 0.85; //  계좌에서 인수값 받아서 처리
        }
        else
            return account.daysOverdrawn * 1.75; // 계좌에서 인수값 받아서 처리
    }
}
```

## 8.2 필드 옮기기 Move Field

### 배경

데이터 구조를 잘 짜놓으면 자연스럽게 단순하고 직관적으로 코드가 짜여진다.

데이터 구조가 적절하지 않다고 느낄 때마다 곧바로 수정을 진행해야 한다.

필드 옮기기는 대체로 더 큰 변경의 일환으로 수행된다.

### 절차

1. 소스 필드가 캡슐화되어 있지 않다면 캡슐화한다.
2. 테스트한다.
3. 타깃 객체에 필드(와 접근자 메서드들)를 생성한다.
4. 정적 검사를 수행한다.
5. 소스 객체에서 타깃 객체를 참조할 수 있는지 확인한다.
  1. 기존 필드나 메서드 중 타깃 객체를 넘겨주는 게 있을지 모른다. 없다면 이런 기능의 메서드를 쉽게 만들 수 있는지 살펴본다. 간단치 안다면 타깃 객체를 저장할 새 필드를 소스 객체에 생성하자. 이는 영구적인 변경이 되겠지만, 더 넓은 맥락에서 리팩토링을 충분히 하고 나면 다시 없앨 수 있을 때도 많다.
6. 접근자들이 타깃 필드를 사용하도록 수정한다.
  1. 여러 소스에서 같은 타깃을 공유한다면 먼저 세터를 수정하여 타깃 필드와 소스 필드 모두를 갱신하게 하고, 이어서 일관성을 깨드리는 갱신을 검출할 수 있도록 어셔션을 추가하자. 모든 게 잘 마무리되었다면 접근자들이 타깃 필드를 사용하도록 수정한다.
7. 테스트한다.
8. 소스 필드를 제거한다.
9. 테스트한다.

### 예시

```jsx
// ASIS
class Customer {
    get plan() { return this._plan; }
    get discountRate() { return this._discountRate; }
}

// TOBE
class Customer {
    get plan() { return this._plan; }
    get discountRate() { return this.plan.discountRate; }
}

// 예시
// 고객 클래스와 계약 클래스에서 시작
// ASIS
// 여기서 할일율을 뜻하는 discountRate 필드를 Customer에서 CustomerContract로 옮기고 싶다고 가정
class Customer {
    constructor(name, discountRate) {
        this._name = name;
        this._discountRate = discountRate;
        this._contract = new CustomerContract(dateToday());
    }

    get discountRate() { return this._discountRate; }
    becomePreferred() {
        this._discountRate += 0.03;
        // 다른 멋진 일들
    }
    applyDiscount(amount) {
        return amount.subtract(amount.mutiply(this._discountRate));
    }
}

class CustomerContract {
    constructor(startDate) {
        this._startDate = startDate;
    }
}

// Step 1 discountRate 필드를 캡슐화
// 할인율를 수정하는 public 세터를 만들고 싶지 않아 세터 속성이 아니라 메서드를 이용
class Customer {
    constructor(name, discountRate) {
        this._name = name;
        this._setDiscountRate(discountRate); // 변경
        this._contract = new CustomerContract(dateToday());
    }

    get discountRate() { return this._discountRate; } // 변경
    _setDiscountRate(aNumber) {this._discountRate = aNumber; } // 추가
    becomePreferred() {
        this._setDiscountRate(this.discountRate + 0.03); // 변경
        // 다른 멋진 일들
    }
    applyDiscount(amount) {
        return amount.subtract(amount.mutiply(this.discountRate)); // 변경
    }
}

class CustomerContract {
    constructor(startDate) {
        this._startDate = startDate;
    }
}

// Step 2 CustomerContract 클래스에 필드 하나와 접근자를 추가
class CustomerContract {
    constructor(startDate, discountRate) {
        this._startDate = startDate;
        this._discountRate = discountRate;
    }

    get discountRate() { return this._discountRate; }
    set discountRate(arg) { this._discountRate = arg; }
}

// Step3 Customer 접근자들이 새로운 필드를 사용하도록 수정한다
class Customer {
    constructor(name, discountRate) {
        this._name = name;
        this._contract = new CustomerContract(dateToday());
        this._setDiscountRate(discountRate); // 변경
    }

    get discountRate() { return this._constract.discountRate; } // 변경
    _setDiscountRate(aNumber) {this._contract.discountRate = aNumber; } 
    becomePreferred() {
        this._setDiscountRate(this.discountRate + 0.03); 
        // 다른 멋진 일들
    }
    applyDiscount(amount) {
        return amount.subtract(amount.mutiply(this.discountRate)); 
    }
}
```

### 날 raw 레코드 변경하기

여러 함수가 날 raw 레코드를 직접 사용하는 경우 리팩토링은 까다롭다

이럴 때는 접근자 함수들을 만들고, 날 레코드 읽고 쓰는 모든 함수가 접근자를 거치도록 고치면 된다.

옮길 필드가 불변이라면 값을 처음 설정할 때 소스와 타깃 필드를 한꺼번에 갱신하게 하고, 읽기 함수들은 점진적으로 마이그레이션하자.

가능하면 가장 먼저 레코드를 캡슐화하여 클래스로 바꾸자.

```jsx
// 예제2 공유 객체로 이동하기
// 이자율 interest rate을 계좌 account별로 설정
// 코드를 수정하여 아자율이 계좌 종류에 따라 정해지도록 수정
// ASIS
class Account {
    constructor(number, type, interestRate) {
        this._number = number;
        this._type = type;
        this._interestRate = interestRate;
    }

    get interestRate() { return this._interestRate; }
}

class AccountType {
    constructor(nameString) {
        this._name = nameString;
    }
}
// Step 1 타겟인 AccountType에 이자율 필드와 필요한 접근자 메서드를 생성해보자
// 주의
// Account가 AccountType의 이자율을 가져오도록 수정하면 문제가 생길수 있다.
// 리팩터링 전에는 각 계좌가 자신만의 이자율을 갖고 있었고, 지금은 종류가 같은 모든 계좌가 이자율을 공유하기를 원한다.
// 만약 수정 전에도 이자율이 계좌 종류별로 같게 설정되어 있었다면 겉보기 동작이 달라지지 않으니 그대로 리팩터링하면 된다.
// 이자율이 다른 계좌가 하나라도 있었다면, 이건 더 이상 리팩터링이 아니다.
// 모든 계좌의 아자율이 계좌 종류에 부합하게 설정되어 있는지 확인해야 한다. 계좌 클래스에 어서션을 추가하는 것도 도움이 된다.

class AccountType {
    constructor(nameString, interestRate) {
        this._name = nameString;
        this._interestRate = interestRate;
    }

    get interestRate() { return this._interestRate; }
}

// Step 2 어서션 추가
class Account {
    constructor(number, type, interestRate) {
        this._number = number;
        this._type = type;
        assert(interestRate === this._type.interestRate);
        this._interestRate = interestRate;
    }

    get interestRate() { return this._interestRate; }
}

// Step3 Account 클래스에서 이자율을 직접 수정하던 코드를 완전히 제거
class Account {
    constructor(number, type) {
        this._number = number;
        this._type = type;
    }

    get interestRate() { return this._interestRate; }
}
```

## 8.3 문장을 함수로 옮기기 Move Statement into Function

### 배경

중복 제거는 코드를 건강하게 관리하는 가장 효과적인 방법 중 하나이다.

문장들을 함수로 옮기려면 그 문장들이 피호출 함수의 일부라는 확신이 있어야 한다.

피호출 함수와 한 몸은 아니지만 여전히 함께 호출해야 하는 경우라면 단순히 해당 문장들과 피호출 함수를 통째로 하나의 함수로 추출한다.

단, 마지막의 인라인과 이름 바꾸기 단계만 제외하면 된다.

### 절차

1. 반복 코드가 함수 호출 부분과 멀리 떨어져 있다면 문장 슬라이드하기를 적용해 근처로 옮긴다.
2. 타깃 함수를 호출하는 곳이 한 곳뿐이라면, 단순히 소스 위치에서 해당 코드를 잘라내어 피호출 함수로 복사하고 테스트한다. 이 경우라면 나머지 단계는 무시한다.
3. 호출자가 둘 이상이면 호출자 중 하나에서 ‘타깃 함수 호출 부분과 그 함수로 옮기려는 문장들을 함께' 다른 함수로 추출한다. 추출한 함수에 기억하기 쉬운 임시 이름을 지어준다.
4. 다른 호출자 모두가 방금 추출한 함수를 사용하도록 수정한다. 하나씩 수정할 때마다 테스트한다.
5. 모든 호출자가 새로운 함수를 사요하게 되면 원래 함수를 새로운 함수 안으로 인라인한 후 원래 함수를 제거한다.
6. 새로운 함수의 이름을 원래 함수의 이름으로 바꿔준다(함수 이름 바꾸기)

### 예시

```jsx
// ASIS
result.push(`<p>제목: ${person.photo.title}</p>`);
result.concat(photoData(person.photo));

function photoData(aPhoto) {
    return [
        `<p>위치: ${aPhoto.location}</p>`,
        `<p>날짜: ${aphoto.toDateString()}</p>`,
    ];
}

// TOBE

result.concat(photoData(person.photo));

function photoData(aPhoto) {
    return [
        `<p>제목: ${aPhoto.title}</p>`,
        `<p>위치: ${aPhoto.location}</p>`,
        `<p>날짜: ${aphoto.toDateString()}</p>`,
    ]
}

// 예시 
// 사진 관련 데이터를 HTML로 내보내는 코드
// 총 두 곳에서 emitPhotoDate()를 호출하며 바로 앞에는 제목 출력 코드가 나온다.
function renderPerson(outStream, person) {
    const result = [];
    result.push(`<p>${person.name}</p>`);
    result.push(renderPhoto(person.photo));
    result.push(`<p>제목: ${person.photo.title}</p>`); // 제목 출력
    result.push(emitPhotoData(person.photo));
    return result.join("\n");
}

function photoDiv(p) {
    return [
        "<div>",
        `<p>제목: ${p.title}</p>`, // 제목 출력
        emitPhotoData(p),
        "</div>",
    ].join("\n");
}

function emitPhotoData(aPhoto) {
    const result = [];
    result.push(`<p>위치: ${aPhoto.location}</p>`);
    result.push(`<p>날짜: ${aPhoto.date.toDateString()}</p>`);
    return result.join("\n");
}
// Step 1 제목을 출력하는 코드를 emitPhotoData()안으로 옮겨 중복을 제거하자
function renderPerson(outStream, person) {
    const result = [];
    result.push(`<p>${person.name}</p>`);
    result.push(renderPhoto(person.photo));
    result.push(`<p>제목: ${person.photo.title}</p>`); // 제목 출력
    result.push(emitPhotoData(person.photo));
    return result.join("\n");
}

function photoDiv(p) {
    return [
        "<div>",
        zznew(p), // 새로운 함수로 호출
        "</div>",
    ].join("\n");
}

function zznew(p) { // 새로운 함수 생성
    return [
        `<p>제목: ${person.photo.title}</p>`,
        emitPhotoData(p),
    ].join("\n");
}

function emitPhotoData(aPhoto) {
    const result = [];
    result.push(`<p>위치: ${aPhoto.location}</p>`);
    result.push(`<p>날짜: ${aPhoto.date.toDateString()}</p>`);
    return result.join("\n");
}

// Step 2 다른 호출자들 renderPerson도 수정
function renderPerson(outStream, person) {
    const result = [];
    result.push(`<p>${person.name}</p>`);
    result.push(renderPhoto(person.photo));
    result.push(zznew(person.photo))
    return result.join("\n");
}

// Step 3 emitPhotoData() 함수를 인라인한다.
function zznew(p) { // 새로운 함수 생성
    return [
        `<p>제목: ${person.photo.title}</p>`,
        `<p>위치: ${aPhoto.location}</p>`,
        `<p>날짜: ${aPhoto.date.toDateString()}</p>`,
    ].join("\n");
}

// Step 4 함수 이름을 바꾸어 마무리 한다

function renderPerson(outStream, person) {
    const result = [];
    result.push(`<p>${person.name}</p>`);
    result.push(renderPhoto(person.photo));
    result.push(emitPhotoData(person.photo))
    return result.join("\n");
}

function photoDiv(p) {
    return [
        "<div>",
        emitPhotoData(p), // 새로운 함수로 호출
        "</div>",
    ].join("\n");
}

function emitPhotoData(p) { // 새로운 함수 생성
    return [
        `<p>제목: ${person.photo.title}</p>`,
        `<p>위치: ${aPhoto.location}</p>`,
        `<p>날짜: ${aPhoto.date.toDateString()}</p>`,
    ].join("\n");
}
```

## 8.4 문장을 호출한 곳으로 옮기기 Move Statements to Callers

### 배경

함수는 프로그래머가 쌓아 올리는 추상화의 기본 빌딩 블록이다.

추상화는 경계를 긋기가 어렵다.

코드베이스의 가능 범위가 달라지면 추상화의 경계도 움직이게 된다.

함수 관점에서 생각해보면 초기에는 응집도 높고 한 가지 일만 수행하던 함수가 어느새 둘 이상의 다른 일을 수행하게 바뀔 수 있다는 뜻이다.

여러 곳에서 사용하던 기능이 일부 호출자에게는 다르게 동작하도록 바꿔야 한다면 달라진 동작을 꺼내 해당 호출자로 옮겨야 한다.

### 절차

1. 호출자가 한두 개 뿐이고 피호출 함수도 간단한 단순한 상황이면, 피호출 함수의 처음(혹은 마지막)줄(들)을 잘라내어 호출자(들)로 복사해 넣는다(필요하면 적당히 수정한다). 테스트만 통과하면 이번 리팩토링은 여기서 끝이다.
2. 더 복잡한 상황에서는, 이동하지 ‘않길' 원하는 모든 문장을 함수로 추출한 다음 검색하기 쉬운 임시 이름을 지어준다
  1. 대상 함수가 서브클래스에서 오버라이드됐다면 오버라이드한 서브클래스들의 메서드 모두에서 동일하게, 남길 부분을 메서드로 추출한다. 이때 남겨질 메서드의 본문은 모든 클래스에서 똑같아야한다. 그런 다음 (슈퍼클래스의 메서드만 남기고) 서브클래스들의 메서드를 제거한다.
3. 원래 함수를 인라인한다.
4. 추출된 함수의 이름을 원래 함수의 이름으로 변경한다(함수 이름 바꾸기)

### 예시

```jsx
// ASIS
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo) {
    outStream.write(`<p>제목: ${photo.title}</p>\n`);
    outStream.write(`<p>위치: ${photo.location}</p>\n`);
}

// TOBE

emitPhotoData(outStream, person.photo);
outStream.write(`<p>위치: ${photo.location}</p>\n`)

function emitPhotoData(outStream, photo) {
    outStream.write(`<p>제목: ${photo.title}</p>\n`);
}

// 예시
// 호출자가 둘뿐인 단순한 상황
// ASIS
// 이 코드를 수정하여 renderPerson()은 그대로 둔 채 listRecentPhotos()가 위치 정보 location를 다르게 렌더링하도록 만들어야 한다.
// renderPerson()의 마지막 줄을 잘라내어 두 호출 코드 아래에 붙여 넣으면 끝이다, 하지만 더 안전한 길로 가본다.
function renderPerson(outStream, person) {
    outStream.write(`<p>${person.name}</p>\n`);
    renderPhoto(outStream, person.photo);
    emitPhotoData(outStream, person.photo);
}

function listRecentPhotos(outStream, photos) {
    photos.filter(p => p.date > recentDateCutoff())
    .forEach(p => {
        outStream.write("<div>\n")
        emitPhotoData(outStream, p);
        outStream.write("</div>\n");
    });
}

function emitPhotoData(outStream, photo) {
    outStream.write(`<p>제목: ${photo.title}</p>\n`);
    outStream.write(`<p>날짜: ${photo.date.toDateString()}</p>\n`);
    outStream.write(`<p>위치: ${photo.location}</p>\n`);
}

// Step 1 emitPhotoData()에 남길 코드를 함수로 추출한다.
function renderPerson(outStream, person) {
    outStream.write(`<p>${person.name}</p>\n`);
    renderPhoto(outStream, person.photo);
    emitPhotoData(outStream, person.photo);
}

function listRecentPhotos(outStream, photos) {
    photos.filter(p => p.date > recentDateCutoff())
    .forEach(p => {
        outStream.write("<div>\n")
        emitPhotoData(outStream, p);
        outStream.write("</div>\n");
    });
}

function emitPhotoData(outStream, photo) {
    zztmp(outStream, photo); // 임시 함수 호출
    outStream.write(`<p>위치: ${photo.location}</p>\n`);
}

function zztmp(outStream, photo) { // 임시 함수 생성, 이동하지 않을 코드
    outStream.write(`<p>제목: ${photo.title}</p>\n`);
    outStream.write(`<p>날짜: ${photo.date.toDateString()}</p>\n`);
}

// Step2 피호출 함수를 호출자들로 한 번에 하나씩 인라인한다.
function renderPerson(outStream, person) {
    outStream.write(`<p>${person.name}</p>\n`);
    renderPhoto(outStream, person.photo);
    zztmp(outStream, photo); // emitPhotoData에서 변경
    outStream.write(`<p>위치: ${photo.location}</p>\n`); // emitPhotoData에서 변경
}

function listRecentPhotos(outStream, photos) {
    photos.filter(p => p.date > recentDateCutoff())
    .forEach(p => {
        outStream.write("<div>\n")
        zztmp(outStream, p); // emitPhotoData에서 변경
        outStream.write(`<p>위치: ${p.location}</p>\n`); // emitPhotoData에서 변경
        outStream.write("</div>\n");
    });
}

function emitPhotoData(outStream, photo) {
    zztmp(outStream, photo); // 임시 함수 호출
    outStream.write(`<p>위치: ${photo.location}</p>\n`);
}

function zztmp(outStream, photo) { // 임시 함수 생성, 이동하지 않을 코드
    outStream.write(`<p>제목: ${photo.title}</p>\n`);
    outStream.write(`<p>날짜: ${photo.date.toDateString()}</p>\n`);
}

// Step3 원래 함수를 지워 인라인하기 마무리
function renderPerson(outStream, person) {
    outStream.write(`<p>${person.name}</p>\n`);
    renderPhoto(outStream, person.photo);
    zztmp(outStream, photo); // emitPhotoData에서 변경
    outStream.write(`<p>위치: ${photo.location}</p>\n`); // emitPhotoData에서 변경
}

function listRecentPhotos(outStream, photos) {
    photos.filter(p => p.date > recentDateCutoff())
    .forEach(p => {
        outStream.write("<div>\n")
        zztmp(outStream, p); // emitPhotoData에서 변경
        outStream.write(`<p>위치: ${p.location}</p>\n`); // emitPhotoData에서 변경
        outStream.write("</div>\n");
    });
}

function zztmp(outStream, photo) { // 임시 함수 생성, 이동하지 않을 코드
    outStream.write(`<p>제목: ${photo.title}</p>\n`);
    outStream.write(`<p>날짜: ${photo.date.toDateString()}</p>\n`);
}

// Step 4 zztmp()의 이름을 원래 함수의 이름으로 되돌린다.
function renderPerson(outStream, person) {
    outStream.write(`<p>${person.name}</p>\n`);
    renderPhoto(outStream, person.photo);
    emitPhotoData(outStream, photo); // 원래 함수 이름으로 되돌림
    outStream.write(`<p>위치: ${photo.location}</p>\n`); 
}

function listRecentPhotos(outStream, photos) {
    photos.filter(p => p.date > recentDateCutoff())
    .forEach(p => {
        outStream.write("<div>\n")
        emitPhotoData(outStream, p); // 원래 함수 이름으로 되돌림
        outStream.write(`<p>위치: ${p.location}</p>\n`); 
        outStream.write("</div>\n");
    });
}

function emitPhotoData(outStream, photo) { // 원래 함수 이름으로 되돌림
    outStream.write(`<p>제목: ${photo.title}</p>\n`);
    outStream.write(`<p>날짜: ${photo.date.toDateString()}</p>\n`);
}
```

## 8.5 인라인 코드를 함수 호출로 바꾸기 Replace Inline Code with Function Call

### 배경

함수는 여러 동작을 하나로 묶어준다.

함수의 이름이 코드의 동작 방식보다 목적을 말해주기 때문에 함수를 활용하면 코드를 이해하기 쉬워진다.

함수는 중복을 없애는 데도 효과적이다.

이미 존재하는 함수와 똑같은 일을 하는 인라인 코드를 발경하면 보통은 해당 코드를 함수 호출로 대체하길 원한다.

기존 함수의 코드를 수정하더라도 인라인 코드의 동작은 바뀌지 않아야 할 뿐이다.

함수의 일름을 잘 지었다면 인라인 코드 대신 함수 이름을 넣어도 말이 된다.

말이 되지 않는다면 함수 이름이 적절하지 않거나 그 함수의 목적이 인라인 코드의 목적과 다르기 때문일 것이다.

### 절차

1. 인라인 코드를 함수 호출로 대체한다
2. 테스트한다

### 함수 추출하기(6.1) 과 다른점

함수 추출하기는 인라인을 대체할 함수가 없을 때 함수를 만들고 대체하는 것이고 이번 8.5 인라인 코드를 기존 함수로 바꾸는 리팩터링은 이미 존재하는 함수를 인라인 코드를 대신하여 사용하는 것이다.

외부 라이브러리가 기존의 함수의 역할을 대신하면 이 리팩터링을 활용할 수 있다.

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}
