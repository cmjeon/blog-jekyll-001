---
layout: single
title: 리팩터링2 스터디 - 009 - 1
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 10장 조건부 로직 간소화

## 10.1 조건문 분해하기 Decompose Conditional

### 배경

복잡한 조건부 로직은 프로그램을 복잡하게 만드는 가장 흔한 원흉에 속한다.

긴 함수는 그 자체로 읽기가 어렵지만, 조건문은 그 어려움을 한층 가중시킨다.

거북한 코드 블록이 주어지면 코드를 부위별로 분해한 다음 해체된 코드 덩어리들을 각 덩어리의 의도를 살린 이름의 함수 호출로 바꿔주자.

### 절차

1. 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출한다

### 예시

```jsx
// ASIS
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
    charge = quantity * plan.summerRate;
else
    charge = quantity * plan.regularRate + plan.regularServiceCharge;
// TOBE
if (summer())
    charge = summerCharge();
else 
    charge = regularCharge();

// 예시 여름철이면 할인율이 달라지는 서비스 요금 계산
// ASIS

if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
    charge = quantity * plan.summerRate;
else
    charge = quantity * plan.regularRate + plan.regularServiceCharge;

// Step 1 조건 부분을 별도 함수로 추출
if (summer())
    charge = quantity * plan.summerRate;
else
    charge = quantity * plan.regularRate + plan.regularServiceCharge;

function summer() {
    return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd)
}

// Step 2 조건을 만족했을 때의 로직도 함수로 추출
if (summer())
    charge = summerCharge()
else
    charge = quantity * plan.regularRate + plan.regularServiceCharge;

function summer() {
    return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd)
}

function summerCharge() {
    return quantity * plan.summerRate;
}

// Step 3 마지막 else 절도 함수로 추출
if (summer())
    charge = summerCharge()
else
    charge = regualrCharge()

function summer() {
    return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd)
}

function summerCharge() {
    return quantity * plan.summerRate;
}

function regualrCharge() {
    return quantity * plan.regularRate + plan.regularServiceCharge;
}

// Step 4 취향에 따라 조건문을 3항 연산자로 변경
charge = summer() ? summerCharge() : regualrCharge()

function summer() {
    return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd)
}

function summerCharge() {
    return quantity * plan.summerRate;
}

function regualrCharge() {
    return quantity * plan.regularRate + plan.regularServiceCharge;
}
```

## 10.2 조건식 통합하기 Consolidate Conditional Expression

### 배경

비교하는 조건은 다르지만 그 결과로 수행하는 동작은 똑같은 코드들이 있다.

같은 일을 할 거라면 조건 검사도 하나로 통합하는게 낫다.

and, or 연산자를 사용하면 여러 개의 비교 로직을 하나로 합칠 수 있다.

조건부 코드를 통합하는게 중요한 이유는 두 가지다.

첫째. 여러 조각으로 나뉘 조건들을 하나로 통합함으로써 내가 하려는 일이 더 명확해진다.

둘째 통합 작업이 함수 추출하기로 이어질 가능성이 있다.

단 진짜로 독립된 검사들이라고 판단되면 이 리팩터링을 하면 안된다.

### 절차

1. 해당 조건식들 모두에 부수효과가 없는지 확인한다.
  1. 부수효과가 있는 조건식들에는 질의 함수와 변수 함수 분리하기를 먼저 적용한다.
2. 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다.
  1. 순차적으로 이뤄지는(레벨이 같은) 조건문은 or로 결합하고, 중첩된 조건문은 and로 결합한다.
3. 테스트한다.
4. 조건이 하나만 남을 때까지 2~3번을 반복한다.
5. 하나로 합쳐진 조건식을 함수로 추출할지 고려해본다.

### 예시

