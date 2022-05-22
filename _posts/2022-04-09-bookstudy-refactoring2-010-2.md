---
layout: single
title: 리팩터링2 스터디 - 010 - 2
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 10장 조건부 로직 간소화

## 10.5 특이 케이스 추가하기 Introduce Special Case

## 배경

특정 값을 확인한 후 똑같은 동작을 수행하는 코드가 곳곳에 있으면 중복 코드이다.

특정 값에 대한 똑같이 반응하는 코드가 여러 곳이라면 그 반응들을 한 데로 모으는 게 효율적이다.

공통 동작을 요소 하나에 모아서 사용하는 특이 케이스 패턴(Special Case Pattern)이라는 것이 있는데, 이럴 때 적용하면 좋은 메커니즘이다.

이 패턴을 사용하면 특이 케이스를 확인하는 코드 대부분을 단순한 함수 호출로 바꿀 수 있다.

널(null)은 특이 케이스로 처리해야 할 때가 많다. 그래서 이 패턴을 널 객체 패턴(Null Object Pattern)이라고도 한다.
## 절차

1. 컨테이너에 특이 케이스인지를 검사하는 속성을 추가하고, false를 반환하게 한다.
2. 특이 케이스 객체르 만든다. 이 객체는 특이 케이스인지를 검사하는 속성만 포함하며, 이 속성은 true를 반환하게 한다.
3. 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출한다. 모든 클라이언트가 값을 직접 비교하는 대신 방금 추출한 함수를 사요하도록 고친다.
4. 코드에 새로운 특이 케이스 대상을 추가한다. 함수의 반환 값으로 받거나 변환 함수를 적용하면 된다.
5. 특이 케이스를 검사하는 함수 본문을 수정하여 특이 케이스 객체의 속성을 사용하도록 한다.
6. 테스트한다.
7. 여러 함수를 클래스로 묶기나 여러 함수를 변환 함수로 묶기를 적용하여 특이 케이스를 처리하는 공통 동작을 새로운 요소로 옮긴다.
  1. 특이 케이스 클래스는 간단한 요청에는 항상 같은 값을 반환하는 게 보통이므로, 해당 특이 케이스의 리터럴 레코드를 만들어 활용할 수 있을 것이다.
8. 아직도 특이 케이스 검사 함수를 이용하는 곳이 남아 있다면 검사 함수를 인라인한다.

## 예시

