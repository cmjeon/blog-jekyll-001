---
layout: single
title: 리팩터링2 스터디 - 012 - 1
categories: 
  - bookstudy - refactoring2
tags: 
  - refactoring2
---

# 12장 상속 다루기

## 12.1 메소드 올리기

### 배경

중복 코드 제거는 중요하다. 중복된 메서드가 지금 당장은 문제없이 동작하더라도, 미래에 벌레가 꼬이는 음식물 쓰레기로 전락할 수 있다.

무언가 중복 된다는 것은 수정 사항이 한쪽에는 반영 되지 않을 수 있다는 위험을 항상 수반한다.

메서드 올리기를 적용하기 가장 쉬운 상황은 메서드들의 본문 코드가 똑같을 때이다.

### 절차

1. 똑같이 동작하는 메서드인지 면밀히 살펴본다.
  1. 실질적으로 하는 일은 같지만 코드가 다르다면 본문 코드가 똑같아질 때까지 리팩터링한다.
2. 메서드 안에서 호출하는 다른 메서드와 참조하는 필드들을 슈퍼클래스에서도 호출하고 참조할 수 있는지 확인한다.
3. 메서드 시그니처가 다르다면 함수 선언 바꾸기로 슈퍼클래스에서 사용하고 싶은 형태로 통일한다.
4. 슈퍼클래스에 새로운 메서드를 생성하고, 대상 메서드의 코드를 복사해넣는다.
5. 정적 검사를 수행한다.
6. 서브클래스 중 하나의 메서드를 제거한다.
7. 테스트한다.
8. 모든 서브클래스의 메서드가 없어질 때까지 다른 서브클래스의 메서드를 하나씩 제거한다.

### 예시

```jsx
// ASIS
class Employee {
    //...
}

class Salesperson extends Employee {
    get name() {
        //...
    }
}

class Engineer extends Employee {
    get name() {
        //...
    }
}

// TOBE
class Employee {
    get name() {
        //...
    }
}

class Salesperson extends Employee {

}

class Engineer extends Employee {

}

// 예시
// ASIS

class Employee extends Party {
    get annualCast() {
        return this.monthlyCost * 12
    }
}

class Department extends Party {
    get totalAnnualCost() {
        return this.monthlyCost * 12
    }
}

// Step 1 두 메서드의 이름을 통일 한다.
class Employee extends Party {
    get annualCast() {
        return this.monthlyCost * 12
    }
}

class Department extends Party {
    get annualCast() {
        return this.monthlyCost * 12
    }
}

// Step 2 부모 클래스 Party 클래스에 서브클래스 중 하나의 메서드를 복사하여 붙여넣기 한다.
class Party {
    get annualCast() {
        return this.monthlyCost * 12
    }
}

// Step 3 자식 클래스의 메소드를 삭제한다.
class Party {
    get annualCast() {
        return this.monthlyCost * 12
    }
}

class Employee extends Party {

}

class Department extends Party {

}

// Step 4 annualCost가 호출하는 monthlyCost()를 서브 클래스가 monthlyCost()를 구현해야 한다는 사실을 함정 메서드로 알려준다.
class Party {
    get annualCast() {
        return this.monthlyCost * 12
    }

    get monthlyCost() {
        throw new SubclassResponsibilityError() // 이런 오류를 서브클래스 책임 오류(subclass responsibilty error라 한다(스몰토크에서 유래)
    }
}

class Employee extends Party {

}

class Department extends Party {

}
```

## 12.2 필드 올리기

### 배경

독립적으로 개발된 서브클래스들에 일부 기능이 중복되어 있을 때가 있다. 필드들이 어떻게 이용되는지 확인 후 비슷한 방식으로 사용되고 있다면 슈퍼클래스로 끌어 올리자.

이 행위의 장점은 데이터 중복 선언을 없앨 수 있고, 해당 필드를 사용하는 동작을 슈퍼클래스로 옮길 수 있다.

