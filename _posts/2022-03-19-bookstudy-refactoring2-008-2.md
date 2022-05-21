---
layout: single
title: 리팩터링2 스터디 - 008 - 2
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 8장 기능 이동

## 8.6 문장 슬라이드하기

### 배경

관련된 코드들이 가까이 모여 있다면 이해하기가 더 쉽다.

하나의 데이터 구조를 이용하는 문장들은 한데 모여 있어야 좋다.

문장 슬라이드하기 리팩터링으로 코드를 한데 모을 수 있다.

모든 변수 선언을 함수 첫머리에 모아두는 사람도 있는데, 저자는 변수를 처음 사용할 때 선언하는 스타일을 선호한다.

관련 코드끼리 모으는 작업은 다른 리팩터링(함수 추출하기)의 준비 단계로 자주 행해진다.

관련된 코드들을 명확히 구분하여 함수로 추출하는게 그저 문장들을 한데로 모으는 것보다 나은 분리법이다.

### 절차

1. 코드 조각(문장들)을 이동할 목표 위치를 찾는다. 코드 조각의 원래 위치와 목표 위치 사이의 코드들을 흝어보면서, 조각을 모으고 나면 동작이 달라지는 코드가 있는지 살핀다. 다음과 같은 간섭이 있으면 이 리팩터링을 포기한다.
  1. 코드 조각에서 참조하는 요소를 선언하는 문장 앞으로는 이동할 수 없다.
  2. 코드 조각을 참조하는 요소의 뒤로는 이동할 수 없다.
  3. 코드 조각에서 참조하는 요소를 수정하는 문장을 건너뛰어 이동할 수 없다.
  4. 코드 조각이 수정하는 요소를 참조하는 요소를 건너뛰어 이동할 수 없다.
2. 코드 조각을 원래 위치에서 잘라내어 목표 위치에 붙여 넣는다.
3. 테스트한다.

테스트가 실패한다면 더 작게 나눠 시도해보자. 이동 거리를 줄이는 방법과 한 번에 옮기는 조각의 크기를 줄이는 방법이 있다.

### 예시

코드 조각을 슬라이드할 떄는 두 가지를 확인해야 한다.

무엇을 슬라이스할지와 슬라이드할 수 있는지 여부이다.

무엇을 슬라이즈할지는 맥락과 관련이 깊다.

요소를 선언하는 곳과 사용하는 곳을 가까이 두기를 좋아하면 선언 코드를 슬라이드하여 처음 사용하는 곳까지 끌어내린다.

그 외에도 다른 리팩터링을 하기 위해서는 거의 항상 코드를 슬라이드하게 된다.

코드 슬라이드하기 위해서는 코드 자체와 그 코드가 건너뛰어야 할 코드를 모두 살펴야 한다.