```jsx
// ASIS
if (aCustomer === "미확인 고객") customerName = '거주자';
// TOBE
class UnknownCustomer {
    get name() { return '거주자'; }
}

// 예시
// 전력 회사는 전력이 필요한 현장에 인프라를 설치해 서비스를 제공한다.
class Site {
    get customer() { return this._customer; }
}

class Customer {
    get name() { return this._name; } // 고객 이름
    get bilingPlan() { return this._billingPlan; } // 요금제
    set bilingPlan(arg) { this._billingPlan = arg } 
    get paymentHistory() { return this._paymentHistory; } // 납부 이력
}

// 클라이언트 1
const aCustomer = site.customer;

let customerName;
if (aCustomer === '미확인 고객') customerName = '거주자';
else customerName = aCustomer.name;

// 클라이언트 2
const plan = (aCustomer === '미확인 고객') ? 
registry.billingPlans.basic : aCustomer.billingPlan;

// 클라이언트 3
if (aCustomer !== '미확인 고객') aCustomer.bilingPlan = newPlan;

// 클라이언트 4
const weeksDelinquent = (aCustomer === '미확인 고객') ?
0 : aCustomer.paymentHistory.weeksDelinquentInLastYear;

// 미확인 고객을 처리하는 클라이언트가 여러개 발견, 그 대부분을 동일한 방식으로 처리
// 고객 이름으로는 거주자를 사용했고, 기본 요금제(billingPlan)을 청구했고, 연체(delinquent) 기간은 0주(week)로 분류
// Step 1 미확인 고객인지를 나타내는 메서드를 고객 클래스에 추가
class Customer {
    get name() { return this._name; } 
    get bilingPlan() { return this._billingPlan; } 
    set bilingPlan(arg) { this._billingPlan = arg } 
    get paymentHistory() { return this._paymentHistory; }

    get isUnknown() { return false; }
}
// Step 2 미확인 고객 전용 클래스 생성
// 자바스크립트의 서브클래스 규칙과 동적 타이핑 능력 때문에 서브 클래스로 만들지 않았음
class UnknownCustomer {
    get isUnknown() { return true; }
}
// Step 3 미확인 고객을 기대하는 곳 모두에 특이 케이스 객체(UnknownCustomer)를 반환하도록하고 값이 미확인 고객인지를 검사하는 곳 모두에서 isUnknown() 메서드 호출
// Customer 클래스를 수정하여 미확인 고객 문자열 대신 UnknownCustomer 객체를 반환하게 한다면, 클라이언트를 각각에서 미확인 고객인지를 확인하는 코드 모두를 isUnknown() 호출로 바꾸는 작업을 한 번에 해야만 한다.
// 여러 곳에서 똑같이 수정해야만 하는 코드를 별로 함수로 추출하여 한데로 모으자
function isUnknown(arg) {
    if (!((arg instanceof Customer) || (arg === '미확인 고객'))) 
        throw new Error(`잘못된 값과 비교: <${arg}>`);
    return (arg === '미확인 고객');
}

// Step 4 isUnknown 함수 적용
// 클라이언트 1

let customerName;
if (isUnknown(aCustomer)) customerName = '거주자'; // 적용
else customerName = aCustomer.name;

// 클라이언트 2
const plan = (isUnknown(aCustomer)) ?  // 적용
registry.billingPlans.basic : aCustomer.billingPlan;

// 클라이언트 3
if (!(isUnknown(aCustomer))) aCustomer.bilingPlan = newPlan; // 적용

// 클라이언트 4
const weeksDelinquent = (isUnknown(aCustomer)) ?    // 적용
0 : aCustomer.paymentHistory.weeksDelinquentInLastYear;

// Step 5 특이 케이스 일 때 Site 클래스가 UnknownCustomer 객체를 반환하도록 수정
class Site {
    get customer() {
        return (this._customer === '미확인 고객') ? new UnknownCustomer() : this._customer;
    }
}

// Step 6 isUnknown() 함수를 수정하여 고객 객체의 속성을 사용하도록 수정
function isUnknown(arg) {
    if (!(arg instanceof Customer || arg instanceof UnknownCustomer))
        throw new Error(`잘못된 값과 비교: <${arg}>`);
    return arg.isUnknown;
}
// Step 7 동일하게 사용하는 거주자를 클래스로 묶기에 적용
class UnknownCustomer {
    get isUnknown() { return true; }

    get name() { return '거주자' };
}

//클라이언트 1
let customerName = aCustomer.name; // 조건절 지워버리고 함수로 대체

// Step 8 요금제 속성도 클래스 메서드로 전환
class UnknownCustomer {
    get isUnknown() { return true; }

    get name() { return '거주자' };
    get billingPlan() { return registry.billingPlan.basic; }
    set billingPlan(arg) { /* 무시한다 */ }
}
// ASIS
// 클라이언트 2
const plan = (isUnknown(aCustomer)) ?  // 적용
registry.billingPlans.basic : aCustomer.billingPlan;

// 클라이언트 3
if (!(isUnknown(aCustomer))) aCustomer.bilingPlan = newPlan; // 적용

// TOBE
// 클라이언트(읽는 경우)
const plan = aCustomer.bilingPlan;

// 클라이언트(쓰는 경우)
aCustomer.bilingPlan = newPlan

// Step 9
// 특이 케이스 값 객체는 항상 불변이어야 한다.
// 특이 케이스가 다른 객체를 반환해야 한다면 그 객체 역시 특이 케이스여야 하는 것이 일반적이다. NullPaymentHistory를 만든다

class UnknownCustomer {
    get paymentHistory() { return new NullPaymentHistory(); }
}

class NullPaymentHistory {
    get weeksDelinquentInLastYear() { return 0; }
}

// 클라이언트 4
// ASIS
const weeksDelinquent = (isUnknown(aCustomer)) ?    // 적용
0 : aCustomer.paymentHistory.weeksDelinquentInLastYear;

//TOBE
const weeksDelinquent = aCustomer.paymentHistory.weeksDelinquentInLastYear;

// Step 10 튀는 클라이언트 확인
// 원래의 특이 케이스 검사 코드 유지를 해야한다. Customer에 선언된 메서드를 사용하도록 수정
// ASIS
const name = !isUnknown(aCustomer) ? aCustomer.name : '미확인 거주자';
// TOBE
const name = aCustomer.isUnknown ? '미확인 거주자' : aCustomer.name;
```

## 10.6 어서션 추가하기 Introduce Assertion

## 배경

특정 조건이 참일 때만 제대로 동작하는 코드 영역이 있을 수 있다.

어서션(Assertion)을 이용하여 코드 자체에 삽입해 놓는 것이 좋은 방법이다.

어서션은 항상 참이라고 가정하는 조건부 문장으로, 어셔션이 실패했다는 건 프로그래머가 잘못했다는 뜻이다.

어서션 실패는 시스템의 다른 부분에서는 절대 검사하지 않아야 하며, 어서션이 있고 없고가 프로그램 기능에 정상 동작에 아무런 영향을 주지 않도록 작성해야 한다.

어서션을 오류 찾기에 활용하는것은 좋다.

어서션은 프로그램이 어떤 상태임을 가정한 채 실행되는지를 다른 개발자에게 알려주는 훌륭한 소통 도구이다.

단위 테스트를 꾸준히 추가하여 사각을 좁히면 어서션보다 나을 때가 많다.