동적 언어 중에는 필드를 클래스 정의에 포함시키지 않는 경우가 많다. 대신 필드에 가장 처음 값이 대입될 때 처음 등장한다. 이런 경우라면 필드를 올리기 전에 반드시 생성자 본문부터 올려야한다.

### 절차

1. 후보 필드들을 사용하는 곳 모두가 그 필드들을 똑같은 방식으로 사용하는지 면밀히 살펴본다.
2. 필드들의 이름이 각기 다르다면 똑같은 이름으로 바꾼다(필드 이름 바꾸기)
3. 슈퍼클래스에 새로운 필드를 생성한다.
  1. 서브클래스에서 이 필드에 접근할 수 있어야 한다(대부분 언어에서는 protected로 선언하면 된다)
4. 서브클래스의 필드들을 제거한다.
5. 테스트한다.

### 예시

```java
// ASIS
class Employee {

}

class Salesperson extends Employee {
    private String name;
}

class Engineer extends Employee {
    private String name;
}
// TOBE
class Employee {
    protected String name;
}

class Salesperson extends Employee {
}

class Engineer extends Employee {
    
}
```

## 12.3 생성자 본문 올리기

### 배경

생성자는 다루기 까다롭다. 일반 메서드와 많이 달라서 생성자가 하는 일에 제약을 두는 편이 좋다.

생성자는 할 수 있는 일과 호출 순서에 제약이 있기 때문에 조금 다른 식으로 접근해야 한다.

### 절차

1. 슈퍼클래스에 생성자가 없다면 하나 정의한다. 서브클래스의 생성자들에서 이 생성자가 호출되는지 확인한다.
2. 문장 슬라이드하기로 공통 문장 모두를 super() 호출 직후로 옮긴다.
3. 공통 코드를 슈퍼클래스에 추가하고 서브클래스에서는 제거한다. 생성자 매개변는 중 공통 코드에서 참조하는 값들을 모두 super()로 거넨다.
4. 테스트한다.
5. 생성자 시작 부분으로 옮길 수 없는 공통 코드에는 함수 추출하기와 메서드 올리기를 차례로 적용한다.

### 예시

```java
// ASIS
class Party {

}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super();
        this._id = id;
        this._name = name;
        this._monthlyCost = monthlyCost;
    }
}

// TOBE

class Party {
    constructor(name) {
        this._name = name;
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super(name);
        this._id = id;
        this._monthlyCost = monthlyCost;
    }
}

// 예시
// ASIS

class Party {}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super();
        this._id = id;
        this._name = name;
        this._monthlyCost = monthlyCost;
    }
}

class Department extends Party {
    constructor(name, staff) {
        super();
        this._name = name;
        this._staff = staff;
    }
}

// 예시 1
// Step 1 공통 코드 name을 Employee에서 super 아래로 옮긴다.
class Party {}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super();
        this._name = name;
        this._id = id;
        this._monthlyCost = monthlyCost;
    }
}

class Department extends Party {
    constructor(name, staff) {
        super();
        this._name = name;
        this._staff = staff;
    }
}

// Step 2 테스트 후 이 공통 코드를 슈퍼 클래스로 옮긴다.
class Party {
    constructor(name) {
        this._name = name;
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super(name);
        this._id = id;
        this._monthlyCost = monthlyCost;
    }
}

class Department extends Party {
    constructor(name, staff) {
        super(name);
        this._staff = staff;
    }
}

// 예시 2 공통 코드가 나중에 올 때
// ASIS
class Employee {
    constructor(name) {
        this._name = name
    }
    get isPrivileged() {

    }

    assignCar() {

    }
}

class Manager extends Employee {
    constructor(name, grade) {
        super(name);
        this._grade = grade;
        if (this.isPrivileged) this.assignCar(); // 모든 서브클래스가 수행한다.
    }

    get isPrivileged() {
        return this._grade > 4;
    }
}

// Step 1 공통 코드를 함수로 추출
class Manager extends Employee {
    constructor(name, grade) {
        super(name);
        this._grade = grade;
        this.finishConstruction();
    }

    finishConstruction() {
        if (this.isPrivileged) this.assignCar()
    }
}
// Step 2 추출한 메서드를 슈퍼 클래스로 옮긴다.
class Employee {
    constructor(name) {
        this._name = name
    }

    get isPrivileged() {
    }
    
    assignCar() {
    }

    finishConstruction() {
        if (this.isPrivileged) this.assignCar()
    }
}
```