```jsx
// ASIS
const pricingPlan = retrievePricingPlan();
const order = retrieveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;

// TOBE
const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retrieveOrder();
let charge;

// 예시
// ASIS
// 처음 7 까지의 줄은 선언이므로 이동하기가 상대적으로 쉽다.
const pricingPlan = retrievePricingPlan(); // 1                                      
const order = retreiveOrder(); // 2                                                 
const baseCharge = pricingPlan.base; // 3                                           
let charge; // 4                                                                    
const chargePerUnit = pricingPlan.unit; // 5                                        
const units = order.units; // 6                                                      
let discont; // 7                                                                    
charge = baseCharge + units * chargePerUnit;  // 8                                  
let discountableUnits = Math.max(units - pricingPlan.discountThreshold, 0); // 9     
discont = discountableUnits * pricingPlan.discountFactor; // 10                      
if (order.isRepeat) discount += 20; // 11                                            
charge = charge - discount; // 12                                                    
chargeOrder(charge); // 13                                                           

// Step 1 let discount 옮기기
// 선언은 부수효과가 없고 다른 변수를 참조하지도 않으므로 discount 자신을 참조하는 첫 번쨰 코드 바로 앞까지는 어디로든 옮겨도 안전하다.

const pricingPlan = retrievePricingPlan(); // 1                                     
const order = retreiveOrder(); // 2                                                 
const baseCharge = pricingPlan.base;  // 3                                          
let charge; // 4                                                                    
const chargePerUnit = pricingPlan.unit; // 5                                        
const units = order.units; // 6                                                     
charge = baseCharge + units * chargePerUnit; // 8                                  
let discountableUnits = Math.max(units - pricingPlan.discountThreshold, 0); // 9    
let discont;  // 7  discount 이동                                                                  
discont = discountableUnits * pricingPlan.discountFactor;  // 10                     
if (order.isRepeat) discount += 20; // 11                                            
charge = charge - discount; // 12                                                    
chargeOrder(charge); // 13                                                           

// Step 2 부수효과가 없는 다른코드에도 비슷한 분석을 수행하면 2번 줄도 6번 줄 바로 위까지 옮겨도 된다.
const pricingPlan = retrievePricingPlan();  // 1                                     
const baseCharge = pricingPlan.base; // 3                                           
let charge; // 4                                                                    
const chargePerUnit = pricingPlan.unit; // 5                                        
const order = retreiveOrder(); // 2 order 선언부 이동                                                 
const units = order.units;  // 6                                                    
charge = baseCharge + units * chargePerUnit; // 8                                   
let discountableUnits = Math.max(units - pricingPlan.discountThreshold, 0); // 9    
let discont; // 7  discount 이동                                                                    
discont = discountableUnits * pricingPlan.discountFactor; // 10                      
if (order.isRepeat) discount += 20; // 11                                             
charge = charge - discount; // 12                                                    
chargeOrder(charge); // 13                                                           

/*
// 명령-질의 분리 원칙(Command-Query Separation)을 지키면 값을 반환하는 함수는 모두 부수효과가 없음을 알 수 있다.
// 명령-질의 분리 원칙 이란?
// 함수는 호출할 떄 본의 아니게 발생한 외부 효과로 예상치 못한 결과가 나오는 일을 방지하는 데 기초가 되는 원칙이다.
// 함수는 그 성격에 따라 크게 두 가지로 분류할 수 있다.
// 하나는 어떤 동작을 수행하는 명령(Command)이고, 다른 하나는 답을 구하는 질의(Query)다.
// 이러한 두 역할은 한데 섞으면 안된다.
// 명령-질의 분리 원칙 예시
function getFirstName() {
    var firstName = document.querySelector("#firstName").value;
    firstName = firstName.toLowerCase();
    setCookie("firstName", firstName);
    if (firstName === null) {
        return "";
    }
    return firstName;
}
 
var activeFirstName = getFirstName();
// getFirstName 함수 이름은 질문에 답하는 질의형 함수 같지만, 정작 실제로 하는 일은 명령형 함수처럼 데이터의 상태를 변환하고 있다.
// 함수 이름에 명확히 드러나지 않은 외부 효과에 해당한다.
// 더 심각한 부분은 소문자로 변환한 이름을 사용자에게 묻지도 않고 쿠키로 설정하는 것인데, 자칫하면 기존에 사용하던 다른 값을 덮어 쓸 위험이 있다.
// 질의형 함수는 절대로 데이터를 덮어쓰는 작업을 수행해선 안 된다.
// 참고 https://webactually.com/2018/02/06/%EB%AA%85%ED%99%95%ED%95%9C-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%EB%B2%95/
*/

// 다시 리팩터링으로 돌아와 부수효과가 있는 코드를 슬라이드하거나 부수효과가 있는 코드를 건너뛰어야 한다면 훨씬 신중해야 한다.
// 11번 줄을 코드 끝으로 슬라이드 하고 싶다고 가정한다
// 12번 줄 떄문에 가로막히는데, 11번 줄에서 상태를 수정한 변수 discount를 12번 줄에서 참조하기 떄문이다.
// 13번 줄도 12번 줄앞으로 이동할 수 없다.
// 13번 줄이 참조하는 변수 charge를 12번 줄에서 수정하기 떄문이다.

// Step 3 8번줄 이동
// 참조하는 줄이 없기 떄문에 12번 앞까지 이동할 수 있다.
const pricingPlan = retrievePricingPlan(); // 1                                     
const baseCharge = pricingPlan.base;  // 3                                          
let charge; // 4                                                                    
const chargePerUnit = pricingPlan.unit; // 5                                       
const order = retreiveOrder(); // 2 order 선언부 이동                                                 
const units = order.units; // 6                                                      
charge = baseCharge + units * chargePerUnit; // 8                                   
let discountableUnits = Math.max(units - pricingPlan.discountThreshold, 0); // 9    
let discont;  // 7  discount 이동                                                                  
discont = discountableUnits * pricingPlan.discountFactor; // 10                       
if (order.isRepeat) discount += 20; // 11                                             
charge = baseCharge + units * chargePerUnit; // 8                                    
charge = charge - discount; // 12                                                    
chargeOrder(charge); // 13                                                           
// 슬라이드할 코드 조각과 건너뛸 코드 중 어느 한쪽이 다른 쪽에서 참조하는 데이터를 수정한다면 슬라이드를 할 수 없다.

// 하지만 다음의 경우에는 예외이다.
a = a + 10;
a = a + 5;
// 슬라이드가 안전한 지를 판단하려면 관련된 연산이 무엇이고 어떻게 구성되는지를 완벽히 이해해야 한다.
// 상태 갱신에 특히나 신경 써야 하기 때문에 상태를 갱신하는 코드 자체를 최대한 제거하는 게 좋다.
// 저자라면 슬라이드를 하기 전에 charge 변수를 쪼갤것이다.
// 더 복잡한 구조에서는 간섭 여부를 확인하기 어렵기 떄문에 테스트를 활용하여 데이터가 깨지는 일이 없도로 살펴야 한다.
// 슬라이드 후 테스트가 실패했을 때 가장 좋은 대처 방법은 더 작게 슬라이드해보는 것이다.

// 예시 2 조건문이 있을 때의 슬라이드
// 조건문 밖으로 슬라이드할 때는 중복 로직이 제거될 것이고, 조건문 안으로 슬라이드할 때는 반대로 중복 로직이 추가될 것이다.
let result;
if (availableResources.length == 0) {
    result = createResource();
    allocatedResources.push(result); // 중복
} else {
    result = availableResources.pop();
    allocatedResources.push(result); // 중복
}
return result;

// Step 1 중복 된 문장들을 밖으로 슬라이드 
let result;
if (availableResources.length == 0) {
    result = createResource();
} else {
    result = availableResources.pop();
}
allocatedResources.push(result); // 슬라이드 
return result;
```

