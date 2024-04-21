---
layout: single
title: "마크 다운에 Tip, Info 등 과 접을 수 있는 블록 작성하기"
categories:
  - "docusaurus"
tags:
  - "admonitions"
---

## 마크 다운에 Tip, Info 등 과 접을 수 있는 블록 작성하기

마크 다운 문서에 준수 사항 admonitions 을 작성할 수 있다.

### 노트

:::note

All files prefixed with an underscore (_) under the docs directory are treated as "partial" pages and will be ignored by default.

Read more about importing partial pages.

:::

<!--truncate-->

### 팁

:::tip

팁입니다.

:::

### 정보

:::info

정보입니다.

:::

### 주의

:::caution

CSS-in-JS support is a work in progress, so libs like MUI may have display quirks. Welcoming PRs.

:::

### 위험

:::danger

Some **content** with _Markdown_ `syntax`. Check [this `api`](#).

:::

### 제목 변경하기

:::note 마크다운 내용

Some **content** with _Markdown_ `syntax`.

:::

### 접는 블록

<details>
  <summary>Alternative installation commands</summary>

You can also initialize a new project using your preferred project manager:

```mdx-code-block
<Tabs>
<TabItem value="npm">
```

```bash
npm init docusaurus
```

```mdx-code-block
</TabItem>
<TabItem value="yarn">
```

```bash
yarn create docusaurus
```

```mdx-code-block
</TabItem>
<TabItem value="pnpm">
```

```bash
pnpm create docusaurus
```

```mdx-code-block
</TabItem>
</Tabs>
```

</details>

준수 사항에 대한 내용은 여기서 확인할 수 있음

[https://docusaurus.io/ko/docs/markdown-features/admonitions](https://docusaurus.io/ko/docs/markdown-features/admonitions)