## 12.4 메서드 내리기

### 배경

특정 서브클래스  하나에만 관련있는 메서드는 슈퍼 클래스에서 제거하고 해당 서브 클래스에 추가하는 편이 깔끔하다.

다만, 이 리팩터링은 해당 기능을 제공하는 서브클래스가 정확히 무엇인지를 호출자가 알고 있을 때만 적용할 수 있다.

### 절차

1. 대상 메서드를 모든 서브클래스에 복사한다.
2. 슈퍼클래스에서 그 메서드를 제거한다
3. 테스트한다
4. 이 메서드를 사용하지 않는 모든 서브클래스에서 삭제한다.
5. 테스트한다.

### 예시

```java
// ASIS
class Employee {
    get quota {
    }
}

class Engineer extends Employee {

}

class Salesperson extends Employee {

}

// TOBE
class Employee {

}

class Engineer extends Employee {

}

class Salesperson extends Employee {
    get quota {
    }
}
```

## 12.6 타입 코드를 서브클래스로 바꾸기

### 배경

소프트웨어 시스템에서는 비슷한 대상들을 특정 특성에 따라 구분해야할 때가 자주 있다.

이런 일을 다루는 수단으로는 타입 코드(type code)필드가 있다.

타입  코드는 프로그래밍 언어에 따라 열거형이나 심볼, 문자열, 숫자 등으로 표현하며, 외부 서비스가 제공하는 데이터를 다루려 할 때 딸려오는 일이 흔하다.

그 이상 무엇언가 필요할 때 사용하는 것이 서브클래스이다.

서브클래스는 두 가지  면에서 매력적이다.

첫째, 조건에 따라 다르게 동작하도록 해주는 다형성을 제공한다. 타입 코드에 따라 동작이 달라져야 하는 함수가 여러 개일 때 특히 유용하다.

서브클래스를 이용하면 이런 함수들에 조건부 로직을 다형성으로 바꾸기를 적용할 수 있다.

두 번째, 특정 타입에서만 의미가 있는 값을 사용하는 필드나 메서드가 있을 때 발현된다. 필드 내리기를 통해 서브클래스만 필드를 갖도록 정리하자.

### 절차

1. 타입 코드 필드를 자가 캡슐화한다
2. 타입 코드 값 하나를 선택하여 그 값에 해당하는 서브클래스를 만든다. 타입 코드 게터 메서드를 오버라이드하여 해당 타입 코드의 리터럴 값을 반환하게 한다.
3. 매개변수로 받은 타입 코드와 방금 만든 서브클래스를 매핑하는 선택 로직을 만든다.
  1. 직접 상속일 때는 생성자를 팩터리 함수로 바꾸기를 적용하고 선택 로직을 팩터리에 넣는다. 간접 상속일 떄는 선택 로직을 생성자에 두면 될 것이다.
4. 테스트한다.
5. 타입 코드 값 각각에 대해 서브클래스 생성과 선택 로직 추가를 반복한다. 클래스 하나가 완성될 때마다 테스트한다.
6. 타입 코드 필드를 제거한다.
7. 테스트한다.
8. 타입 코드 접근자를 이용하는 메서드 모두에 메서드 내려기와 조건부 로직을 다형성으로 바꾸기를 적용한다.

### 예시(직접상속)

```java
// ASIS
function createEmployee(name, type) {
    return new Employee(name, type);
}

// TOBE
function createEmployee(name, type) {
    switch (type) {
        case "engineer": return new Engineer(name);
        case "salesperson": return new Salesperson(name);
        case "maanger": return new Manager(name);
    }
}

// 예시
// ASIS
class Employee {
    constructor(name, type) {
        this.validateType(type);
        this._name = name;
        this._type = type;
    }

    validateType(arg) {
        if (!["engineer", "manager", "salesperson"].includes(arg))
            throw new Error(`${arg}라는 직원 유형은 없습니다.'`);
    }
    toString() {
        return `${this._name} (${this._type})`
    }
}