### 더 읽을거리

문장 교환하기(Swap Statement)라는 이름의 거의 똑같은 리팩터링도 있다.

문장 교환하기는 인접한 코드 조각을 이동하지만, 문장 하나짜리 조각만 취급한다.

따라서, 이동할 조각과 건너뛸 조각 모두 단일 문장으로 구성된 문장 슬라이드로 생각해도 된다.

## 8.7 반복문 쪼개기

### 배경

한 반복문에서 여러 일을 처리하면, 반복문을 수정해야 할 때마다 반복문에서 일어나는 여러 일을 잘 이해하고 진행해야 한다.

반대로 각각의 반복문으로 분리해두면 수정할 동작 하나만 이해하면 된다.

반복문을 분리하면 사용하기 쉽고, 한 가지 값만 계산하는 반복문이라면 그 값만 곧바로 반환할 수 있다.

반면 여러 일을 수행하는 반복문이라면 구조체를 반환하거나 지역 변수를 활용해야 한다.

참고로 반복문 쪼개기는 서로 다른 일들이 한 함수에서 이뤄지고 있다는 신호일 수 있고, 그래서 반복문 쪼개기와 함수 추출하기는 연이어 수행하는 일이 잦다.

최적화는 코드를 깔끔히 정리한 이후에 수행하자.

### 절차

1. 반복문을 복제해 두 개로 만든다.
2. 반복문이 중복되어 생기는 부수효과를 파악해서 제거한다.
3. 테스트한다.
4. 완료됐으면, 각 반복문을 함수로 추출할지 고민한다.

### 예시

```jsx
// ASIS
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
    averageAge += p.age;
    totalSalary += p.salary;
}

averageAge = averageAge / people.length;

// TOBE
let totalSalary = 0;
for (const p of people) {
    totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
    averageAge += p.age;
}

averageAge = averageAge / people.length;

// 예시
// 전체 급여와 가장 어린 나이를 계산하는 코드
// ASIS
let youngest = people[0] ? people[0].age : Infinity;
let totalSalary = 0;
for (const p of people) {
    if (p.age < youngest) youngest = p.age;
    totalSalary += p.salary;
}

return `최연수 : ${youngest}, 총 급여: ${totalSalary}`;

// TOBE
// Step 1 반복문 복제
let youngest = people[0] ? people[0].age : Infinity;
let totalSalary = 0;
for (const p of people) {
    if (p.age < youngest) youngest = p.age;
    totalSalary += p.salary;
}

for (const p of people) {
    if (p.age < youngest) youngest = p.age;
    totalSalary += p.salary;
}

return `최연수 : ${youngest}, 총 급여: ${totalSalary}`;
// Step 부수효과 제거

let youngest = people[0] ? people[0].age : Infinity;
let totalSalary = 0;
for (const p of people) {
    totalSalary += p.salary; // 한 가
}

for (const p of people) {
    if (p.age < youngest) youngest = p.age; // 한 가지 일만 
}

return `최연소 : ${youngest}, 총 급여: ${totalSalary}`;
```

