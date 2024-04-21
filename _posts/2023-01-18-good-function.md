---
layout: single
title: "좋은 함수 만들기"
categories:
  - "javascript"
tags:
  - "클린코드"
---

## 좋은 함수 만들기 - 부작용과 거리두기

[1. 좋은 함수 만들기 - 부작용과 거리두기](https://jojoldu.tistory.com/697)

좋은 함수란 순수 함수

:::info

순수함수는 동일한 입력일 경우 항상 동일한 출력을 반환하며, 부작용이 없는 함수

:::

리팩토링 될 함수

```javascript
export async function sendCompanyFees(companySellings: CompanySelling[]) {
  for (const companySelling of companySellings) {
    const fee = companySelling.sellingAmount * companySelling.commission;

    if (fee >= 100) {
      await axiosSendFee(companySelling.bankCode, fee);
    }
  }

  Modal.open(`${companySellings.length} 개 기업들에게 송금되었습니다.`);
}
```

### 나쁜 리팩토링 사례

```javascript
// 전체 기능을 관리한다
export async function sendCompanyFees(companySellings: CompanySelling[]) {
  await sendFees(companySellings);
  Modal.open(`${companySellings.length} 개 기업들에게 송금되었습니다.`);
}

// companySellings 만큼 sendFee를 호출한다
export async function sendFees(companySellings: CompanySelling[]) {
  for (const companySelling of companySellings) {
    await sendFee(companySelling);
  }
}

// 100원이상인 경우 axiosSendFee를 호출한다
export async function sendFee(companySelling: CompanySelling) {
  const fee = getFee(companySelling);

  if (fee >= 100) {
    // 100원 이상이면 계산된 금액 송금하기
    await axiosSendFee(companySelling.bankCode, fee);
  }
}

function getFee(companySelling: CompanySelling) {
  return companySelling.sellingAmount * companySelling.commission;
}
```

하나의 기능만 하는 함수로 만들었지만 테스트하기 어렵다.

문제점은 크게 3가지

- 상태 검증 불가능하고 axiosSendFee 가 호출되었는지 행위 검증만 가능함
- Mocking 대상 교체되면 테스트 코드가 수정대상이 되어 버림
- 테스트 도구가 교체되면 테스트 코드가 수정되어야 함

### 왜 테스트 하기 어려운가?

`axiosSendFee` 가 함수 내부에 깊게 들어와있기 때문이다.

`axiosSendFee` 가 부작용함수라서 그렇다?

아무튼 테스트 하기 편하기 위해서는 부작용 함수와 순수 함수를 격리해야 한다.

### 좋은 리팩토링 사례

```javascript
// ----------------- 부작용 함수 ----------------------------
export async function sendCompanyFees(companySellings: CompanySelling[]) {
  await sendFees(companySellings);
  Modal.open(`${companySellings.length} 개 기업들에게 송금되었습니다.`);
}

async function sendFees(companySellings: CompanySelling[]) {
  // 전송 가능한 데이터들만 전달 받아서
  const companyFees = getCompanyFees(companySellings);
  
  // 별도의 추가 작업 없이 바로 전송만 한다.
  for (const companyFee of companyFees) {
    await axiosSendFee(companyFee.bankCode, companyFee.fee);
  }
}

// ----------------- 순수함수 ----------------------------
export function getCompanyFees(companySellings: CompanySelling[]) {
  return companySellings
    .map((c) => getCompanyFee(c))
    .filter((c) => isSend(c));
}

function isSend(c: { fee: number }) {
  return c.fee >= 100;
}

function getCompanyFee(companySelling: CompanySelling) {
  return {
    fee: companySelling.sellingAmount * companySelling.commission,
    bankCode: companySelling.bankCode,
  };
}
```

최대한 순수 함수로 만들고 부작용 함수를 사용하는 작업을 최대한 지연시켰다.

순수 함수는 Mocking 없이도 기능을 테스트할 수 있다.

부작용 함수는 Mocking 테스트 혹은 E2E 테스트로 작성하면 된다.

현재의 함수가 테스트 하기가 어렵다면, 함수 설계에 대해 고민해볼 수 있는 신호가 된다.