// Step 1 코드 타입을 자가 캡슐화 한다.

class Employee {
    get type() { return this._type; }
    
    toString() {
        return  `${this._name} (${this.type})` // 게터 사용하여 type 가져옴
    }
}

// Step 2 타입 코드 중 하나 엔지니어를 서브클래싱한다 코드 게터를 오버라이드하여 적절한 리터럴 값을 반환하기만 하면 된다.
class Engineer extends Employee {
    get type() { return "engineer"; }
}

// Step 3 생성자를 팩터리 함수로 바꿔서 선택로직을 담을 별도 장소를 마련한다.
function createEmployee(name, type) {
    return new Employee(name, type);
}

// Step 4 새로 만들 서브클래스를 사용하기 위해 선택 로직을 팩터리에 추가한다.
function createEmployee(name, type) {
    switch (type) {
        case "engineer": return new Engineer(name, type)
    }
    return new Employee(name, type);
}

// Step 5 다른 타입도 서브클래스로 만든다.
class Salesperson extends Employee {
    get type() { return "slaesperson";}
}

class Manager extends Employee {
    get type() { return "manager"; }
}

function createEmployee(name, type) {
    switch (type) {
        case "engineer": return new Engineer(name, type);
        case "salesperson": return new Salesperson(name, type);
        case "manager": return new Manager(name, type);
    }
    return new Employee(name, type);
}

// Step 6 타입 코드 필드와 슈퍼클래스의 게터를 제거한다.
class Employee {
    constructor(name, type) {
        this.validateType(type);
        this._name = name;
    }
    
    toString() {
        return  `${this._name} (${this.type})`
    }
}

// Step 7 검증 로직도 제거한다. switch 문이 검증을 수행하기 떄문이다.
class Employee {
    constructor(name, type) {
        this._name = name;
    }
    
    toString() {
        return  `${this._name} (${this.type})`
    }
}

function createEmployee(name, type) {
    switch (type) {
        case "engineer": return new Engineer(name, type);
        case "salesperson": return new Salesperson(name, type);
        case "manager": return new Manager(name, type);
        default: throw new Error(`${type}라는 직원 유형은 없습니다.`)
    }
}

// Step 8 생성자에서 건네는 타입 코드 인수는 쓰이지 않으니 제거한다
class Employee {
    constructor(name) {
        this._name = name;
    }
    
    toString() {
        return  `${this._name} (${this.type})`
    }
}

function createEmployee(name, type) {
    switch (type) {
        case "engineer": return new Engineer(name);
        case "salesperson": return new Salesperson(name);
        case "manager": return new Manager(name);
        default: throw new Error(`${type}라는 직원 유형은 없습니다.`)
    }
}

// 서브클래스들에 타입 코드 게터(get type()) 여전히 남아 있다. 보통은 제거하고 싶겠지만, 이용하는 코드가 어딘가에 남아 있을 수 있다.
// 조건부 로직을 다형성으로 바꾸기와 메서드 내리기로 문제를 해결하자. 하나씩 해결하다 보면 타입 게터를 호출하는 코드가 모두 사라질 것이다.
```

### 예시(간접상속)

```java
// ASIS
class Employee {
    constructor(name, type) {
        this.validateType(type);
        this._name = name;
        this._type = type;
    }

    validateType(arg) {
        if (!["engineer", "manager", "salesperson"].includes(arg))
            throw new Error(`${arg}라는 직원 유형은 없습니다.`)
    }
    get type() { return this._type; }
    set type(arg) {this._type = arg;}

    get capitalizedType() {
        return this._type.charAt(0).toUpperCase() + this._type.substr(1).toLowerCase();
    }