### 더 가다듬기

각각 나눔 반복문을 함수로 추출하면 어떨지 한 묶으로 고민하자.

```jsx
// 더 가다듬기
// Step 1 문장 슬라이드하기
let totalSalary = 0;
for (const p of people) {
    totalSalary += p.salary;
}

let youngest = people[0] ? people[0].age : Infinity; // 이동
for (const p of people) {
    if (p.age < youngest) youngest = p.age; 
}

return `최연소 : ${youngest}, 총 급여: ${totalSalary}`;

// Step 2 각 반복문을 함수로 추출
return `최연소 : ${youngestAge()}, 총 급여: ${totalSalary()}`;

function totalSalary() {
    let totalSalary = 0;
    for (const p of people) {
        totalSalary += p.salary;
    }
    return totalSalary;
}

function yongestAge() {
    let youngest = people[0] ? people[0].age : Infinity; // 이동
    for (const p of people) {
        if (p.age < youngest) youngest = p.age; 
    }
    return youngest;
}

// Step 3 파이프라인 바꾸기와 알고리즘 교체하기 적용
return `최연소 : ${youngestAge()}, 총 급여: ${totalSalary()}`;

function totalSalary() {
    return people.reduce((total, p) => total + p.salary, 0);
}

function yongestAge() {
    return Math.min(...people.map(p => p.age));
}
```

## 8.8 반복문을 파이프라인으로 바꾸기

### 배경

컬렉션 파이프라인(Collection Pipeline)을 이용하면 컬렉션 순회를 일련의 연산으로 표현할 수 있다.

이때 각 연산은 컬렉션을 입력받아 다른 컬렉션을 내뱉는다.

대표적인 연산은 map과 filter다.

map은 함수를 사용해 입력 컬렉션의 각 원소를 변환하고, filter는 또 다른 함수를 사용해 입력 컬렉션을 필터링해 부분집합을 만든다.

논리의 파이프라인을 표현하면 이해하기 쉬워진다.

### 절차

1. 반복문에서 사용하는 컬렉션을 가르키는 변수를 하나 만든다.
  1. 기존 변수를 단순히 복사한 것일 수도 있다.
2. 반복문의 첫 줄부터 시작해서, 각각의 단위 행위를 적절한 컬렉션 파이프라인 연산으로 대체한다. 이 때 컬렉션 파이프라인 연산은 1에서 만든 반복문 컬렉션 변수에서 시작하여, 이전 연산의 결과를 기초로 연쇄적으로 수행된다. 하나를 대체할 때마다 테스트한다.
3. 반복문의 모든 동작을 대체했다면 반복문 자체를 지운다.
  1. 반복문이 결과를 누적 변수(accumulator)에 대입했다면 파이프라인의 결과를 그 누적 변수에 대입한다.

### 예시

