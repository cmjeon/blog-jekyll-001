---
layout: single
title: "markdown 에 diagram 그려넣을 수 있는 mermaid"
categories:
  - "diagram"
tags:
  - "mermaid"
  - "plantuml"
---

## markdown 에 diagram 그려넣을 수 있는 mermaid

markdown 에 표를 이용하여 로그인 개편 방안을 기록하려니까 Sequence Diagram 으로 그리고 싶어졌다.

이미 내가 알고 있는 Sequence Diagram 그리는 도구인 https://plantuml.com/ko/ 가 있다.

plantuml 은 text 로 스크립트를 입력하면 이미지로 된 다이어그램을 그려주는 아주 고마운 도구이다.

plantuml 덕분에 파워포인트나 draw.io 로 한땀한땀 요소를 옮겨가면서 Diagram 을 그리지 않아도 되서 엄청 편해졌다.

좋은 도구이기는 한데 이 도구를 이용해서 markdown 문서에 넣는 것은 text 를 작성하고, 이미지를 업로드 하는 등 조금 귀찮은 일이다.

markdown 에서 바로 다이어그램을 그려보고 싶었다.

그래서 찾다가 mermaid.js 라는 서비스를 알게 되었다.

[https://mermaid-js.github.io/mermaid/#/](https://mermaid-js.github.io/mermaid/#/)

docusaurus 에서는 아직은 뭔가 잘 안되는 듯 하지만, [https://docusaurus.io/feature-requests/p/add-mermaid-support](https://docusaurus.io/feature-requests/p/add-mermaid-support)

mermaid.js 에서는 script 를 추가해서 사용할 수 있다고 한다.

그러면 [https://plausible.io/docs/docusaurus-integration](https://plausible.io/docs/docusaurus-integration) 예시처럼 해볼 수 있지 않을까?

```javascript
exports.config = {
  scripts: [{src: 'https://plausible.io/js/script.js', defer: true, 'data-domain': 'yourdomain.com'}],
};
```

예시를 따라 해봤더니 당장 되지는 않는다.

```javascript
// mermaid import
<link href="https://cdnjs.cloudflare.com/ajax/libs/mermaid/6.0.0/mermaid.css" rel="stylesheet" />
<script src="https://cdnjs.cloudflare.com/ajax/libs/mermaid/6.0.0/mermaid.js"></script>
// mermaid 구동
<script>mermaid.initialize({ startOnLoad: true });</script>

<body>
  <div className="mermaid">
    graph TD
    A[Client] --> B[Load Balancer]
    B --> C[Server01]
    B --> D[Server02]
  </div>
</body>
```

## 금주 회고

다음주부터 주중 스터디는 '토비의 스프링'을 진행한다.

'토비의 스프링' 은 '오브젝트' 와 같이 예전부터 꼭 보고 싶었던 책이었다.

'오브젝트' 는 올해 6월부터 읽기 시작했는데, 아직까지 지지부진하다. ㅎㅎㅎ

'토비의 스프링' 도 책의 분량에 압도되어 시작을 못했었는데, 이번 기회에 보게 되었다.

이번 스터디에서 꼭 완독할 수 있도록 마음을 다져본다.

주말 스터디는 'react.js 강좌듣고 공유하기'를 진행한다.

회사에서는 vue.js 를 사용하고 있지만 react.js 도 한번 접해보고 싶었다.

docusaurus 도 react.js 기반으로 만들어져있기 때문에 배워둔다면 블로그를 구성하는데 좋을 것 같다.

react.js 는 App 이나 PC 용 프로그램으로 개발이 가능하다고 해서 기대가 크다.