```jsx
// ASIS
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;

// TOBE
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisability() {
    return ((anEmployee.seniority < 2) || (anEmployee.monthsDisabled > 12) || (anEmployee.isPartTime));
}

// 예시
// or 사용하기
// ASIS
function disabilityAmount(anEmployee) {
    if (anEmployee.seniority < 2) return 0;
    if (anEmployee.monthsDisabled > 12) return 0;
    if (anEmployee.isPartTime) return 0;
    // 장애 수당 계산
}

// Step 1 결과로 행하는 동작이 같으므로 이 조건들을 하나의 식으로 통합
function disabilityAmount(anEmployee) {
    if((anEmployee.seniority < 2)
        || (anEmployee.monthsDisabled > 12)) return 0;
    if (anEmployee.isPartTime) return 0;
    // 장애 수당 계산
}

// Step 2 테스트한 후 다음 조건에도 적용
function disabilityAmount(anEmployee) {
    if((anEmployee.seniority < 2)
        || (anEmployee.monthsDisabled > 12)
        || (anEmployee.isPartTime)) return 0;
    // 장애 수당 계산
}

// Step 3 모든 조건을 통합했다면 최종 조건식을 함수로 추출
function disabilityAmount(anEmployee) {
    if (isNotEligibleForDisability) return 0;
    //장애 수당 계산

    function isNotEligibleForDisability() { //장애 수당 적용 여부 확인
        return ((anEmployee.seniority < 2)
        || (anEmployee.monthsDisabled > 12)
        || (anEmployee.isPartTime));
    }
}

// 예시 2
// and 사용하기
// ASIS
if (anEmployee.onVacation)
    if (anEmployee.seniority > 10)
        return 1;
return 0.5;

// Step 1
if ((anEmployee.onVacation) && (anEmployee.seniority > 10)) return 1;
    return 0.5
```

## 10.3 중첩 조건문을 보호 구문으로 바꾸기 Replace Nested Conditional With Guard Clauses

### 배경

조건문은 주로 두 가지 형태로 쓰인다.

참인 경로와 거짓인 경로 모두 정상 동작으로 이어지는 형태와, 한쪽만 정상인 형태다.

두 형태는 의도하는 바가 서로 다르므로 그 의도가 코드에 드러나야 한다.

두 경로 모두 정상 동작이라면 if와 else절을 사용한다.

한쪽만 정상이라면 비정상 조건을 if에서 검사한 다음, 조건이 참이면(비정상이면) 함수에서 빠져나온다.

중첩 조건문을 보호 구문으로 바꾸기의 핵심은 의도를 부각하는데 있다.

보호 구문은 “이건 이함수의 핵심이 아니다. 이 일이 일어나면 무언가 조치를 취한 후 함수에서 빠져나온다”라고 이야기한다.

함수의 진입점이 하나라는 것은 최신 프로그래밍 언어에서 강제하지만 반환점이 하나여야 한다는 규칙은 유용하지 않으니 명확한 코드로 구성하자.

### 절차

1. 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 바꾼다.
2. 테스트한다.
3. 1~2과정을 필요한 만큼 반복한다.
4. 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합한다.

### 예시