    toString() {
        return `${this._name} (${this.capitalizedType})`;
    }
}

// TOBE
// Step 1 타입 코드를 객체로 바꾸기
class EmployeeType {
    constructor(aString) {
        this._value = aString;
    }

    toString() {
        return this._value;
    }
}

class Employee {
    constructor(name, type) {
        this.validateType(type);
        this._name = name;
        this.type = type;
    }

    validateType(arg) {
        if (!["engineer", "manager", "salesperson"].includes(arg))
            throw new Error(`${arg}라는 직원 유형은 없습니다.`)
    }
    get typeString() { return this._type.toString();}
    get type() { return this._type;}
    set type(arg) { this._type = new EmployeeType(arg);}

    get capitalizedType() {
        return this.typeString.charAt(0).toUpperCase() + this.typeString.substr(1).toLowerCase();
    }

    toString() {
        return `${this._name} (${this.capitalizedType})`
    }
}

// Step 2 직원 유형 리팩터링
class Employee {
    set type(arg) { this._type = Employee.createEmployeeType(arg);}
    static createEmployeeType(aString) {
        switch(aString) {
            case "engineer": return new Engineer();
            case "manager": return new Manager();
            case "salesperson": return new Salesperson();
            default: throw new Error(`${aString}라는 직원 유형은 없습니다.`)
        }
    }
}
class EmployeeType {

}

class Engineer extends EmployeeType {
    toString() { return "engineer";}
}

class Manager extends EmployeeType {
    toString() { return "manager";}
}

class Salesperson extends EmployeeType {
    toString() { return "salesperson";}
}
// 빈 EmployeeType을 제거할 수도 있지만 다양한 서브클래스 사이의 관계를 명확히 알려주는 클래스라면 그냥 두어도 괜찮다.
// 또한, 이 클래스는 다른 기능을 옮겨놓기에 편리한 장소이기도 하다.
// Step 3 capitalizedName 메소드를 옮겨보자
class Employee {
    toString() {
        return `${this._name} (${this.type.capitalizedName})`
    }
}

class EmployeeType {
    get capitalizedName() {
        return this.toString().charAt(0).toUpperCase() + this.toString().substr(1).toLowerCase();
    }
}
```

## 12.7 서브클래스 제거하기

### 배경

소프트웨어가 성장함에 따라 서브클래스로 만든 변종이 다른 모듈로 이동하거나 완전히 사라지기도 하면서 가치가 사라진다.

사용하지 않는 코드가 되면 서브클래스를 슈퍼클래스의 필드로 대체해 제거하는게 최선이다.

### 절차

1. 서브클래스의 생성자를 팩터리 함수로 바꾼다.
  1. 생성자를 사용하는 측에서 데이터 필드를 이용해 어떤 서브클래스를 생성할지 결정한다면 그 결정 로직을 슈퍼클래스의 팩터리 메서드에 넣는다.
2. 서브클래스의 타입을 검사하는 코드가 있다면 그 검사 코드에 함수 추출하기와 함수 옮기기를 차례로 적용하여 슈퍼클래스로 옮긴다. 하나 변경할 때마다 테스트한다.
3. 서브클래스의 타입을 나타내는 필드를 슈퍼클래스에 만든다.
4. 서브클래스를 참조하는 메서드가 방금 만든 타입 필드를 이용하도록 수정한다.
5. 서브클래스를 지운다.
6. 테스트한다.

### 예시

```java
// ASIS
class Person {
    get genderCode() { return "X"; }
}

class Male extends Person {
    get genderCode() { return "M"; }
}

class Female extends Person {
    get genderCode() { return "F"; }
}

// TOBE

class Person {
    get genderCode() {
        return this._genderCode;
    }
}

// 예시
class Person {
    constructor(name) {
        this._name = name;
    }

    get name() { return this._name; }
    get genderCode() { return "X"; }
}

class Male extends Person {
    get genderCode() { return "M"; }
}

class Female extends Person {
    get genderCode() { return "F"; }
}

// 서브클래스가 하는 일이 이게 다라면 존재할 이유가 없다