```jsx
// ASIS
const names = [];
for (const i of input) {
    if (i.job === "programmer")
        names.push(i.names);
}

// TOBE
const names = input
    .filter(i => i.job === "programmer")
    .map(i => i.name);

// 예시
// ASIS
function acquireData(input) {
    const lines = input.split("\n"); // 컬렉션
    let firstLine = true;
    const result = [];
    for (const line of lines) { // 반복문
        if (firstLine) {
            firstLine = false;
            continue;
        }
        if (line.trim() === "") continue;
        const record = line.split(",");
        if (record[1].trim() === "India") {
            result.push({city: record[0].trim(), phone: record[2].trim()});
        }
    }
    return result;
}

// TOBE
// Step 1 컬렉션을 가르키는 별로 변수를 새로 만든다.
function acquireData(input) {
    const lines = input.split("\n"); 
    let firstLine = true;
    const result = [];
    const loopItems = lines; // 새로 만든 변수
    for (const line of loopItems) { // 새로 만든 변수로 변경
        if (firstLine) {
            firstLine = false;
            continue;
        }
        if (line.trim() === "") continue;
        const record = line.split(",");
        if (record[1].trim() === "India") {
            result.push({city: record[0].trim(), phone: record[2].trim()});
        }
    }
    return result;
}

// Step 2 slice() 연산을 루프 변수에서 수행하고 반복문 안의 if문을 제거하자
// 이 코드의 반복문에서 첫 if문은 CSV 데이터의 첫 줄을 건너뛰는 역할이다.
// firstLine 변수도 보너스로 함께 제거
function acquireData(input) {
    const lines = input.split("\n"); 
    const result = [];
    const loopItems = lines
        .slice(1); // 슬라이스로 대체
    for (const line of loopItems) { 
            // 제거 
        if (line.trim() === "") continue;
        const record = line.split(",");
        if (record[1].trim() === "India") {
            result.push({city: record[0].trim(), phone: record[2].trim()});
        }
    }
    return result;
}

// Step 3 filter() 연산으로 trim 대체
function acquireData(input) {
    const lines = input.split("\n"); 
    const result = [];
    const loopItems = lines
        .slice(1)
        .filter(line => line.trim() != "") // trim을 filter로 대체
        ;  // 파이프라인을 사용할 때 문장 종료 세미클론(;)을 별도 줄에 적어주면 편하다.
    for (const line of loopItems) { 
        const record = line.split(",");
        if (record[1].trim() === "India") {
            result.push({city: record[0].trim(), phone: record[2].trim()});
        }
    }
    return result;
}

// Step 4 map 연산을 이용하여 문자열을 배열로 변환
function acquireData(input) {
    const lines = input.split("\n"); 
    const result = [];
    const loopItems = lines
        .slice(1)
        .filter(line => line.trim() != "")
        .map(line => line.split(",")) // map으로 배열로 변환
        ; 
    for (const line of loopItems) { 
        const record = line;
        if (record[1].trim() === "India") {
            result.push({city: record[0].trim(), phone: record[2].trim()});
        }
    }
    return result;
}

// Step 5 다시 filter 연산자로 인도에 위치한 사무실 레코드를 뽑아낸다
function acquireData(input) {
    const lines = input.split("\n"); 
    const result = [];
    const loopItems = lines
        .slice(1)
        .filter(line => line.trim() != "")
        .map(line => line.split(","))
        .filter(record => record[1].trim() === "India")
        ; 
    for (const line of loopItems) { 
        const record = line;
        result.push({city: record[0].trim(), phone: record[2].trim()});
        
    }
    return result;
}

// Step 6 map을 사용하여 결과 레코드를 생성
function acquireData(input) {
    const lines = input.split("\n"); 
    const result = [];
    const loopItems = lines
        .slice(1)
        .filter(line => line.trim() != "")
        .map(line => line.split(","))
        .filter(record => record[1].trim() === "India")
        .map(record => ({city: record[0].trim(), phone: record[2].trim()}))
        ; 
    for (const line of loopItems) { 
        const record = line;
        result.push({city: record[0].trim(), phone: record[2].trim()});
        
    }
    return result;
}

// Step 7 결과를 누적 변수에 추가
function acquireData(input) {
    const lines = input.split("\n"); 
    const result = [];
    const result = lines // 결과를 result 변수에 추가
        .slice(1)
        .filter(line => line.trim() != "")
        .map(line => line.split(","))
        .filter(record => record[1].trim() === "India")
        .map(record => ({city: record[0].trim(), phone: record[2].trim()}))
        ; 
    return result;
}

// 더 가다듬기 
// result 변수를 인라인하고, 람다(lamda) 변수 중 일부의 이름을 바꾸고, 코드를 읽기 쉽도록 레이아웃을 표 형태로 정돈한다.
function acquireData(input) {
    const lines = input.split("\n"); 
    return lines 
        .slice(1)
        .filter (line => line.trim() != "")
        .map    (line => line.split(","))
        .filter (fields => fields[1].trim() === "India")
        .map    (fileds => ({city: fileds[0].trim(), phone: fileds[2].trim()}))
        ; 
}
```

### 더 가다듬기

result 변수를 인사인하고, 람드(lamda) 변수 중 일부의 이름을 바꾸고, 코드를 읽기 쉽도록 레이아웃을 표 형태로 정돈한다.

## 8.9 죽은 코드 제거하기

### 배경

사용되지 않는 코드가 있다면 소프트웨어의 동작을 이해하는 데는 커다란 걸림돌이 될 수 있다.

코드가 더 이상 사용되지 않게 됐다면 지워야 한다.

버전 관리 시스템이 있기 때문에 소스를 주석으로 처리하지 않아도 된다.

### 절차

1. 죽은 코드를 외부로터에서 참조할 수 있는 경우라면(예컨데 함수 하나가 통째로 죽었을 때) 혹시라도 호출하는 곳이 있는지 확인한다.
2. 없다면 죽은 코드를 제거한다.
3. 테스트한다.

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}