```jsx
// ASIS
function getPayAmount() {
    let result;
    if (isDead)
        result = deadAmount();
    else {
        if (isSeparated)
            retult = separatedAmount();
        else {
            if (isRetired)
                result = retiredAmount();
            else
                result = normalPayAmount();
        }
    }
    return result
}
// TOBE
function getPayAmount() {
    if (isDead) return deadAmount()
    if (isSeparated) return separatedAmount();
    if (isRetired) return retiredAmount();
    return normalPayAmount();
}

// 예시
// 직원 급열를 계산하는 코드
// ASIS
function payAmount(employee) {
    let result;
    if(employee.isSeparated) { // 퇴사한 직원인가 ?
        result = {amount: 0, reasonCode: "SEP" };
    }
    else {
        if (employee.isRetired) { // 은퇴한 직원인가 ?
            result = {amount: 0, reasonCode: "RET"};
        }
        else {
            //급여 계산 로직
            lorem.ipsum(dolor.sitAmet);
            consectetur(adipiscing).elit();
            sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
            ut.enim.ad(minim.veniam);
            result = someFinalComputation();
        }
    }
    return result;
} 

// Step 1 최상위 조건 부터 보호 구문으로 변경
function payAmount(employee) {
    let result;
    if (employee.isSeparated) return {amount: 0, reasonCode: "SEP" };
    if (employee.isRetired) {
        return = {amount: 0, reasonCode: "RET"};
    }
    else {
         //급여 계산 로직
         lorem.ipsum(dolor.sitAmet);
         consectetur(adipiscing).elit();
         sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
         ut.enim.ad(minim.veniam);
         result = someFinalComputation()
    }
    return result;
} 

// Step 2  다음 조건으로 넘어간다
function payAmount(employee) {
    let result;
    if (employee.isSeparated) return  {amount: 0, reasonCode: "SEP" };
    if (employee.isRetired) return {amount: 0, reasonCode: "RET"};
    else {
         //급여 계산 로직
         lorem.ipsum(dolor.sitAmet);
         consectetur(adipiscing).elit();
         sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
         ut.enim.ad(minim.veniam);
         result = someFinalComputation()
    }
    return result;
} 
// Step 3 아무일도 하지 않는 result는 제거한다
function payAmount(employee) {
    if (employee.isSeparated) return  {amount: 0, reasonCode: "SEP" };
    if (employee.isRetired) return  {amount: 0, reasonCode: "RET"};
    //급여 계산 로직
    lorem.ipsum(dolor.sitAmet);
    consectetur(adipiscing).elit();
    sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
    ut.enim.ad(minim.veniam);
    return someFinalComputation()
    
} 

// 예시 2 조건 반대로 만들기
function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital > 0) {
        if ( anInstrument.interestRate > 0 && anInstrument.duration > 0) {
            result = (anInstrument.income / anInstrument.duration) * anInstrument.adjustmentFactor;
        }
    }
    return result;
}

// Step 1 첫 조건 보호구문으로 만든다.
function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital <= 0) return result;
    if (anInstrument.interestRate > 0 && anInstrument.duration > 0) {
        result = (anInstrument.income / anInstrument.duration) * anInstrument.adjustmentFactor;
    }
    return result;
}
// Step 2 not 연산자(!) 추가
function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital <= 0) return result;
    if (!(anInstrument.interestRate > 0 && anInstrument.duration > 0)) return result;
    result = (anInstrument.income / anInstrument.duration) * anInstrument.adjustmentFactor;
    return result;
}

// Step 3 간소화 not 제거
function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital <= 0) return result;
    if (anInstrument.interestRate <= 0 || anInstrument.duration <= 0) return result;
    result = (anInstrument.income / anInstrument.duration) * anInstrument.adjustmentFactor;
    return result;
}

// Step 4 같은 결과를 내는 조건식 통합
function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital <= 0 || anInstrument.interestRate <= 0 || anInstrument.duration <= 0) return result;
    result = (anInstrument.income / anInstrument.duration) * anInstrument.adjustmentFactor;
    return result;
}
// Step 5 result 한 가지 일만 하도록 
function adjustedCapital(anInstrument) {
    if (anInstrument.capital <= 0 || anInstrument.interestRate <= 0 || anInstrument.duration <= 0) return 0;
    return (anInstrument.income / anInstrument.duration) * anInstrument.adjustmentFactor;
}
```

## 10.4 조건부 로직을 다형성으로 바꾸기 Replace Conditional With Polymorphism

### 배경

조건부 로직은 프로그래밍에서 해석하기 가장 난해한 대상에 속한다.

더 높은 수준의 개념을 도입해 이 조건들을 분리해낼 수 있다.

클래스와 다형성을 이용하면 더 확실하게 분리할 수 있다.

타입을 여러 개 만들고 각 타입이 조건부 로직을 자신만의 방식으로 처리하도록 구성하는 방법이 있다.