// 클라이언트
const numberOfMales = people.filter(p => p instanceof Male).length;

// Step 1 생성자를 팩터리 함수로 바꾸기
function createPerson(name) {
    return new Person(name);
}
function createMale(name) {
    return new Male(name);
}
function createFemale(name) {
    return new Female(name);
}

// Step 2 성별 코드를 사용하는 곳에서 직접 생성
function loadFromInput(data) {
    const result = [];
    data.forEach(aRecord => {
        let p;
        switch(aRecord.gender) {
            case 'M': p = new Male(aRecord.name); break;
            case 'F': p = new Female(aRecord.name); break;
            default: p = new Person(aRecord.name);
        }
        result.push(p);
    });
    return result
}

// Step 3 생성할 클래스를 선택하는 로직을 함수로 추출하고 그 함수를 팩터리 함수로 만들기
function creatPerson(aRecord) {
    let p;
    switch(aRecord.gender) {
        case 'M': p = new Male(aRecord.name); break;
        case 'F': p = new Female(aRecord.name); break;
        default: p = new Person(aRecord.name);
    }
    return p;
}
function loadFromInput(data) {
    const result = [];
    data.forEach(aRecord => {
        result.push(creatPerson(aRecord));
    });
    return result
}

// Step 4 두 함수를 정리 createPerson()에서 변수 p를 인라인
function creatPerson(aRecord) {
    switch(aRecord.gender) {
        case 'M': return new Male(aRecord.name); 
        case 'F': return new Female(aRecord.name); 
        default: return new Person(aRecord.name);
    }
}

// Step 5 loadFromInput() 의 반복문을 파이프라인으로 변경
function loadFromInput(data) {
    return data.map(aRecord => creatPerson(aRecord));
}

// Step 6 클라이언트의 타입 검사 코드(instanceof)를 함수로 추출
// 클라이언트
const numberOfMales = people.filter(p => isMale(p)).length;

function isMale(aPerson) {
    return aPerson instanceof Male;
}

// Step 7 추출한 함수를 Person 클래스로 옮기기
class Person {
    get isMale() { return this instanceof Male; }
}

const numberOfMales = people.filter(p => p.isMale).length;

// Step 8 서브클래스들의 차이를 나타낼 필드를 추가, 성별 정보는 Person 클래스 외부에서 정해 전달하는 방식이니 생성자에서 매개변수로 받아 설정
class Person {
    constructor(name, genderCode) {
        this._name = name;
        this._genderCode = genderCode || "X";
    }

    get genderCode() { return this._genderCode; }
}

// Step 9 남성인 경우의 로직을 슈퍼클래스로 옮긴다.
function createPerson(aRecord) {
    switch(aRecord.gender) {
        case 'M': return new Person(aRecord.name, "M");
        case 'F': return new Female(aRecord.name);
        default: return new Person(aRecord.name);
    }
}

class Person {
    constructor(name, genderCode) {
        this._name = name;
        this._genderCode = genderCode || "X";
    }

    get genderCode() { return this._genderCode; }
    get isMale() { return "M" === this._genderCode; }
}

// Step 10 테스트 후 남성 클래스 제거 후 여성 클래스도 같은 작업 반복
function createPerson(aRecord) {
    switch(aRecord.gender) {
        case 'M': return new Person(aRecord.name, "M");
        case 'F': return new Person(aRecord.name, "F");
        default: return new Person(aRecord.name);
    }
}

// Step 11 코드 일관성을 위해 디폴트에도 성별 코드 건네도록 설정
function createPerson(aRecord) {
    switch(aRecord.gender) {
        case 'M': return new Person(aRecord.name, "M");
        case 'F': return new Person(aRecord.name, "F");
        default: return new Person(aRecord.name, "X");
    }
}

class Person {
    constructor(name, genderCode) {
        this._name = name;
        this._genderCode = genderCode
    }
}
```

# 참고
- [리팩터링 2판](http://www.yes24.com/Product/Goods/89649360){:target="_blank"}