하지만 소통 측명에서는 어서션이 매력적이다.
## 절차

1. 참이라고 가정하는 조건이 보이면 그 조건을 명시하는 어서션을 추가한다.
  1. 어서션은 시스템 운영에 영향을 주면 안 되므로 어서션을 추가한다고 해서 동작이 달라지지는 않는다.

## 예시

```jsx
// ASIS
if (this.discountRate)
{
    base = base - (this.discountRate * base)
}
// TOBE
assert(this.discountRate >= 0);
if (this.discountRate)
{
    base = base - (this.discountRate * base)
}

// 예시 1
class Customer {
    applyDiscount(aNumber) {
        return (this.discountRate)
        ? aNumber - (this.discountRate * aNumber)
        : aNumber;
    }
}
// Step 1 3항 표현식에는 어서션을 넣을 장소가 적당치 않으니, 먼저 if-then 문장으로 재구성
class Customer {
    applyDiscount(aNumber) {
        if (!this.discountRate) return aNumber;
        else return aNumber - (this.discountRate * aNumber);
    }
}

// Step 2 어서션 추가
class Customer {
    applyDiscount(aNumber) {
        if (!this.discountRate) return aNumber;
        else {
            assert(this.discountRate >= 0); // 양수만 허용
            return aNumber - (this.discountRate * aNumber);
        }
    }
}

// Step 3 서드 파티로 어서션 이동, 어서션이 applyDiscount()에서 실패한다면 이 문제가 언제 처음 발생했는지를 찾는 문제를 다시 풀어야 하기 때문
class Customer {
    set discountRate(aNumber) {
        assert(null === aNumber || aNumber >=0);
        this._discountRate = aNumber;
    }
}
// 어서션을 남발하는 것은 위험하다. 반드시 참이어야 하는 것만 어서션을 사용해 검사하자
```

## 10.7 제어 플래그를 탈출문으로 바꾸기

## 배경

제어 플래그란 코드의 동작을 변경하는 데 사용하는 변수를 말하며, 어딘가에서 값을 계산해 제어 플래그에 설정한 후 다른 어딘가의 조건문에서 검사하는 형태로 쓰인다.

제어 플래그의 주 서식지는 반복문이다.

break문이나 continu문에 익숙하지 않거나, 함수의 return 문을 하나로 유지하고자 노력하는 사람이 주로 심는다.

함수에서 할 일을 다 마쳤다면, 그 사실을 return 문으로 명확히 알리는 편이 좋다.

## 절차

1. 제어 플래그를 사용하는 코드를 함수로 추출할지 고려한다.
2. 제어 플래그를 갱신하는 코드 각각을 적절한 제어문으로 바꾼다. 하나 바꿀 때마다 테스트한다.
  1. 제어문으로는 주로 return, break, continue가 쓰인다.
3. 모두 수정했다면 제어 플래그를 제거한다.

## 예시

```jsx
// ASIS
for (const p of people) {
    if(!found) {
        sendAlert();
        found = true;
    }
}
// TOBE
for (const p of people) {
    if(p === '조커'){
        sendAlert();
        break;
    }
}

// 예시
// ASIS
// 여기서 제어 플래그는 found이며, 제어 흐름을 변경하는 데 쓰인다.
let found = false;
for (const p of people) {
    if(!found) {
        if(p === '조커') {
            sendAlert();
            found = true;
        }
        if(p === '사루만'){
            sendAlert();
            found = true;
        }
    }
}

// Step 1 함수 추출하기를 활용해서 서로 밀접한 코드만 담은 함수를 뽑아내자
checkForMiscreants(people);

function checkForMiscreants(people) {
    let found = false;
    for (const p of people) {
        if(!found) {
            if(p === '조커') {
                sendAlert();
                found = true;
            }
            if(p === '사루만'){
                sendAlert();
                found = true;
            }
        }
    }
}
// Step 2플래그가 참이면 break나 return을 써서 함수에서 빠져나오면 된다. return을 사용해 보자
function checkForMiscreants(people) {
    let found = false;
    for (const p of people) {
        if(!found) {
            if(p === '조커') {
                sendAlert();
                return;
            }
            if(p === '사루만'){
                sendAlert();
                return;
            }
        }
    }
}
// Step 3 갱신 코드를 모두 제거했다면 제어 플래그를 참조하는 다른 코드도 모두 제거한다
function checkForMiscreants(people) {
    for (const p of people) {
        if(p === '조커') {
            sendAlert();
            return;
        }
        if(p === '사루만'){
            sendAlert();
            return;
        }
    }
    
}
```

## 더 가다듬기

일급함수로 변경한다.

```jsx
// Step 1
function checkForMiscreants(people) {
    if(people.some(p => ["조커", "사루만"].includes(p))) sendAlert();
}

// Step 2
function checkForMiscreants(people) {
   ['조커', '사루만'].isDisjointWith(people)
}
```


## 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}