타입이 다른 경우 케이스 별로 클래스를 하나씩 만들어 공통 switch 로직의 중복을 없앨 수 있다.

기본 동작을 위한 case문과 변형 동작으로 구성된 로직을 만들 수 있다.

가장 직관적인 동작을 기본동작으로 슈퍼 클래스에 넣고 변형 동작을 뜻하는 case들을 각각의 서브 클래스로 만든다.

다형성은 유용하지만 남발해서는 안된다.

### 절차

1. 다형적 동작을 표현하는 클래스들이 아직 없다면 만들어준다. 이왕이면 적합한 인스턴스를 알아서 만들어 반환하는 팩터리 함수도 함께 만든다.
2. 호출하는 코드에서 팩터리 함수를 사용하게 한다.
3. 조건부 로직 함수를 슈퍼클래스로 옮긴다.
  1. 조건부 로직이 온전한 함수로 분리되어 있지 않다면 먼저 함수로 추출한다.
4. 서브클래스 중 하나를 선택한다. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다. 조건부 문장 중 선택된 서브클래스에 해당하는 조건절을 서브클래스 메서드로 복사한 다음 적절히 수정한다.
5. 같은 방식으로 각 조건절을 해당 서브클래스에서 메서드로 구현한다.
6. 슈퍼클래스 메서드에는 기본 동작 부분만 남긴다. 혹은 슈퍼클래스가 추상 클래스여야 한다면, 이 메서드를 추상으로 선언하거나 서브클래스에서 처리해야 함을 알리는 에러를 던진다.

### 예시 

새의 종에 따란 비행 속도와 깃털 상태를 알려주는 프로그램

```jsx
// ASIS
switch (bird.type) {
    case '유럽 제비':
        return '보통이다'
    case '아프리카 제비':
        return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다"
    case '노르웨이 파랑 앵무':
        return (bird.voltage > 100) ? "그을렸다" : "예쁘다"
    default:
        return "알 수 없다"
}

// TOBE
class EuropeanSwallow {
    get plumage() {
        return "보통이다"
    }
}

class AfricanSwallow {
    get plumage() {
        return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다"
    }
}

class NorwegianBlueParrot {
    get plumage() {
        return (bird.voltage > 100) ? "그을렸다" : "예쁘다"
    }
}

// 예시
// ASIS
function plumages(birds) {
    return new Map(birds.map(b => [b.name, plumage(b)]));
} 

function speeds(birds) {
    return new Map(birds.map(b => [b.name, airSpeedVelocity(b)]));
} 

function plumage(bird) { // 깃털 상태
    switch(bird.type) {
        case '유럽 제비':
            return '보통이다'
        case '아프리카 제비':
            return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다"
        case '노르웨이 파랑 앵무':
            return (bird.voltage > 100) ? "그을렸다" : "예쁘다"
        default:
            return "알 수 없다"
    }
}

function airSpeedVelocity(bird) { // 비행 속도
    switch(bird.type) {
        case '유럽 제비':
            return 35
        case '아프리카 제비':
            return 40 - 2 * bird.numberOfCoconuts
        case '노르웨이 파랑 앵무':
            return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
        default:
            return null;
    }
}

// TOBE
// Step 1 airSpeedVelocity 와 plumage를 Bird라는 클래스로 묶기
function plumage(bird) {
    return new Bird(bird).plumage
}

function airSpeedVelocity(bird) {
    return new Bird(bird).airSpeedVelocity;
}

class Bird {
    constructor(birdObject) {
        Object.assign(this, birdObject);
    }

    get plumage() { // 깃털 상태
        switch(this.type) {
            case '유럽 제비':
                return '보통이다'
            case '아프리카 제비':
                return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다"
            case '노르웨이 파랑 앵무':
                return (bird.voltage > 100) ? "그을렸다" : "예쁘다"
            default:
                return "알 수 없다"
    }
}

    get airSpeedVelocity() { // 비행 속도
        switch(this.type) {
            case '유럽 제비':
                return 35
            case '아프리카 제비':
                return 40 - 2 * bird.numberOfCoconuts
            case '노르웨이 파랑 앵무':
                return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
            default:
                return null;
    }
}

// Step 2 종별 서브 클래스 생성

function plumage(bird) {
    return createBird(bird).plumage
}

function airSpeedVelocity(bird) {
    return createBird(bird).airSpeedVelocity
}

function createBird(bird) {
    switch (bird.type) {
        case '유럽 제비':
            return new EuropeanSwallow(bird);
        case '아프리카 제비':
            return new AfricanSwallow(bird);
        case '노르웨이 파랑 앵무':
            return new NorwegianBlueParrot(bird);
        default:
            return new Bird(bird);
    }
}

class EuropeanSwallow extends Bird {

}

class AfricanSwallow extends Bird {

}

class NorwegianBlueParrot extends Bird {

}

// Step 3 switch 문의 절 하나를 선택해 해당 서브클래스에서 오버라이드
class EuropeanSwallow extends Bird {
    get plumage() {
        return "보통이다"
    }
}

class Bird {
    get plumage() {
        switch(this.type) {
            case '유럽 제비':
                throw "오류 발생";
            case '아프리카 제비':
                return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다"
            case '노르웨이 파랑 앵무':
                return (this.voltage > 100) ? "그을렸다" : "예쁘다";
            default:
                return "알 수 없다";
        }
    }
}

// Step 4 조건절 처리
class AfricanSwallow extends Bird {
    get plumage() {
        return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    }
}

class NorwegianBlueParrot extends Bird {
    get plumage() {
        return (this.voltage > 100) ? "그을렸다": " 예쁘다";
    }
}

// Step 5 슈퍼클래스의 메서드는 기본 동작만 남긴다
class Bird{
    get plumage() {
        return "알 수 없다";
    }
}

// Step 6 똑같은 과정을 airSpeedVelocity()에도 수행한다.
function plumages(birds) {
    return new Map(birds
        .map(b => createBird(b))
        .map(bird => [bird.name, bird.plumage]));
}

function speeds(birds) {
    return new Map(birds
        .map(b => createBird(b))
        .map(bird => [bird.name, bird.airSpeedVelocity]));
}

function createBird(bird) {
    switch (bird.type) {
        case '유럽 제비':
                return new EuropeanSwallow(bird);
        case '아프리카 제비':
            return new AfricanSwallow(bird);
        case '노르웨이 파랑 앵무':
            return new NorwegianBlueParrot(bird);
        default:
            return Bird(bird);
    }
}

class Bird {
    constructor(birdObject) {
        Object.assign(this, birdObject);
    }
    
    get plumage() {
        return "알 수 없다"
    }

    get airSpeedVelocity() {
        return null
    }
}

class EuropeanSwallow extends Bird {
    get plumage() {
        return "보통이다"
    }

    get airSpeedVelocity() {
        return 35;
    }
}

class AfricanSwallow extends Bird {
    get plumage() {
        return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    }
    get airSpeedVelocity() {
        return 40 - 2 * this.numberOfCoconuts;
    }
}

class NorwegianBlueParrot extends Bird {
    get plumage() {
        return (this.voltage > 100) ? "그을렸다" : "예쁘다";
    }
    get airSpeedVelocity() {
        return (this.isNailed) ? 0 : 10 + this.voltage / 10;
    }
}

// Bird는 없어도 되면 없어도 된다 자바스크립트에서는 타입 계층 구조 없이도 다형성을 표현할 수 있다.
// 객체가 적절한 이름의 메서드만 구현하고 있다면 아무 문제없이 같은 타입으로 취급하기 때문 (이를 덕 타이핑duck typing) 
```

### 예시 2

```jsx
// 예시 2 
// 변형 동작을 다형성으로 표현하기
// 신용 평가 기관에서 선박의 항해 투자 등급을 계산하는 코드
// 평가기관은 위험요소와 잠재 수익에 영향을 주는 다양한 요인을 기초로 향해 등급을 A와 B로 나눈다
// 위험요소로는 항해 경로의 자연조건과 선장의 항해 이력을 고려한다
// ASIS
function rating(voyage, history) // 투자 등급
{
    const vpf = voyageProfitFactor(voyage, history);
    const vr = voyageRisk(voyage);
    const chr = captainHistoryRisk(voyage, history);
    if (vpf * 3 > (vr + chr * 2)) return "A";
    else return "B";
}

function voyageRisk(voyage) { // 향해 경로 위험요소
    let result = 1;
    if (voyage.length > 4) result += 2;
    if (voyage.length > 8) result += voyage.length - 8;
    if(["중국", "동인도"].includes(voyage.zone)) result += 4;
    return Math.max(result, 0);
}

function captainHistoryRisk(voyage, history) { // 선장의 향해 이력 위험요소
    let result = 1;
    if (history.length < 5) result += 4;
    result += history.filter(v => v.profit < 0).length;
    if (voyage.zone == "중국" && hasChina(history)) result -= 2;
    return Math.max(result, 0);
}

function hasChina(history) { // 중국을 경유하는가?
    return history.some(v => "중국" === v.zone);
}

function voyageProfitFactor(voyage, history) { //수익 요인
    let result = 2;
    if (voyage.zone === "중국") result += 1;
    if (voyage.zone === "동인도") result += 1;
    if (voyage.zone === "중국" && hasChina(history)) {
        result += 3;
        if (history.length > 10) result += 1;
        if (voyage.length > 12) result += 1;
        if (voyage.length > 18) result -= 1;
    }
    else {
        if (history.length > 8) result += 1;
        if (voyage.length > 14) result -= 1;
    }
    return result;
}
// voyageRisk() 와 captainHistoryRisk() 함수의 점수는 위험요소에, voyageProfitFactor() 점수는 잠재 수익에 반영된다.
// rating() 함수는 두 값을 종합하여 요청한 항해의 최종 등급을 계산한다.
// 호출하는 코드
const voyage = {zone: "서인도", length: 10};
const history = [
    {zone: "동인도", profit: 5},
    {zone: "서인도", profit: 15},
    {zone: "중국", profit: -2},
    {zone: "서아프리카", profit: 7},
];

const myRating = rating(voyage, history);

// Step 1 여러 함수를 클래스로 묶기
function rating(voyage, history) {
    return new rating(voyage, history).value;
}

class Rating {
    constructor(voyage, history) {
        this.voyage = voyage;
        this.history = history;
    }

    get value() {
        const vpf = this.voyageProfitFactor;
        const vr = this.voyageRisk;
        const chr = this.captainHistoryRisk;
        if (vpf * 3 > (vr + chr * 2)) return "A";
        else return "B";
    }

    get voyageRisk() {
        let result = 1;
        if (this.voyage.length > 4) result += 2;
        if (this.voyage.length > 8) result += this.voyage.length - 8;
        if(["중국", "동인도"].includes(this.voyage.zone)) result += 4;
        return Math.max(result, 0);
    }

    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        if (this.voyage.zone == "중국" && this.hasChina(history)) result -= 2;
        return Math.max(result, 0);
    }

    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        if (this.voyage.zone === "중국" && this.hasChina(history)) {
            result += 3;
            if (this.history.length > 10) result += 1;
            if (this.voyage.length > 12) result += 1;
            if (this.voyage.length > 18) result -= 1;
        }
        else {
            if (this.history.length > 8) result += 1;
            if (this.voyage.length > 14) result -= 1;
        }
        return result;
    }

    get hasChinaHistory() {
        return this.history.some(v => "중국" === v.zone);
    }
}

// Step 2  변형 동작을 담을 빈 서브 클래스 만들기
class ExperienceChinaRating extends Rating {

}
// 변형 클래스 반환하는 팩터리 함수 만들기
function createRating(voyage, history) {
    if (voyage.zone === "중국" && history.some(v => "중국" === v.zone))
        return new ExperienceChinaRating(voyage, history);
    else return new Rating(voyage, history);
}

// Step 3 생성자를 호출하는 코드를 팩터리 함수로 수정
function rating(voyage, history) {
    return createRating(voyage, history).value
}

// Step 4 서브 클래스로 captainHistoryRisk() 옮기기
class Rating {
    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        if (this.voyage.zone == "중국" && this.hasChinaHistory(history)) result -= 2;
        return Math.max(result, 0);
    }
}

// Step 5 서브 클래스에서 메서드 오버라이딩
class ExperienceChinaRating extends Rating { //서브 클래스에서 오버라이딩을 처리
    get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
    }
}

class Rating {
    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        //제거
        return Math.max(result, 0);
    }
}

// Step 6 voyageProfitFactor() 옮기기
// 조건부 블록 전체를 함수로 추출
class Rating {
    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        //제거
        return Math.max(result, 0);
    }

    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.voyageAndHistoryLengthFactor; // 조건부 자체를 함수로 추출
        return result;
    }

    get voyageAndHistoryLengthFactor() { // 추출한 함수
        let result = 0;

        if (this.voyage.zone === "중국" && this.hasChinaHistory(history)) {
            result += 3;
            if (this.history.length > 10) result += 1;
            if (this.voyage.length > 12) result += 1;
            if (this.voyage.length > 18) result -= 1;
        }
        else {
            if (this.history.length > 8) result += 1;
            if (this.voyage.length > 14) result -= 1;
        }
        return result;
    }
}
// Step 7 전체 클래스 
class Rating {
    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        //제거
        return Math.max(result, 0);
    }

    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.voyageAndHistoryLengthFactor; // 조건부 자체를 함수로 추출
        return result;
    }

    get voyageAndHistoryLengthFactor() {  //서브 클래스로
        let result = 0;
        if (this.history.length > 8) result += 1;
        if (this.voyage.length > 14) result -= 1;
        return result;
    }
}

class ExperienceChinaRating extends Rating {
    get voyageAndHistoryLengthFactor() { // 서브 클래스에 로직 분리
        let result = 0;
        result += 3;
        if (this.history.length > 10) result += 1;
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
    }

    get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
    }
}
```

### 더 가다듬기

```jsx
// 더 가다듬기
// 변형 동작은 슈퍼클래스와의 차이를 표현해야 하는 서브클래스에서만 신경 쓰면 된다.
// And 라는 이름의 두 가지 일을 하는 함수에서 이력 길이를 수정하는 부분을 함수로 추출
// Step 1
class Rating {
    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        return Math.max(result, 0);
    }

    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.voyageAndHistoryLengthFactor; 
        return result;
    }

    get voyageAndHistoryLengthFactor() {
        let result = 0;
        result += this.historyLengthFactor; // 길이 수정하는 부분 함수로 추출
        if (this.voyage.length > 14) result -= 1;
        return result;
    }

    get historyLengthFactor() { // 함수로 추출
        return (this.history.length > 0) ? 1 : 0;
    }
}

class ExperienceChinaRating extends Rating {
    get voyageAndHistoryLengthFactor() {
        let result = 0;
        result += 3;
        result += this.historyLengthFactor; // 서브 클래스에서도 동일하게 리팩터링 함수로 추출
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
    }

    get historyLengthFactor() { // 서브 클래스에서도 빼내줌
        return (this.history.length > 10) ? 1 : 0;
    }

    get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
    }
}

// Step 2 슈퍼 클래스에서 문장을 호출한 곳으로 옮기기 적용
class Rating {
    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        //제거
        return Math.max(result, 0);
    }

    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.historyLengthFactor; // voyageAndHistoryLengthFactor에서 옮겨짐 
        result += this.voyageAndHistoryLengthFactor; 
        return result;
    } 

    get voyageAndHistoryLengthFactor() {
        let result = 0;
        // 제거 
        if (this.voyage.length > 14) result -= 1;
        return result;
    }

    get historyLengthFactor() { // 함수로 추출
        return (this.history.length > 0) ? 1 : 0;
    }
}

class ExperienceChinaRating extends Rating {
    get voyageAndHistoryLengthFactor() {
        let result = 0;
        result += 3;
        // 제거 
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
    }

    get historyLengthFactor() { // 서브 클래스에서도 빼내줌
        return (this.history.length > 10) ? 1 : 0;
    }

    get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
    }
}

// Step 3 함수 이름 변경
class Rating {
    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        //제거
        return Math.max(result, 0);
    }

    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.historyLengthFactor; 
        result += this.voyageLengthFactor; // 함수 이름 변경
        return result;
    } 

    get voyageLengthFactor() { // 함수 이름 변경
        return (this.voyage.length > 14) ? -1 : 0; // 삼항 연산자로 간소화
    }

    get historyLengthFactor() {
        return (this.history.length > 0) ? 1 : 0;
    }
}

class ExperienceChinaRating extends Rating {
    get voyageLengthFactor() {
        let result = 0;
        result += 3; // 요기
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
    }

    get historyLengthFactor() { 
        return (this.history.length > 10) ? 1 : 0;
    }

    get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
    }
}

// Step 4 항해 거리 요인 계산할 때 3점을 더하는 로직 전체 결과를 계산하는 voyageProfitFactor 로 이동
class Rating {
    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        //제거
        return Math.max(result, 0);
    }

    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.historyLengthFactor; 
        result += this.voyageLengthFactor; // 함수 이름 변경
        return result;
    } 

    get voyageLengthFactor() { // 함수 이름 변경
        return (this.voyage.length > 14) ? -1 : 0; // 삼항 연산자로 간소화
    }

    get historyLengthFactor() {
        return (this.history.length > 0) ? 1 : 0;
    }
}

class ExperienceChinaRating extends Rating {
    get voyageProfitFactor() {
        return super.voyageProfitFactor + 3; // 이동
    }
    get voyageLengthFactor() {
        let result = 0;
        // 제거
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
    }

    get historyLengthFactor() { 
        return (this.history.length > 10) ? 1 : 0;
    }

    get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
    }
}

// TOBE
class Rating {
    constructor(voyage, history) {
        this.voyage = voyage;
        this.history = history;
    }

    get value() {
        const vpf = this.voyageProfitFactor;
        const vr = this.voyageRisk;
        const chr = this.captainHistoryRisk;
        if (vpf * 3 > (vr + chr * 2)) return "A";
        else return "B";
    }

    get voyageRisk() {
        let result = 1;
        if (this.voyage.length > 4) result += 2;
        if (this.voyage.length > 8) result += this.voyage.length - 8;
        if(["중국", "동인도"].includes(this.voyage.zone)) result += 4;
        return Math.max(result, 0);
    }

    get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter(v => v.profit < 0).length;
        return Math.max(result, 0);
    }

    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.historyLengthFactor; 
        result += this.voyageLengthFactor; // 함수 이름 변경
        return result;
    } 

    get voyageLengthFactor() { // 함수 이름 변경
        return (this.voyage.length > 14) ? -1 : 0; // 삼항 연산자로 간소화
    }

    get historyLengthFactor() {
        return (this.history.length > 0) ? 1 : 0;
    }
}
// 중국 항해 경험 클래스는 기본 클래스와의 차이만 담고 있다.
class ExperienceChinaRating extends Rating {
    get voyageProfitFactor() {
        return super.voyageProfitFactor + 3; // 이동
    }
    get voyageLengthFactor() {
        let result = 0;
        // 제거
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
    }

    get historyLengthFactor() { 
        return (this.history.length > 10) ? 1 : 0;
    }

    get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
    }
}
```

## 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}